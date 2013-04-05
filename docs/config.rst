.. _config:

Configuration Handling
======================
Generally Eve configuration is best done with configuration files. The
configuration files themselves are actual Python files. 

Configuration Files
-------------------
On startup Eve will look for a `settings.py` file in the application folder.
You can choose an alternative filename/path, just pass it as an argument when
you instantiate the application.

::
    
    from eve import Eve

    app = Eve(settings='my_settings.py')
    app.run()

Development / Production
''''''''''''''''''''''''
Most applications need more than one configuration. There should be at least
separate configurations for the production server and the one used during
development. The easiest way to handle this is to use a default configuration
that is always loaded and part of the version control, and a separate
configuration that overrides the values as necessary.

This is the main reason why you can override or extend the settings with the
contents of the file the :envvar:`EVE_SETTINGS` environment variable points to.
The development/local settings could be stored in `settings.py` and then, in
production, you could export EVE_SETTINGS=/path/to/production_setting.py, and
you are done. 

There are many alternative ways to handle development/production
however. Using Python modules for configuration is very convenient, as they
allow for all kind of nice tricks, like being able to seamlessly launch the
same API on both local and production systems, connecting to the appropriate
database instance as needed.  Consider the following example, taken directly
from the :ref:`demo`:

::

    # We want to seamlessy run our API both locally and on Heroku, so:
    if os.environ.get('PORT'):
        # We're hosted on Heroku! Use the MongoHQ sandbox as our backend.
        MONGO_HOST = 'alex.mongohq.com'
        MONGO_PORT = 10047
        MONGO_USERNAME = '<user>'
        MONGO_PASSWORD = '<pw>'
        MONGO_DBNAME = '<dbname>'

        # also, correctly set the API entry point
        SERVER_NAME = 'eve-demo.herokuapp.com'
    else:
        # Running on local machine. Let's just use the local mongod instance.
        MONGO_HOST = 'localhost'
        MONGO_PORT = 27017
        MONGO_USERNAME = 'user'
        MONGO_PASSWORD = 'user'
        MONGO_DBNAME = 'apitest'

        # let's not forget the API entry point
        SERVER_NAME = 'localhost:5000'


.. _global:

Global Configuration
--------------------
Besides defining the general API behaviour, most global configuration settings
are used to define the standard endpoint ruleset, and can be fine-tuned later,
when configuring individual endpoints. Global configuration settings are always
uppercase. 

.. tabularcolumns:: |p{6.5cm}|p{8.5cm}|

=============================== =========================================
``SERVER_NAME``                 Domain on which the API is being hosted. 
                                Supports subdomains. Defaults to 
                                ``localhost:5000``. 

``URL_PREFIX``                  URL prefix for all API endpoints. Will be used 
                                in conjunction with ``SERVER_NAME`` and 
                                ``API_VERSION`` to construct all API urls 
                                (e.g. ``api`` will be rendered to 
                                ``localhost:5000/api/``).  Defaults to ``''``.

``API_VERSION``                 API version. Will be used in conjunction with 
                                ``SERVER_NAME`` and ``URL_PREFIX`` to construct
                                API urls (e.g. ``v1`` will be rendered to
                                ``localhost:5000/v1/``). Defaults to ``''``.

``FILTERS``                     ``True`` if filters are supported for ``GET`` 
                                requests, ``False`` otherwise. Can be overriden
                                by resource settings. Defaults to ``True``.

``SORTING``                     ``True`` if sorting is supported for ``GET``
                                requests, otherwise ``False``. Can be overriden
                                by resource settings. Defaults to ``True``.

``PAGINATION``                  ``True`` if pagination is enabled for ``GET`` 
                                requests, otherwise ``False``. Can be overriden
                                by resource settings. Defaults to ``True``.

``PAGINATION_LIMIT``            Maximum value allowed for ``max_results``
                                querydef parameter. Values exceeding the limit
                                will be silently replaced with this value.
                                You want to aim for a reasonable compromise
                                between performance and transfer size. Defaults
                                to 50.

``PAGINATION_DEFAULT``          Default value for ``max_results`` applied when 
                                the parameter is omitted.  Defaults to 25.

``DATE_FORMAT``                 A Python date format used to parse and render 
                                datetime values. When serving requests, 
                                matching JSON strings will be parsed and stored as
                                ``datetime`` values. In responses, ``datetime``
                                values will be rendered as JSON strings using
                                this format. Defaults to the RFC1123 (ex RFC
                                822) standard ``a, %d %b %Y %H:%M:%S UTC`` 
                                ("Tue, 02 Apr 2013 10:29:13 UTC"). 

