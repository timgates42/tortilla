Tortilla
========


|build| |coverage| |docs| |version| |pyversions| |license|

.. |build| image:: https://img.shields.io/travis/tortilla/tortilla.svg
    :target: https://travis-ci.org/tortilla/tortilla

.. |coverage| image:: https://img.shields.io/codecov/c/github/tortilla/tortilla.svg
    :target: https://codecov.io/gh/tortilla/tortilla

.. |docs| image:: https://readthedocs.org/projects/tortilla/badge/?version=latest
    :target: https://tortilla.readthedocs.org/en/latest/

.. |version| image:: https://img.shields.io/pypi/v/tortilla.svg
    :target: https://pypi.org/project/tortilla

.. |pyversions| image:: https://img.shields.io/pypi/pyversions/tortilla.svg
    :target: https://pypi.org/project/tortilla

.. |license| image:: https://img.shields.io/github/license/tortilla/tortilla.svg
    :target: https://github.com/tortilla/tortilla/blob/master/LICENSE


*Wrapping web APIs made easy.*


Installation via PIP:

.. code-block:: text

    pip install tortilla


Quick usage overview:

.. code-block:: python

    >>> import tortilla
    >>> github = tortilla.wrap('https://api.github.com')
    >>> user = github.users.get('octocat')
    >>> user.location
    'San Francisco'


The Basics
~~~~~~~~~~

Tortilla uses a bit of magic to wrap APIs. Whenever you get or call an
attribute of a wrapper, the URL is appended by that attribute's name or
method parameter. Let's say we have the following code:

.. code-block:: python

    id, count = 71, 20
    api = tortilla.wrap('https://api.example.org')
    api.video(id).comments.get(count)

Every attribute and method call represents a part of the URL:

.. code-block:: text

    api         -> https://api.example.org
    .video      -> /video
    (id)        -> /71
    .comments   -> /comments
    .get(count) -> /20
    Final URL   -> https://api.example.org/video/71/comments/20

The last part of the chain (``.get()``) executes the request. It also
(optionally) appends one last part to the URL. Which allows you to do
stuff like this:

.. code-block:: python

    api.video.get(id)
    # instead of this
    api.video(id).get()

So to summarize, getting attributes is used to define static parts of a
URL and calling them is used to define dynamic parts of a URL.

Once you've chained everything together, Tortilla will execute the
request and parse the response for you.

At the moment, Tortilla only accepts JSON-formatted responses.
Supporting more formats is on the roadmap for future Tortilla versions.

The parsed response will be *bunchified* which makes dictionary keys
accessible through attributes. So, say we get the following JSON
response for the user 'john':

.. code-block:: json

    {"name": "John Doe"}

If we request this with an already created wrapper, we can access the
response data through attributes:

.. code-block:: python

    >>> user = api.users.get('john')
    >>> user.name
    'John Doe'


Headers
~~~~~~~

A common requirement for accessing APIs is providing authentication
data. This usually has to be described in the headers of each request.
Tortilla makes it very easy for you to describe those recurring headers:

.. code-block:: python

    api.config.headers.token = 'secret authentication token'

You can also define custom headers per request:

.. code-block:: python

    api.endpoint.get(headers={'this': 'that'})

These headers will be appended to the existing headers of the wrapper.


Parameters
~~~~~~~~~~

URL parameters can be defined per request in the ``params`` option:

.. code-block:: python

    api.search.get(params={'q': 'search query'})


Caching
~~~~~~~

Some APIs have a limit on the amount of requests you can make. In these
cases, caching can be very helpful. You can activate this with the
``cache_lifetime`` parameter:

.. code-block:: python

    api = tortilla.wrap('https://api.example.org', cache_lifetime=100)

All the requests made on this wrapper will now be cached for 100
seconds. If you want to ignore the cache in a specific situation, you
can use the ``ignore_cache`` parameter:

.. code-block:: python

    api.special.request.get(ignore_cache=True)

The response will now be reloaded.


URL Extensions
~~~~~~~~~~~~~~

APIs like Twitter's require an extension in the URL that specifies the
response format. This can be defined in the ``extension`` parameter:

.. code-block:: python

    api = tortilla.wrap('https://api.twitter.com/1.1', extension='json')

This option can be overridden with every request or subwrap:

.. code-block:: python

    api.special.endpoint.extension = 'xml'
    api.special.endpoint.get(extension='xml')


URL Suffix
~~~~~~~~~~

Some APIs uses a trailing slash at the end of URLs like in example below:

.. code-block:: text

    https://api.example.org/resource/

You can add the trailing slash with ``suffix="/"`` argument when wrapping
the API or getting the URL with ``.url(suffix="/")`` method:

.. code-block:: python

    api = tortilla.wrap('https://api.example.org', suffix="/")
    api.video(71).comments.url()

Will return the following URL:

.. code-block:: text

    api         -> https://api.example.org
    .video      -> /video
    (id)        -> /71/
    Final URL   -> https://api.example.org/video/71/


Debugging
~~~~~~~~~

Activating debug mode can be done with the ``debug`` parameter:

.. code-block:: python

    api.debug = True
    # OR
    api = tortilla.wrap('https://api.example.org', debug=True)

You can override the ``debug`` parameter per request:

.. code-block:: python

    api.stuff.get(debug=False)
    api.other.stuff.get(debug=True)

An example using the GitHub API:

.. code-block:: python

    >>> user = github.users.get('octocat')
    Executing GET request:
        URL:     https://api.github.com/users/octocat
        headers: {}
        query:   None
        data:    None

    Got 200 OK:
        {u'public_repos': 5, u'site_admin': ...


*Enjoy your data.*
