# RealPython: [Get Started With Django: Build a Portfolio App](https://realpython.com/get-started-with-django-1/)

## Notes, Doubts

**Doubts here**<br>
`django-admin startproject` command<br>
`python manage.py startapp` command

**Subapps** are added to the `INSTALLED_APPS` list in `settings.py`: To include an app in your Django project, you need to add a reference to its configuration class at the beginning of the `INSTALLED_APPS` list in `settings.py`.

**Views** are added in the `view.py` file<br>
&nbsp;&nbsp;&nbsp;&nbsp;Views can be rendered templates if our app is a ui that interacts directly with the database.<br>
&nbsp;&nbsp;&nbsp;&nbsp;Views can be responses if our app is backend service.

**Routes** for a subapp are added in the `urlpatterns` list in the `urls.py` file for the whole application, this is usually a path object with a reference to the the `urls.py` file in the subapp.<br>
The `urls.py` file for the subapp is similar to the `urls.py` for the superapp but contains only the routes for the subapp, this helps with separation of routes for each subapp.

**Templates**<br>
The default settings register `templates` directories in each app, but not in the root directory itself, to do that, we need to add the `templates` directory in the `TEMPLATES` list's first dict's "DIRS" list.

**Django ORM**<br>
An ORM is a program that allows you to create classes that correspond to database tables. Class attributes correspond to columns, and instances of the classes correspond to rows in the database.<br>
When you’re using an ORM, the classes that represent database tables are referred to as Models. In Django, they live in the models.py module of each Django app.<br>
Django models come with many [built-in model field types](https://docs.djangoproject.com/en/4.2/ref/models/fields/).<br>
By default, the Django ORM creates databases in SQLite, but you can use other databases that use the SQL language, such as PostgreSQL or MySQL, with the Django ORM.<br>
For the Models, how do we define a specific primary key rather than the ORM doing that itself

**Doubts here**<br>
`python manage.py makemigrations` command<br>
`python manage.py migrate` command

Running `makemigrations` command creates the `0001_initial.py` file in the subapp's `migrations` folder. If the `migrations` folder isn't there, it is created by the command. This file contains the instructions necessary to apply the Models for the subapp to the database.

When running a query in a `View` using a `Model`, the database query returns a collection of all the objects that match the query, known as a `Queryset`.