``RESOURCE_METHODS``            A list of HTTP methods supported at resource 
                                endpoints. Allowed values: ``GET``, ``POST``,
                                ``DELETE``. ``POST`` is used for insertions.
                                ``DELETE`` will delete *all* resource contents
                                (enable with caution). Can be overriden by
                                resource settings. Defaults to ``['GET']``.

``PUBLIC_METHODS``              A list of HTTP methods supported at resource
                                endpoints, open to public access even when
                                :ref:`auth` is enabled. Can be overriden by
                                resource settings. Defaults to ``[]``.

``ITEM_METHODS``                A list of HTTP methods supported at item 
                                endpoints. Allowed values: ``GET``, ``PATCH``
                                and ``DELETE``. ``PATCH`` or, for clients not
                                supporting PATCH, ``POST`` with the
                                ``X-HTTP-Method-Override`` header tag, is used
                                for item updates; ``DELETE`` for item deletion.
                                Can be overriden by resource settings. Defaults
                                to ``['GET']``.  

``PUBLIC_ITEM_METHODS``         A list of HTTP methods supported at item
                                endpoints, left open to public access when when
                                :ref:`auth` is enabled. Can be overriden by
                                resource settings. Defaults to ``[]``.

``ALLOWED_ROLES``               A list of allowed `roles` for resource
                                endpoints. Can be overriden by resource
                                settings. See :ref:`auth` for more
                                informations. Defaults to ``[]``.

``ALLOWED_ITEM_ROLES``          A list of allowed `roles` for item endpoints. 
                                See :ref:`auth` for more informations. Can be
                                overriden by resource settings.  Defaults to
                                ``[]``.

``CACHE_CONTROL``               Value of the ``Cache-Control`` header field 
                                used when serving ``GET`` requests (e.g. 
                                ``max-age=20,must-revalidate``). Leave empty if
                                you don't want to include cache directives with
                                API responses. Can be overriden by resource
                                settings. Defaults to ``''``.

``CACHE_EXPIRES``               Value (in seconds) of the ``Expires`` header 
                                field used when serving ``GET`` requests. If
                                set to a non-zero value, the header will 
                                always be included, regardless of the setting
                                of ``CACHE_CONTROL``. Can be overriden by
                                resource settings. Defaults to 0.

``X_DOMAINS``                   CORS (Cross-Origin Resource Sharing) support. 
                                Allows API maintainers to specify which domains
                                are allowed to perform CORS requests. Allowed
                                values are: ``None``, a list of domains or '*'
                                for a wide-open API. Defaults to ``None``.

``LAST_UPDATED``                Name of the field used to record a document's 
                                last update date. This field is automatically
                                handled the Eve. Defaults to ``updated``.

``DATE_CREATED``                Name for the field used to record a document
                                creation date. This field is automatically
                                handled by Eve. Defaults to ``created``.

``STATUS_OK``                   Status message returned when data validation is
                                successful. Defaults to `OK`.

``STATUS_ERR``                  Status message returned when data validation
                                failed. Defaults to `ERR`.

``ID_FIELD``                    Name of the field used to uniquely identify
                                resource items within the database. You want
                                this field to be properly indexed on the
                                database.  Defaults to ``_id``. 

``ITEM_LOOKUP``                 ``True`` if item endpoints should be generally 
                                available acroos the API, ``False`` otherwise. 
                                Can be overriden by resource settings. Defaults
                                to ``True``.

``ITEM_LOOKUP_FIELD``           Document field used when looking up a resource
                                item. Can be overriden by resource settings.
                                Defaults to ``ID_FIELD``.

``ITEM_URL``                    RegEx used to construct default item
                                endpoint URLs. Can be overriden by resource
                                settings. Defaults ``[a-f0-9]{24}`` which is
                                MongoDB standard ``Object_Id`` format.

``ITEM_TITLE``                  Title to be used when building item references, 
                                both in XML and JSON responses. Defaults to 
                                resource name, with the plural 's' stripped if
                                present. Can and most likely will be overriden 
                                when configuring single resource endpoints.

``AUTH_USERNAME_FIELD``         Works in conjunction with :ref:`auth`. When 
                                enabled users can only read/update/delete
                                resource items created by themselves. The
                                keyword contains the actual name of the field
                                used to store the username of the user who
                                created the resource item. Can be overwritten 
                                by resource settings. Defaults to ``''``, which 
                                disables the feature. 

