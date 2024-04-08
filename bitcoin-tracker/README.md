# RealPython: [Django Migrations: A Primer](https://realpython.com/django-migrations-a-primer/)
# RealPython: [Digging Deeper Into Django Migrations](https://realpython.com/digging-deeper-into-migrations/)

## Notes, Doubts

Migrations are mainly for keeping the data model of you database up-to-date. Creating the database tables to store your Django models is the job of a database migration. Additionally, whenever you make a change to your models, like adding a field, the database has to be changed too. Migrations handle that as well.

**Easy of use**<br>
Without `migrations` one would have to use SQL everytime a schema change is required, which could be frequent in rapidly changing applications, can might be a huge change in an already running application.

**Lesser effort in keeping Models and Database in Sync**<br>
When changing schemas, one would have to modify the code at the application level and at the DB level or maintain separate tools to do so. An ORM would help us avoiding writing SQL but it actually is the `migrations` that save the double effort, allowing us to make changes at the application level, creating an automatic migration and applying it. This also reduces contex switching between writing application code and SQL.

**Easier to track changes**<br>
`migrations` are plain Python in Django and can be checked in to VCS for easy track of changes.

Running the `makemigrations` command also creates the file `db.sqlite3`, which contains your SQLite database.<br>
When you try to access a non-existing SQLite3 database file, it will automatically be created.<br>
This behavior is unique to SQLite3. If you use any other database backend like PostgreSQL or MySQL, you have to create the database yourself before running makemigrations.

One can take a look at the SQLite DB using the following command:
`python manage.py dbshell`

If we run the `migrate` command:<br>
`python manage.py migrate`<br>
twice, it will first apply new migrations the first time, and if there aren't any migrations the second time, it will not to anything.

use:<br>
`python manage.py showmigrations`<br>
to list all `migrations` applied till now.

### [Unapplying Migrations](https://realpython.com/django-migrations-a-primer/#unapplying-migrations)

A `Migration` operation can be undone by applying the `Migration` before the `Migration` you want to unapply. Undoing a `Migration` doesn't revert the Models in application code to the state before the `Migration` was applied.<br>
One should be careful when applying a previous `Migration` because some effects of applying a migration can't be undone. If you remove a field from a model, create a migration, and apply it, Django will remove the respective column from the database. Unapplying that migration will re-create the column, but it won’t bring back the data that was stored in that column.

### [Naming Migrations](https://realpython.com/django-migrations-a-primer/#naming-migrations)

We can provide a name to a `Migration` rather than using the default name that Django provides it by providing the `--name` flag to the `makemigrations` command:<br>
`python manage.py makemigrations <subapp_folder_name> --name <custom_name>`

**Doubts here**<br>
Can we use migrations to add data to a database?

### [How Django Knows Which Migrations to Apply](https://realpython.com/digging-deeper-into-migrations/#how-django-knows-which-migrations-to-apply)

Django uses a database table called `django_migrations`. Django automatically creates this table in your database the first time you apply any migrations. For each migration that’s applied or faked, a new row is inserted into the table. This table contains migrations that were applied for all subapps. Django will skip the migrations that are already present in this table. Mucking around with the entries in this table can render the migrations in an inconsistent state and should generally be avoided unless you know what you are doing.

**Doubts here**<br>
How would consistency be maintained for the `django_migrations` table if we are working with multiple instances of a database, i.e, we scale out our instances or have a distributed database?

The `makemigrations` command looks for changes in any of the `Model`s, and if it detects a change, it creates a `Migration` file for that change. A single file is created per subapp even if multiple `Model`s for that subapp have changed. The `.py` file generated contains instructions to keep the database consistent with you `Model`s' changes.

For the `migrations` to work, your subapp has to be listed in the `INSTALLED_APPS` setting, and it must contain a `migrations` directory with an `__init__.py` file. Otherwise Django will not create any migrations for it. The migrations directory is automatically created when you create a new subapp with the startapp management command, but it’s easy to forget when creating an app manually.

### [The Migration File](https://realpython.com/digging-deeper-into-migrations/#the-migration-file)

Each `Migration` `.py` file contains a `Migration` class inheriting from `django.db.migrations.Migration` class. It has 2 class attributes. The are 2 `list`s, `dependencies` `list` and `operations` `list`.<br>

