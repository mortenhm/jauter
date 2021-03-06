Jauter is a Java library that can route paths to targets and create paths from
targets and params (reverse routing).

This library is tiny, without additional dependencies, and is intended for use
together with an HTTP server side library. If you want to use with
`Netty <http://netty.io/>`_, see `Netty-Router <https://github.com/sinetja/netty-router>`_
and `Sinetja <https://github.com/sinetja/sinetja>`_.

See `tests <https://github.com/sinetja/jauter/tree/master/src/test/scala/jauter>`_ for examples.

`Javadoc <http://sinetja.github.io/jauter>`_

Create router
~~~~~~~~~~~~~

::

  // Tell Router about your method type.
  // This example uses enum for simplicity.
  public enum MyMethod {
    CONNECT, DELETE, GET, HEAD, OPTIONS, PATCH, POST, PUT, TRACE
  }

  public class MyRouter extends Router<MyMethod, Class<? extends MyAction>, MyRouter> {
    // This is to overcome method chaining inheritance problem
    // http://stackoverflow.com/questions/1069528/method-chaining-inheritance-don-t-play-well-together-java
    @Override protected MyRouter getThis() { return this; }

    @Override protected MyMethod CONNECT() { return MyMethod.CONNECT; }
    @Override protected MyMethod DELETE()  { return MyMethod.DELETE ; }
    @Override protected MyMethod GET()     { return MyMethod.GET    ; }
    @Override protected MyMethod HEAD()    { return MyMethod.HEAD   ; }
    @Override protected MyMethod OPTIONS() { return MyMethod.OPTIONS; }
    @Override protected MyMethod PATCH()   { return MyMethod.PATCH  ; }
    @Override protected MyMethod POST()    { return MyMethod.POST   ; }
    @Override protected MyMethod PUT()     { return MyMethod.PUT    ; }
    @Override protected MyMethod TRACE()   { return MyMethod.TRACE  ; }
  }

Add routes
~~~~~~~~~~

::

  MyRouter router = new MyRouter()
    .GET      ("/articles",       MyArticleIndex.class)
    .GET      ("/articles/:id",   MyArticleShow.class)
    .GET      ("/download/:*",    MyDownload.class)      // ":*" must be the last token
    .GET_FIRST("/articles/new",   MyArticleNew.class)    // This will be matched first
    .ANY      ("/form_or_create", MyFormOrCreate.class)  // This will match any method
    .notFound (My404NotFound.class);

Jauter ignores slashes at both ends, these are the same:

::

  router.GET("articles",   MyArticleIndex.class)
  router.GET("/articles",  MyArticleIndex.class)
  router.GET("/articles/", MyArticleIndex.class)

You can remove routes by target or by path:

::

  router.removeTarget(MyArticleIndex.class)
  router.removePath("/articles")

Match route
~~~~~~~~~~~

::

  import jauter.Routed;

  Routed routed1 = router.route(GET, "/articles/123");
  // routed1.target()   => MyArticleShow.class
  // routed1.notFound() => false
  // routed1.params()   => Map "id" -> "123"

  Routed routed2 = router.route(GET, "/download/foo/bar.png");
  // routed2.target()   => MyDownload.class
  // routed2.notFound() => false
  // routed2.params()   => Map of "*" -> "foo/bar.png"

  Routed routed3 = router.route(GET, "/noexist");
  // routed3.target()   => My404NotFound.class
  // routed3.notFound() => true
  // routed3.params()   => Empty Map
  // If a notFound were not registered, routed3 will be null

You should pass only the path part of the request URL to ``route``.
Do not pass ``/articles/123?foo=bar`` or ``http://example.com/articles/123`` etc.

Create path (reverse routing)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

  router.path(GET, MyArticleIndex.class);  // => "/articles"

You can skip method if there's no confusion:

::

  router.path(MyArticleIndex.class)  // => "/articles"
  router.path(NoExist.class)         // => null

With params:

::

  // Things in params will be converted to String
  Map<Object, Object> params = new HashMap<Object, Object>();
  params.put("id", 123);
  router.path(MyArticleShow.class, params)  // => "/articles/123"

With params (more convenient):

::

  router.path(MyArticleShow.class, "id", 123)     // => "/articles/123"
  router.path(MyDownload.class,    "foo/bar.png") // => "/download/foo/bar.png"

Additional params will be put to the query part:

::

  router.path(MyArticleIndex.class, "x", 1, "y", 2)              // => "/articles?x=1&y=2"
  router.path(MyArticleShow.class, "id", 123, "format", "json")  // => "/articles/123?format=json"

You can specify an instance in pattern, but use the instance's class to create
path. This feature is useful if you want to create web frameworks:

::

  // Optimize speed by precreating.
  // Optimize memory by sharing for all requests.
  MyArticleIndex cachedInstance = new MyArticleIndex();

  Router router = new Router<Object>()
    .pattern("/articles",     cachedInstance)
    .pattern("/articles/:id", MyArticleShow.class);

  // These are the same:
  router.path(cachedInstance);
  router.path(MyArticleIndex.class);

Use with Maven
~~~~~~~~~~~~~~

::

  <dependency>
    <groupId>tv.cntt</groupId>
    <artifactId>jauter</artifactId>
    <version>1.7</version>
  </dependency>