``DEBUG``                       ``True`` to enable Debug Mode, ``False``
                                otherwise. 

``MONGO_HOST``                  MongoDB server address.

``MONGO_PORT``                  MongoDB port.

``MONGO_USERNAME``              MongoDB user name.

``MONGO_PASSWORD``              MongoDB password.

``MONGO_DBNAME``                MongoDB database name.

``DOMAIN``                      A dict holding the API domain definition.
                                See `Domain Configuration`_.
=============================== =========================================

.. _domain:

Domain Configuration
--------------------
In Eve terminology `domain` is the definition of the API structure, the area
where you design your API, fine-tune resources endpoints, and define validation
rules. 

``DOMAIN`` is a :ref:`global configuration setting <global>`: a Python
dictionary where keys are API resources and values their definitions. 

::

    # Here we define two API endpoints, 'people' and 'works', leaving their
    # definitions empty.
    DOMAIN = {
        'people': {},
        'works': {},
        }

In the following two sections we will customize the `people` resource.

.. _local:

Resource / Item Endpoints
'''''''''''''''''''''''''
Endpoint customization is mostly done by overriding some :ref:`global settings
<global>`, but other unique settings are also available. Resource settings are
always lowercase.

.. tabularcolumns:: |p{6.5cm}|p{8.5cm}|

=============================== =========================================
``url``                         The endpoint URL. If omitted, the resource key 
                                of the ``DOMAIN`` dict will be used to build
                                the URL. As an example, ``contacts`` would make the
                                `people` resource available at ``/contacts/`` (instead of ``/people/``).

``filters``                     ``True`` if filters are enabled, ``False`` 
                                otherwise. Locally overrides ``FILTERS``.

``sorting``                     ``True`` if sorting is enabled, ``False`` 
                                otherwise. Locally ovverrides ``SORTING``.
                                
``pagination``                  ``True`` if pagination is enabled, ``False``
                                otherwise. Locally overrides ``PAGINATION``.

``methods``                     A list of HTTP methods supported at resource 
                                endpoint. Allowed values: ``GET``, ``POST``,
                                ``DELETE``. Locally overrides
                                ``RESOURCE_METHODS``.

``public_methods``              A list of HTTP methods supported at resource
                                endpoint, open to public access even when
                                :ref:`auth` is enabled. Locally overrides
                                ``PUBLIC_METHODS``.

``item_methods``                A list of HTTP methods supported at item 
                                endpoint. Allowed values: ``GET``, ``PATCH``
                                and ``DELETE``. ``PATCH`` or, for clients not
                                supporting PATCH, ``POST`` with the
                                ``X-HTTP-Method-Override`` header tag.
                                Locally overrides ``ITEM_METHODS``.

``public_item_methods``         A list of HTTP methods supported at item
                                endpoint, left open to public access when when
                                :ref:`auth` is enabled. Locally overrides
                                ``PUBLIC_ITEM_METHODS``.

``allowed_roles``               A list of allowed `roles` for resource
                                endpoint. See :ref:`auth` for more
                                informations. Locally overrides
                                ``ALLOWED_ROLES``.

``allowed_item_roles``          A list of allowed `roles` for item endpoint. 
                                See :ref:`auth` for more informations.
                                Locally overrides ``ALLOWED_ITEM_ROLES``.

``cache_control``               Value of the ``Cache-Control`` header field 
                                used when serving ``GET`` requests. Leave empty
                                if you don't want to include cache directives
                                with API responses. Locally overrides
                                ``CACHE_CONTROL``.

``cache_expires``               Value (in seconds) of the ``Expires`` header 
                                field used when serving ``GET`` requests. If
                                set to a non-zero value, the header will 
                                always be included, regardless of the setting
                                of ``CACHE_CONTROL``. Locally overrides
                                ``CACHE_EXPIRES``.

``item_lookup``                 ``True`` if item endpoint should be available, 
                                ``False`` otherwise. Locally overrides
                                ``ITEM_LOOKUP``.

``item_lookup_field``           Field used when looking up a resource
                                item. Locally overrides ``ITEM_LOOKUP_FIELD``.

``item_url``                    RegEx used to construct item endpoint URL.
                                Locally overrides ``ITEM_URL``.

``item_title``                  Title to be used when building item references, 
                                both in XML and JSON responses. Overrides
                                ``ITEM_TITLE``.

