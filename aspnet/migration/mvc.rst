Migrating From ASP.NET MVC to ASP.NET Core MVC
================================================

By `Rick Anderson`_, `Daniel Roth`_, `Steve Smith`_ and `Scott Addie`_

This article shows how to get started migrating an ASP.NET MVC project to :doc:`ASP.NET Core MVC </mvc/index>`. In the process, it highlights many of the things that have changed from ASP.NET MVC. Migrating from ASP.NET MVC is a multiple step process and this article covers the initial setup, basic controllers and views, static content, and client-side dependencies. Additional articles cover migrating configuration and identity code found in many ASP.NET MVC projects.

.. contents:: Sections:
  :local:
  :depth: 1

Create the starter ASP.NET MVC project
---------------------------------------

To demonstrate the upgrade, we'll start by creating a ASP.NET MVC app. Create it with the name *WebApp1* so the namespace will match the ASP.NET Core project we create in the next step.

.. image:: mvc/_static/new-project.png

.. image:: mvc/_static/new-project-select-mvc-template.png

*Optional:* Change the name of the Solution from *WebApp1* to *Mvc5*. Visual Studio will display the new solution name (*Mvc5*), which will make it easier to tell this project from the next project.

Create the ASP.NET Core project
-----------------------------------

Create a new *empty* ASP.NET Core web app with the same name as the previous project (*WebApp1*) so the namespaces in the two projects match. Having the same namespace makes it easier to copy code between the two projects. You'll have to create this project in a different directory than the previous project to use the same name.

.. image:: mvc/_static/new_core.png

.. image:: mvc/_static/new-project-select-empty-aspnet5-template.png
.. ToDo I think we no longer need full template with **Individual User Accounts**. Next update try this without **Individual User Accounts**

- *Optional:* Create a new ASP.NET Core app using the *Web Application* project template. Name the project *WebApp1*, and select an authentication option of **Individual User Accounts**. Rename this app to *FullAspNetCore*. Creating this project will save you time in the conversion. You can look at the template-generated code to see the end result or to copy code to the conversion project. It's also helpful when you get stuck on a conversion step to compare with the template-generated project.

Configure the site to use MVC
-----------------------------

Open the *project.json* file.

- Add ``Microsoft.AspNetCore.Mvc`` and ``Microsoft.AspNetCore.StaticFiles`` to the ``dependencies`` property:
- Add the ``prepublish`` line to the ``scripts`` section:

.. literalinclude:: mvc/samples/WebApp1/src/WebApp1/project.json
  :language: javascript
  :lines: 49
  :dedent: 8

- ``Microsoft.AspNetCore.Mvc`` installs the ASP.NET Core MVC framework package.
- ``Microsoft.AspNetCore.StaticFiles`` is the static file handler. The ASP.NET runtime is modular, and you must explicitly opt in to serve static files (see :doc:`/fundamentals/static-files`).
- The ``scripts/prepublish`` line is needed for acquiring client-side libraries via Bower. We'll talk about that later.

- Open the *Startup.cs* file and change the code to match the following:

.. literalinclude:: mvc/samples/WebApp1/src/WebApp1/Startup.cs
  :language: none
  :emphasize-lines: 14,27-34

The ``UseStaticFiles`` extension method adds the static file handler. As mentioned previously, the ASP.NET runtime is modular, and you must explicitly opt in to serve static files. The ``UseMvc`` extension method adds routing. For more information, see :doc:`/fundamentals/startup` and :doc:`/fundamentals/routing`.

Add a controller and view
-------------------------

In this section, you'll add a minimal controller and view to serve as placeholders for the ASP.NET MVC controller and views you'll migrate in the next section.

- Add a *Controllers* folder.
- Add an **MVC controller class** with the name *HomeController.cs* to the *Controllers* folder.

.. image:: mvc/_static/add_mvc_ctl.png

- Add a *Views* folder.
- Add a *Views/Home* folder.
- Add an *Index.cshtml* MVC view page to the *Views/Home* folder.

.. image:: mvc/_static/view.png

The project structure is shown below:

.. image:: mvc/_static/project-structure-controller-view.png

Replace the contents of the *Views/Home/Index.cshtml* file with the following:

.. code-block:: html

  <h1>Hello world!</h1>

Run the app.

.. image:: mvc/_static/hello-world.png

See :doc:`/mvc/controllers/index` and :doc:`/mvc/views/index` for more information.

