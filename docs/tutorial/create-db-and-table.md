# Create a Table with SQLModel - Use the Engine

Now let's get to the code. 👩‍💻

Make sure you are inside of your project directory and with your virtual environment activated as [explained in the previous chapter](index.md){.internal-link target=_blank}.

We will:

* Define a table with **SQLModel**
* Create the same SQLite database and table with **SQLModel**
* Use **DB Browser for SQLite** to confirm the operations

Here's a reminder of the table structure we want:

<table>
<tr>
<th>id</th><th>name</th><th>secret_name</th><th>age</th>
</tr>
<tr>
<td>1</td><td>Deadpond</td><td>Dive Wilson</td><td>null</td>
</tr>
<tr>
<td>2</td><td>Spider-Boy</td><td>Pedro Parqueador</td><td>null</td>
</tr>
<tr>
<td>3</td><td>Rusty-Man</td><td>Tommy Sharp</td><td>48</td>
</tr>
</table>

## Create the Table Model Class

The first thing we need to do is create a class to represent the data in the table.

A class like this that represents some data is commonly called a **model**.

/// tip

That's why this package is called `SQLModel`. Because it's mainly used to create **SQL Models**.

///

For that, we will import `SQLModel` (plus other things we will also use) and create a class `Hero` that inherits from `SQLModel` and represents the **table model** for our heroes:

{* ./docs_src/tutorial/create_db_and_table/tutorial001_py310.py ln[1:8] hl[1,4] *}

This class `Hero` **represents the table** for our heroes. And each instance we create later will **represent a row** in the table.

We use the config `table=True` to tell **SQLModel** that this is a **table model**, it represents a table.

/// info

It's also possible to have models without `table=True`, those would be only **data models**, without a table in the database, they would not be **table models**.

Those **data models** will be **very useful later**, but for now, we'll just keep adding the `table=True` configuration.

///

## Define the Fields, Columns

The next step is to define the fields or columns of the class by using standard Python type annotations.

The name of each of these variables will be the name of the column in the table.

And the type of each of them will also be the type of table column:

{* ./docs_src/tutorial/create_db_and_table/tutorial001_py310.py ln[1:8] hl[1,5:8] *}

Let's now see with more detail these field/column declarations.

### `None` Fields, Nullable Columns

Let's start with `age`, notice that it has a type of `int | None`.

That is the standard way to declare that something "could be an `int` or `None`" in Python.

And we also set the default value of `age` to `None`.

{* ./docs_src/tutorial/create_db_and_table/tutorial001_py310.py ln[1:8] hl[8] *}

/// tip

We also define `id` with `int | None`. But we will talk about `id` below.

///

Because the type is `int | None`:

* When validating data, `None` will be an allowed value for `age`.
* In the database, the column for `age` will be allowed to have `NULL` (the SQL equivalent to Python's `None`).

And because there's a default value `= None`:

* When validating data, this `age` field won't be required, it will be `None` by default.
* When saving to the database, the `age` column will have a `NULL` value by default.

/// tip

The default value could have been something else, like `= 42`.

///

### Primary Key `id`

Now let's review the `id` field. This is the <abbr title="That unique identifier of each row in a specific table.">**primary key**</abbr> of the table.

So, we need to mark `id` as the **primary key**.

To do that, we use the special `Field` function from `sqlmodel` and set the argument `primary_key=True`:

{* ./docs_src/tutorial/create_db_and_table/tutorial001_py310.py ln[1:8] hl[1,5] *}

That way, we tell **SQLModel** that this `id` field/column is the primary key of the table.

But inside the SQL database, it is **always required** and can't be `NULL`. Why should we declare it with `int | None`?

The `id` will be required in the database, but it will be *generated by the database*, not by our code.

So, whenever we create an instance of this class (in the next chapters), we *will not set the `id`*. And the value of `id` will be `None` **until we save it in the database**, and then it will finally have a value.

```Python
my_hero = Hero(name="Spider-Boy", secret_name="Pedro Parqueador")

do_something(my_hero.id)  # Oh no! my_hero.id is None! 😱🚨

# Imagine this saves it to the database
somehow_save_in_db(my_hero)

do_something(my_hero.id)  # Now my_hero.id has a value generated in DB 🎉
```

So, because in *our code* (not in the database) the value of `id` *could be* `None`, we use `int | None`. This way **the editor will be able to help us**, for example, if we try to access the `id` of an object that we haven't saved in the database yet and would still be `None`.

<img class="shadow" src="/img/create-db-and-table/inline-errors01.png">

Now, because we are taking the place of the default value with our `Field()` function, we set **the actual default value** of `id` to `None` with the argument `default=None` in `Field()`:

```Python
Field(default=None)
```

If we didn't set the `default` value, whenever we use this model later to do data validation (powered by Pydantic) it would *accept* a value of `None` apart from an `int`, but it would still **require** passing that `None` value. And it would be confusing for whoever is using this model later (probably us), so **better set the default value here**.

## Create the Engine

Now we need to create the SQLAlchemy **Engine**.

It is an object that handles the communication with the database.

If you have a server database (for example PostgreSQL or MySQL), the **engine** will hold the **network connections** to that database.

Creating the **engine** is very simple, just call `create_engine()` with a URL for the database to use:

{* ./docs_src/tutorial/create_db_and_table/tutorial001_py310.py ln[1:16] hl[1,14] *}

You should normally have a single **engine** object for your whole application and re-use it everywhere.

/// tip

There's another related thing called a **Session** that normally should *not* be a single object per application.

But we will talk about it later.

///

### Engine Database URL

Each supported database has its own URL type. For example, for **SQLite** it is `sqlite:///` followed by the file path. For example:

* `sqlite:///database.db`
* `sqlite:///databases/local/application.db`
* `sqlite:///db.sqlite`

SQLite supports a special database that lives all *in memory*. Hence, it's very fast, but be careful, the database gets deleted after the program terminates. You can specify this in-memory database by using just two slash characters (`//`) and no file name:

* `sqlite://`

{* ./docs_src/tutorial/create_db_and_table/tutorial001_py310.py ln[1:16] hl[11:12,14] *}

You can read a lot more about all the databases supported by **SQLAlchemy** (and that way supported by **SQLModel**) in the <a href="https://docs.sqlalchemy.org/en/14/core/engines.html" class="external-link" target="_blank">SQLAlchemy documentation</a>.

### Engine Echo

In this example, we are also using the argument `echo=True`.

It will make the engine print all the SQL statements it executes, which can help you understand what's happening.

It is particularly useful for **learning** and **debugging**:

{* ./docs_src/tutorial/create_db_and_table/tutorial001_py310.py ln[1:16] hl[14] *}

But in production, you would probably want to remove `echo=True`:

```Python
engine = create_engine(sqlite_url)
```

### Engine Technical Details

/// tip

If you didn't know about SQLAlchemy before and are just learning **SQLModel**, you can probably skip this section, scroll below.

///

You can read a lot more about the engine in the <a href="https://docs.sqlalchemy.org/en/14/tutorial/engine.html" class="external-link" target="_blank">SQLAlchemy documentation</a>.

**SQLModel** defines its own `create_engine()` function. It is the same as SQLAlchemy's `create_engine()`, but with the difference that it defaults to use `future=True` (which means that it uses the style of the latest SQLAlchemy, 1.4, and the future 2.0).

And SQLModel's version of `create_engine()` is type annotated internally, so your editor will be able to help you with autocompletion and inline errors.

## Create the Database and Table

Now everything is in place to finally create the database and table:

{* ./docs_src/tutorial/create_db_and_table/tutorial001_py310.py hl[16] *}

/// tip

Creating the engine doesn't create the `database.db` file.

But once we run `SQLModel.metadata.create_all(engine)`, it creates the `database.db` file **and** creates the `hero` table in that database.

Both things are done in this single step.

///

Let's unwrap that:

```Python
SQLModel.metadata.create_all(engine)
```

### SQLModel MetaData

The `SQLModel` class has a `metadata` attribute. It is an instance of a class `MetaData`.

Whenever you create a class that inherits from `SQLModel` **and is configured with `table = True`**, it is registered in this `metadata` attribute.

So, by the last line, `SQLModel.metadata` already has the `Hero` registered.

### Calling `create_all()`

This `MetaData` object at `SQLModel.metadata` has a `create_all()` method.

It takes an **engine** and uses it to create the database and all the tables registered in this `MetaData` object.

### SQLModel MetaData Order Matters

This also means that you have to call `SQLModel.metadata.create_all()` *after* the code that creates new model classes inheriting from `SQLModel`.

For example, let's imagine you do this:

* Create the models in one Python file `models.py`.
* Create the engine object in a file `db.py`.
* Create your main app and call `SQLModel.metadata.create_all()` in `app.py`.

If you only imported `SQLModel` and tried to call `SQLModel.metadata.create_all()` in `app.py`, it would not create your tables:

```Python
# This wouldn't work! 🚨
from sqlmodel import SQLModel

from .db import engine

SQLModel.metadata.create_all(engine)
```

It wouldn't work because when you import `SQLModel` alone, Python doesn't execute all the code creating the classes inheriting from it (in our example, the class `Hero`), so `SQLModel.metadata` is still empty.

But if you import the models *before* calling `SQLModel.metadata.create_all()`, it will work:

```Python
from sqlmodel import SQLModel

from . import models
from .db import engine

SQLModel.metadata.create_all(engine)
```

This would work because by importing the models, Python executes all the code creating the classes inheriting from `SQLModel` and registering them in the `SQLModel.metadata`.

As an alternative, you could import `SQLModel` and your models inside of `db.py`:

```Python
# db.py
from sqlmodel import SQLModel, create_engine
from . import models


sqlite_file_name = "database.db"
sqlite_url = f"sqlite:///{sqlite_file_name}"

engine = create_engine(sqlite_url)
```

And then import `SQLModel` *from* `db.py` in `app.py`, and there call `SQLModel.metadata.create_all()`:

```Python
# app.py
from .db import engine, SQLModel

SQLModel.metadata.create_all(engine)
```

The import of `SQLModel` from `db.py` would work because `SQLModel` is also imported in `db.py`.

And this trick would work correctly and create the tables in the database because by importing `SQLModel` from `db.py`, Python executes all the code creating the classes that inherit from `SQLModel` in that `db.py` file, for example, the class `Hero`.

## Migrations

For this simple example, and for most of the **Tutorial - User Guide**, using `SQLModel.metadata.create_all()` is enough.

But for a production system you would probably want to use a system to migrate the database.

This would be useful and important, for example, whenever you add or remove a column, add a new table, change a type, etc.

But you will learn about migrations later in the Advanced User Guide.

## Run The Program

Let's run the program to see it all working.

Put the code it in a file `app.py` if you haven't already.

{* ./docs_src/tutorial/create_db_and_table/tutorial001_py310.py *}

/// tip

Remember to [activate the virtual environment](./index.md#create-a-python-virtual-environment){.internal-link target=_blank} before running it.

///

Now run the program with Python:

<div class="termy">

```console
// We set echo=True, so this will show the SQL code
$ python app.py

// First, some boilerplate SQL that we are not that interested in

INFO Engine BEGIN (implicit)
INFO Engine PRAGMA main.table_info("hero")
INFO Engine [raw sql] ()
INFO Engine PRAGMA temp.table_info("hero")
INFO Engine [raw sql] ()
INFO Engine

// Finally, the glorious SQL to create the table ✨

CREATE TABLE hero (
        id INTEGER,
        name VARCHAR NOT NULL,
        secret_name VARCHAR NOT NULL,
        age INTEGER,
        PRIMARY KEY (id)
)

// More SQL boilerplate

INFO Engine [no key 0.00020s] ()
INFO Engine COMMIT
```

</div>

/// info

I simplified the output above a bit to make it easier to read.

But in reality, instead of showing:

```
INFO Engine BEGIN (implicit)
```

it would show something like:

```
2021-07-25 21:37:39,175 INFO sqlalchemy.engine.Engine BEGIN (implicit)
```

///

### `TEXT` or `VARCHAR`

In the example in the previous chapter we created the table using `TEXT` for some columns.

But in this output SQLAlchemy is using `VARCHAR` instead. Let's see what's going on.

Remember that [each SQL Database has some different variations in what they support?](../databases.md#sql-the-language){.internal-link target=_blank}

This is one of the differences. Each database supports some particular **data types**, like `INTEGER` and `TEXT`.

Some databases have some particular types that are special for certain things. For example, PostgreSQL and MySQL support `BOOLEAN` for values of `True` and `False`. SQLite accepts SQL with booleans, even when defining table columns, but what it actually uses internally are `INTEGER`s, with `1` to represent `True` and `0` to represent `False`.

The same way, there are several possible types for storing strings. SQLite uses the `TEXT` type. But other databases like PostgreSQL and MySQL use the `VARCHAR` type by default, and `VARCHAR` is one of the most common data types.

**`VARCHAR`** comes from **variable** length **character**.

SQLAlchemy generates the SQL statements to create tables using `VARCHAR`, and then SQLite receives them, and internally converts them to `TEXT`s.

Additional to the difference between those two data types, some databases like MySQL require setting a maximum length for the `VARCHAR` types, for example `VARCHAR(255)` sets the maximum number of characters to 255.

To make it easier to start using **SQLModel** right away independent of the database you use (even with MySQL), and without any extra configurations, by default, `str` fields are interpreted as `VARCHAR` in most databases and `VARCHAR(255)` in MySQL, this way you know the same class will be compatible with the most popular databases without extra effort.

/// tip

You will learn how to change the maximum length of string columns later in the Advanced Tutorial - User Guide.

///

### Verify the Database

Now, open the database with **DB Browser for SQLite**, you will see that the program created the table `hero` just as before. 🎉

<img class="shadow" src="/img/create-db-and-table-with-db-browser/image008.png">

## Refactor Data Creation

Now let's restructure the code a bit to make it easier to **reuse**, **share**, and **test** later.

Let's move the code that has the main **side effects**, that changes data (creates a file with a database and a table) to a function.

In this example it's just the `SQLModel.metadata.create_all(engine)`.

Let's put it in a function `create_db_and_tables()`:

{* ./docs_src/tutorial/create_db_and_table/tutorial002_py310.py ln[1:18] hl[17:18] *}

If `SQLModel.metadata.create_all(engine)` was not in a function and we tried to import something from this module (from this file) in another, it would try to create the database and table **every time** we executed that other file that imported this module.

We don't want that to happen like that, only when we **intend** it to happen, that's why we put it in a function, because we can make sure that the tables are created only when we call that function, and not when this module is imported somewhere else.

Now we would be able to, for example, import the `Hero` class in some other file without having those **side effects**.

/// tip

😅 **Spoiler alert**: The function is called `create_db_and_tables()` because we will have more **tables** in the future with other classes apart from `Hero`. 🚀

///

### Create Data as a Script

We prevented the side effects when importing something from your `app.py` file.

But we still want it to **create the database and table** when we call it with Python directly as an independent script from the terminal, just as above.

/// tip

Think of the word **script** and **program** as interchangeable.

The word **script** often implies that the code could be run independently and easily. Or in some cases it refers to a relatively simple program.

///

For that we can use the special variable `__name__` in an `if` block:

{* ./docs_src/tutorial/create_db_and_table/tutorial002_py310.py hl[21:22] *}

### About `__name__ == "__main__"`

The main purpose of the `__name__ == "__main__"` is to have some code that is executed when your file is called with:

<div class="termy">

```console
$ python app.py

// Something happens here ✨
```

</div>

...but is not called when another file imports it, like in:

```Python
from app import Hero
```

/// tip

That `if` block using `if __name__ == "__main__":` is sometimes called the "**main block**".

The official name (in the <a href="https://docs.python.org/3/library/__main__.html" class="external-link" target="_blank">Python docs</a>) is "**Top-level script environment**".

///

#### More details

Let's say your file is named `myapp.py`.

If you run it with:

<div class="termy">

```console
$ python myapp.py

// This will call create_db_and_tables()
```

</div>

...then the internal variable `__name__` in your file, created automatically by Python, will have as value the string `"__main__"`.

So, the function in:

```Python hl_lines="2"
if __name__ == "__main__":
    create_db_and_tables()
```

...will run.

---

This won't happen if you import that module (file).

So, if you have another file `importer.py` with:

```Python
from myapp import Hero

# Some more code
```

...in that case, the automatic variable inside of `myapp.py` will not have the variable `__name__` with a value of `"__main__"`.

So, the line:

```Python hl_lines="2"
if __name__ == "__main__":
    create_db_and_tables()
```

...will **not** be executed.

/// info

For more information, check <a href="https://docs.python.org/3/library/__main__.html" class="external-link" target="_blank">the official Python docs</a>.

///

## Last Review

After those changes, you could run it again, and it would generate the same output as before.

But now we can import things from this module in other files.

Now, let's give the code a final look:

//// tab | Python 3.10+

```{.python .annotate}
{!./docs_src/tutorial/create_db_and_table/tutorial003_py310.py!}
```

{!./docs_src/tutorial/create_db_and_table/annotations/en/tutorial003.md!}

////

//// tab | Python 3.8+

```{.python .annotate}
{!./docs_src/tutorial/create_db_and_table/tutorial003.py!}
```

{!./docs_src/tutorial/create_db_and_table/annotations/en/tutorial003.md!}

////

/// tip

Review what each line does by clicking each number bubble in the code. 👆

///

## Recap

We learnt how to use **SQLModel** to define how a table in the database should look like, and we created a database and a table using **SQLModel**.

We also refactored the code to make it easier to reuse, share, and test later.

In the next chapters we will see how **SQLModel** will help us interact with SQL databases from code. 🤓