``additional_lookup``           RegEx defining and additional, custom read-only item
                                endpoint. See the example below.

``datasource``                  Explicitly links API resources to database 
                                collections, allowing for some `Advanced
                                Datasource Patterns`_. 

``auth_username_field``         Works in conjunction with :ref:`auth`. When 
                                enabled users can only read/update/delete
                                resource items created by themselves. The
                                keyword contains the actual name of the field
                                used to store the username of the user who
                                created the resource item. Locally overrides 
                                ``AUTH_USERNAME_FIELD``.

``schema``                      A dict defining the actual data structure being
                                handled by the resource. Enables data
                                validation. See `Schema Definition`_.
=============================== =========================================

Here's an example of resource customization, mostly done by overriding global
API settings:

::

    people = {
        # 'title' tag used in item links. Defaults to the resource title minus
        # the final, plural 's' (works fine in most cases but not for 'people')
        'item_title': 'person',

        # by default the standard item entry point is defined as
        # '/people/<ObjectId>/'. We leave it untouched, and we also enable an
        # additional read-only entry point. This way consumers can also perform 
        # GET requests at '/people/<lastname>/'.
        'additional_lookup': {
            'url': '[\w]+',
            'field': 'lastname'
        },

        # We choose to override global cache-control directives for this resource.
        'cache_control': 'max-age=10,must-revalidate',
        'cache_expires': 10,

        # we only allow GET and POST at this resource endpoint.
        'resource_methods': ['GET', 'POST'],
    }

.. _schema:

Schema Definition
'''''''''''''''''
Unless your API is read-only you probably want to define resource `schemas`.
Schemas are important because they enable proper validation for incoming
streams.

::

    # 'people' schema definition
    'schema'= {
        'firstname': {
            'type': 'string',
            'minlength': 1,
            'maxlength': 10,
        },
        'lastname': {
            'type': 'string',
            'minlength': 1,
            'maxlength': 15,
            'required': True,
            'unique': True,
        },
        # 'role' is a list, and can only contain values from 'allowed'.
        'role': {
            'type': 'list',
            'allowed': ["author", "contributor", "copy"],
        },
        # An embedded 'strongly-typed' dictionary.
        'location': {
            'type': 'dict',
            'schema': {
                'address': {'type': 'string'},
                'city': {'type': 'string'}
            },
        },
        'born': {
            'type': 'datetime',
        },
    }

Schema syntax is based on Cerberus_ and yes, it can be extended.  In fact Eve
itself is extending the original grammar by adding the `unique` keyword and the
``ObjectId`` datatype.  For more informations on schema syntax and custom
validation see :ref:`validation`.

In :ref:`local` you customized the `people` endpoint. Then, in this section,
you defined `people` validation rules. Now you are ready to update the domain
which was originally set up in `Domain Configuration`_:

::

    # add the schema to the 'people' resource definition
    people['schema'] = schema
    # update the domain
    DOMAIN['people'] = people

Advanced Datasource Patterns
----------------------------
Predefined Database Filters
'''''''''''''''''''''''''''
By using the ``datasource`` resource keyword it is possibile to set predefined
database filters. 

::

    people = {
        'datasource': {
            'filter': {'username': {'$exists': True}}
            }
        }
  
In the example above the API endpoint will only expose and update documents
with an existent `username` field.

Predefined filters run on top of user queries (GET requests with `where`
clauses) and standard conditional requests (`If-Modified-Since`, etc.)

Please note that datasource filters are applied on GET, PATCH and DELETE
requests. If your resource allows POST requests (document insertions),
then you will probably want to set the validation rules accordingly (in our
example, 'username' should probably be a required field).

Multiple API Endpoints, One Datasource
''''''''''''''''''''''''''''''''''''''
Multiple API endpoints can target the same database collection. For
example you can set both ``/admins/`` and ``/users/`` to read and write from
the same collection on the database, `people`.

The ``datasource`` keyword allows to explicitly link API resources to
database collections (if you omit it, the domain resource key is assumed to be
the name of the database collection). It is a dictionary with two allowed keys:
`source` and `filter`. ``source`` dictates the database collection consumed by
the resource, while ``filter`` is the underlying query applied by the API when
retrieving and validating data for the resource.  

::

    people = {
        'datasource': {
            'source': 'people', 
            'filter': {'userlevel': 1}
            }
        }

The above setting will retrieve, edit and delete only documents from the
`people` collection with a `userlevel` of 1.

.. _Cerberus: http://cerberus.readthedocs.org