Now that we have a minimal working ASP.NET Core project, we can start migrating functionality from the ASP.NET MVC project. We will need to move the following:

- client-side content (CSS, fonts, and scripts)
- controllers
- views
- models
- bundling
- filters
- Log in/out, identity (This will be done in the next tutorial.)

Controllers and views
---------------------

- Copy each of the methods from the ASP.NET MVC ``HomeController`` to the new ``HomeController``. Note that in ASP.NET MVC, the built-in template's controller action method return type is `ActionResult <https://msdn.microsoft.com/en-us/library/system.web.mvc.actionresult(v=vs.118).aspx>`__; in ASP.NET Core MVC, the action methods return `IActionResult <https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/IActionResult/index.html>`__ instead. ``ActionResult`` implements ``IActionResult``, so there is no need to change the return type of your action methods.
- Copy the *About.cshtml*, *Contact.cshtml*, and *Index.cshtml* Razor view files from the ASP.NET MVC project to the ASP.NET Core project.
- Run the ASP.NET Core app and test each method. We haven't migrated the layout file or styles yet, so the rendered views will only contain the content in the view files. You won't have the layout file generated links for the ``About`` and ``Contact`` views, so you'll have to invoke them from the browser (replace **4492** with the port number used in your project).

  - \http://localhost:4492/home/about
  - \http://localhost:4492/home/contact

.. image:: mvc/_static/contact-page.png

Note the lack of styling and menu items. We'll fix that in the next section.

Static content
--------------

In previous versions of  ASP.NET MVC, static content was hosted from the root of the web project and was intermixed with server-side files. In ASP.NET Core, static content is hosted in the *wwwroot* folder. You'll want to copy the static content from your old ASP.NET MVC app to the *wwwroot* folder in your ASP.NET Core project. In this sample conversion:

- Copy the *favicon.ico* file from the old MVC project to the *wwwroot* folder in the ASP.NET Core project.

The old ASP.NET MVC project uses `Bootstrap <http://getbootstrap.com/>`__ for its styling and stores the Bootstrap files in the *Content* and *Scripts* folders. The template, which generated the old ASP.NET MVC project, references Bootstrap in the layout file (*Views/Shared/_Layout.cshtml*). You could copy the *bootstrap.js* and *bootstrap.css* files from the ASP.NET MVC project to the *wwwroot* folder in the new project, but that approach doesn't use the improved mechanism for managing client-side dependencies in ASP.NET Core.

In the new project, we'll add support for Bootstrap (and other client-side libraries) using `Bower <http://bower.io/>`__:

- Add a `Bower <http://bower.io/>`__ configuration file named *bower.json* to the project root (Right-click on the project, and then **Add > New Item > Bower Configuration File**). Add `Bootstrap <http://getbootstrap.com/>`__ and `jQuery <https://jquery.com/>`__ to the file :sup:`[1]` (see the highlighted lines below).

.. literalinclude:: mvc/samples/WebApp1/src/WebApp1/bower.json
  :language: json
  :emphasize-lines: 5-6

Upon saving the file, Bower will automatically download the dependencies to the *wwwroot/lib* folder. You can use the **Search Solution Explorer** box to find the path of the assets:

.. image:: mvc/_static/search.png

.. Note:: *bower.json* is not visible in **Solution Explorer**. You can display the hidden *.json* files by selecting the project in **Solution Explorer** and then tapping the **Show All Files** icon. You won't see **Show All Files** unless the project is selected.

.. image:: mvc/_static/show_all_files.png

See :doc:`/client-side/bower` for more information.

.. _migrate-layout-file:

Migrate the layout file
-----------------------

- Copy the *_ViewStart.cshtml* file from the old ASP.NET MVC project's *Views* folder into the ASP.NET Core project's *Views* folder. The *_ViewStart.cshtml* file has not changed in ASP.NET Core MVC.
- Create a *Views/Shared* folder.
- *Optional:* Copy *_ViewImports.cshtml* from the old MVC project's *Views* folder into the ASP.NET Core project's *Views* folder. Remove any namespace declaration in the *_ViewImports.cshtml* file. The *_ViewImports.cshtml* file provides namespaces for all the view files and brings in :doc:`Tag Helpers </mvc/views/tag-helpers/index>`. Tag Helpers are used in the new layout file. The *_ViewImports.cshtml* file is new for ASP.NET Core.
- Copy the *_Layout.cshtml* file from the old ASP.NET MVC project's *Views/Shared* folder into the ASP.NET Core project's *Views/Shared* folder.

