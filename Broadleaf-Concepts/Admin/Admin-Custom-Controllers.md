# Admin Custom Controllers

If you are trying to add a new section to the Broadleaf admin application and you want to write the entirety of the controller logic (meaning, you don't want to leverage the annotation-driven generated admin pages), you're at the right place. If you want to perform simple CRUD operations on entities, you might be looking for [[Admin Controllers]].

## Adding a new module and section to the Broadleaf admin

1. Create your controller. Here is a very simple implementation of one:

    ```java
    package com.myco.admin.web.controller;

    @Controller
    @RequestMapping("/" + MyController.SECTION_KEY)
    @Secured("PERMISSION_OTHER_DEFAULT")
    public class MyController extends AdminAbstractController {
        
        protected static final String SECTION_KEY = "test";
        
        @RequestMapping(value = "", method = RequestMethod.GET)
        public String test(HttpServletRequest request, HttpServletResponse response, Model model) throws Exception {
            setModelAttributes(model, SECTION_KEY);
            return "myTemplates/test";
        }
            
    }
    ```

    > **Important Notes**: Note that this controller is extending `AdminAbstractController` and calling the `setModelAttributes()` method before returning a template. This line is responsible for setting the currently active menu element and should be invoked in all controller requests.

2. Make sure your controller is picked up by Spring. We're using the annotation driven approach in the example above, so we just need to make sure that the class is component scanned. We'll add this to `applicationContext-servlet-admin.xml`:

    ```xml
    <context:component-scan base-package="com.myco.admin.web" />
    ```

3. Add the necessary database records to generate the menu.

    ```sql
    INSERT INTO `blc_admin_module` (`ADMIN_MODULE_ID`, `DISPLAY_ORDER`, `ICON`, `MODULE_KEY`, `NAME`) VALUES (1, 7, 'icon-barcode', 'MyCustomModule', 'My Custom Module');
    INSERT INTO `blc_admin_section` (`ADMIN_SECTION_ID`, `CEILING_ENTITY`, `DISPLAY_CONTROLLER`, `DISPLAY_ORDER`, `NAME`, `SECTION_KEY`, `URL`, `USE_DEFAULT_HANDLER`, `ADMIN_MODULE_ID`) VALUES (1, NULL, NULL, 1000, 'My Custom Section', 'MyCustomSection', '/test', NULL, 1);
    ```

    > Note: We left the `ceiling_entity` column blank on purpose because this custom controller is not leveraging the annotation-driven approach. Also note that the configured URL for this row matches our controller.

4. Add the necessary database records to configure security. In this example, we'll allow everyone to access it by mapping the section to the `Default Permission`

    ```sql
    INSERT INTO `blc_admin_sec_perm_xref` (`ADMIN_SECTION_ID`, `ADMIN_PERMISSION_ID`) VALUES (1, -1);
    ```

    > **Important Note**: When creating a custom controller that doesn't leverage the annotation-driven approach, you are responsible for setting the appropriate security values. The record inserted into the above table will only control which users are able to see the admin section in the menu. The `@Secured` annotation on the controller method is reponsible for actually enforcing the required permissions/roles to enter that method.

5. Create your template. By default, Broadleaf comes configured to leverage templates found in the admin project. So, given the string that the controller returns, we'll create a file at the path that Thymeleaf is configured to look for templates in:

    ```text
    admin/src/main/webapp/WEB-INF/templates/admin/myTemplates/test.html
    ```

That's it! At this point, your admin menu should be generated appropriately and your controller should be successfully invoked at the `/test` URL.
