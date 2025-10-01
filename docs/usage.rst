Usage
=====

Installation
------------

Install from PyPI:

::

    pip install django-autoslug

Optional extras for transliteration support:

- ``pip install 'django-autoslug[cyrillic]'`` to use ``pytils`` for Cyrillic.
- ``pip install 'django-autoslug[translitcodec]'`` to use ``translitcodec``.


Quickstart
----------

Add an ``AutoSlugField`` to your model. By default a populated slug is not
editable and will be generated on save.

.. code-block:: python

    from django.db import models
    from autoslug import AutoSlugField

    class Article(models.Model):
        title = models.CharField(max_length=200)
        slug = AutoSlugField(populate_from='title')

Run migrations as usual.


Advanced examples
-----------------

Uniqueness within a date granularity:

.. code-block:: python

    class Post(models.Model):
        title = models.CharField(max_length=200)
        pub_date = models.DateField(auto_now_add=True)
        slug = AutoSlugField(populate_from='title', unique_with='pub_date__month')

Uniqueness with a related object and a callable ``populate_from``:

.. code-block:: python

    class Author(models.Model):
        name = models.CharField(max_length=100)

    class Post(models.Model):
        title = models.CharField(max_length=200)
        author = models.ForeignKey(Author, on_delete=models.CASCADE)
        slug = AutoSlugField(
            populate_from=lambda instance: instance.title,
            unique_with=['author__name']
        )

Custom ``slugify`` function and a different index separator:

.. code-block:: python

    from autoslug.settings import slugify as default_slugify

    def custom_slugify(value):
        return default_slugify(value).replace('-', '_')

    class Item(models.Model):
        name = models.CharField(max_length=200)
        slug = AutoSlugField(populate_from='name', slugify=custom_slugify, sep='~')


Field options at a glance
-------------------------

- ``populate_from``: string attribute name or a callable that receives the instance.
- ``unique``: enforce global uniqueness of the slug.
- ``unique_with``: string or tuple of attributes (supports lookups like ``date__month`` or ``author__name``) to scope uniqueness.
- ``slugify``: callable to override the slugification behavior.
- ``sep``: separator used when appending an index (default: ``-``).
- ``allow_unicode``: allow non-ASCII letters in generated slugs.
- ``always_update``: regenerate the slug on every save; use with care.
- ``manager`` / ``manager_name``: manager to use when checking uniqueness across model inheritance trees.


Project settings
----------------

Two optional settings influence behavior:

- ``AUTOSLUG_SLUGIFY_FUNCTION``: dotted path or callable for a project-wide slugify function.
- ``AUTOSLUG_MODELTRANSLATION_ENABLE``: set to ``True`` to enable experimental integration with ``django-modeltranslation``.

Example:

.. code-block:: python

    # settings.py
    AUTOSLUG_SLUGIFY_FUNCTION = 'autoslug.utils.slugify'  # default
    AUTOSLUG_MODELTRANSLATION_ENABLE = False


Compatibility
-------------

The library is tested in CI with Python 3.11 and Django 4.2 (LTS). Older
Python/Django combinations may still work but are not covered by the current CI
matrix.


Testing locally
---------------

Install dev requirements and run tests with coverage:

::

    pip install -r requirements/testing.txt
    pip install -r requirements/devel.txt
    pip install "Django>=4.2,<4.3"
    coverage run --source=autoslug run_tests.py
    coverage report -m