**The `operations` `list`** contains operations that need to be performed on the DB for the changes that you've made in your `Model`s. Each element of the `operations` `list` is an instance of some class in the `django.db.migrations` module.<br>
Each operation element is responsible for generating the necessary SQL statements for the database that you application is connected to.<br>
In some cases, Django might not correctly detect your changes. If you rename a `Model` and change several of its `Field`s, then Django might mistake this for a new `Model`. Instead of a `RenameModel` and several `AlterField` operations, it will create a `DeleteModel` and a `CreateModel` operation. Instead of renaming the database table for the `Model`, it will drop it and create a new table with the new name, effectively deleting all your data.<br>
Make it a habit to check the generated migrations and test them on a copy of your database (probably a Docker container) before running them on production data.

Django provides three more operation classes for advanced use cases:<br>
&nbsp;&nbsp;&nbsp;&nbsp;1. `RunSQL` allows you to run custom SQL in the database.<br>
&nbsp;&nbsp;&nbsp;&nbsp;2. `RunPython` allows you to run any Python code.<br>
&nbsp;&nbsp;&nbsp;&nbsp;3. `SeparateDatabaseAndState` is a specialized operation for advanced uses.

**Doubts here**<br>
How would we use `migrations` with a DB other than SQL.

**The `dependencies` `list`** in a `Migration` class contains any `Migration`s that must be applied before this `Migration` can be applied.<br>
A `Migration` can have a dependency on a `Migration` from another subapp, i.e, `Model` changes applied to this subapp were affected by `Model` changes in another subapp. This is usually necessary if a `Model` has a foreign key pointing to another app (and that `Model` changed?).

A `Migration` can be forced to run before the current `Migration` explicitly by populating the `run_before` list attribute in the current `Migration` class.

Dependencies can also be combined so you can have multiple dependencies. This functionality provides a lot of flexibility, as you can accommodate foreign keys that depend upon `Model`s from different apps. With great flexibility come chances of complexity and should be used with caution.<br>

**Doubts here**<br>
Need to practice example of `run_before` and multiple dependencies -> as a POC as well as on scale (how to create an on scale POC?).

The option to explicitly define dependencies between migrations also means that the numbering of the migrations (usually 0001, 0002, 0003, …) doesn’t strictly represent the order in which migrations are applied. You can add any dependency you want and thus control the order without having to re-number all the migrations.

### [How Django Detects Changes to Your Models](https://realpython.com/digging-deeper-into-migrations/#how-django-detects-changes-to-your-models)

When running `makemigrations`, Django doesn't compare the current `Model` to the table schema in the database. Neither does it compare the current `Model` file to an earlier version. Instead, Django goes through all migrations that have been applied and builds a state of what the `Model`s should look like based on them. If that state is different from the current state of the `Model`s, a new `Migration` is created which contains the necessary operations required to reach the current the state. Few caveats:<br>
&nbsp;&nbsp;&nbsp;&nbsp;1. Django will try to create the most efficient migrations: If you add a field named A to a `Model`, then rename it to B, and then run `makemigrations`, then Django will create a new `Migration` to add a field named B.<br>
&nbsp;&nbsp;&nbsp;&nbsp;2. Django `Migration`s have their limits: Django might not come up with the correct migration if you make too many changes at once.<br>
&nbsp;&nbsp;&nbsp;&nbsp;3. Django migration expect you to play by the rules: Messing around with the `django_migrations` table or change your database schema outside of migrations, for example by deleting the database table for a model can lead to a broken migration system.

[**Understanding `SeparateDatabaseAndState`**](https://realpython.com/digging-deeper-into-migrations/#understanding-separatedatabaseandstate)

`SeparateDatabaseAndState` operation can separate the project state (the mental model Django builds) from your database. It is instantiated with two lists of operations:<br>
&nbsp;&nbsp;&nbsp;&nbsp;1. `state_operations` contains operations that are only applied to the project state.<br>
&nbsp;&nbsp;&nbsp;&nbsp;2. `database_operations` contains operations that are only applied to the database.<br>
This operation lets you do any kind of change to your database, but it’s your responsibility to make sure that the project state fits the database afterwards. Example use cases for `SeparateDatabaseAndState` are moving a `Model` from one app to another or [creating an index on a huge database without downtime](https://realpython.com/create-django-index-without-downtime/). It is an advanced operation and carries quite a bit of risk but is sometimes necessary.

**Doubts here**<br>
Need to practice example of `SeparateDatabaseAndState` -> as a POC as well as on scale (how to create an on scale POC?).