Open *_Layout.cshtml* file and make the following changes (the completed code is shown below):

  - Replace ``@Styles.Render("~/Content/css")`` with a ``<link>`` element to load *bootstrap.css* (see below).
  - Remove ``@Scripts.Render("~/bundles/modernizr")``.
  - Comment out the ``@Html.Partial("_LoginPartial")`` line (surround the line with ``@*...*@``). We'll return to it in a future tutorial.
  - Replace ``@Scripts.Render("~/bundles/jquery")`` with a ``<script>`` element (see below).
  - Replace ``@Scripts.Render("~/bundles/bootstrap")`` with a ``<script>`` element (see below)..

The replacement CSS link:

.. code-block:: html

  <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />

The replacement script tags:

.. code-block:: html

  <script src="~/lib/jquery/dist/jquery.js"></script>
  <script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>

The updated _Layout.cshtml file is shown below:

.. literalinclude:: mvc/samples/WebApp1/src/WebApp1/Views/Shared/_Layout.cshtml
  :language: html
  :emphasize-lines: 7,26,38-39

View the site in the browser. It should now load correctly, with the expected styles in place.

- *Optional:* You might want to try using the new layout file. For this project you can copy the layout file from the *FullAspNetCore* project. The new layout file uses :doc:`Tag Helpers </mvc/views/tag-helpers/index>` and has other improvements.

Configure Bundling & Minification
---------------------------------

The ASP.NET MVC starter web template utilized the ASP.NET Web Optimization Framework for bundling and minification. In ASP.NET Core, this functionality is performed as part of the build process using `BundlerMinifier.Core <https://www.nuget.org/packages/BundlerMinifier.Core/>`__. To configure it, do the following:

.. Note:: If you created the optional *FullAspNetCore* project, copy the *wwwroot/css/site.css* and *wwwroot/js/site.js* files to the *wwwroot* folder in the *WebApp1* project; otherwise, manually create these files. The ASP.NET Core project's *_Layout.cshtml* file will reference these two files.

- Add a *bundleconfig.json* file to the project root with the content below. This file describes how the bundling and minification of JavaScript and CSS files will take place.

.. literalinclude:: mvc/samples/WebApp1/src/WebApp1/bundleconfig.json
  :language: json

- Add a ``BundlerMinifier.Core`` NuGet package entry to the ``tools`` section of *project.json* :sup:`[1]`:

.. literalinclude:: mvc/samples/WebApp1/src/WebApp1/project.json
  :language: javascript
  :lines: 15-18
  :dedent: 4

- Add a ``precompile`` script to *project.json*'s ``scripts`` section. It should resemble the snippet below. It's this ``dotnet bundle`` command which will use the ``BundlerMinifier.Core`` package's features to bundle and minify the static content.

.. literalinclude:: mvc/samples/WebApp1/src/WebApp1/project.json
  :language: javascript
  :lines: 48
  :dedent: 8

Now that we've configured bundling and minification, all that's left is to change the references to Bootstrap, jQuery, and other assets to use the bundled and minified versions. You can see how this is done in the layout file (*Views/Shared/_Layout.cshtml*) of the full template project. See :doc:`/client-side/bundling-and-minification` for more information.

.. _Solving-HTTP-500-errors:

Solving HTTP 500 errors
-------------------------

There are many problems that can cause a HTTP 500 error message that contain no information on the source of the problem. For example, if the *Views/_ViewImports.cshtml* file contains a namespace that doesn't exist in your project, you'll get a HTTP 500 error. To get a detailed error message, add the following code:

.. code-block:: c#
  :emphasize-lines: 3-6

   public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
   {
       if (env.IsDevelopment())
       {
            app.UseDeveloperExceptionPage();
       }

       app.UseStaticFiles();

       app.UseMvc(routes =>
       {
           routes.MapRoute(
               name: "default",
               template: "{controller=Home}/{action=Index}/{id?}");
       });
   }

See **Using the Developer Exception Page** in :doc:`/fundamentals/error-handling` for more information.

Additional Resources
--------------------

- :doc:`/client-side/index`
- :doc:`Tag Helpers </mvc/views/tag-helpers/index>`

[1] The version numbers in the samples might not be current. You may need to update your projects accordingly.