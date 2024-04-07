# python_django

## doubts

`django-admin startproject` command<br>
`python manage.py startapp` command

## notes

subapps are added to the `INSTALLED_APPS` list in `settings.py`: To include an app in your Django project, you need to add a reference to its configuration class at the beginning of the `INSTALLED_APPS` list in `settings.py`.

views are added in the view.py file<br>
&nbsp;&nbsp;&nbsp;&nbsp;views can be rendered templates if our app is a ui that interacts directly with the database<br>
&nbsp;&nbsp;&nbsp;&nbsp;views can be responses if our app is backend service

routes for a subapp are added in the `urlpatterns` list in the `urls.py` file for the whole application, this is usually a path object with a reference to the the `urls.py` file in the subapp.<br>
the `urls.py` file for the subapp is similar to the `urls.py` for the superapp but contains only the routes for the subapp, this helps with separation of routes

templates<br>
the default settings register templates/ directories in each app, but not in the root directory itself, to do that, we need to add the templates directory in the TEMPLATES list's first dict's "DIRS" list
