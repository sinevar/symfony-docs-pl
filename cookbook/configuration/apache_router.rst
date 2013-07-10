.. index::
   single: Apache Router

Jak używać Apache Router
========================

Symfony2, mimo tego, iż szybkie z założenia, prezentuje dodatkowe sposoby na
zwiększenie szybkości za sprawą pewnych udoskonaleń. Jednym z nich jest pozwolenie
serwerowi Apache na obsługiwanie wszystkich tras.

Zmiana parametrów konfiguracji Routera
--------------------------------------

W celu zrzucenia tras serwera Apache należy najpierw dopracować kilka parametrów
konfiguracyjnych tak, aby Symfony2 zamiast swojej domyślnej klasy, używało
klasę ``ApacheUrlMatcher``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_prod.yml
        parameters:
            router.options.matcher.cache_class: ~ # wyłączenie pamięci podręcznej routera
            router.options.matcher_class: Symfony\Component\Routing\Matcher\ApacheUrlMatcher

    .. code-block:: xml

        <!-- app/config/config_prod.xml -->
        <parameters>
            <parameter key="router.options.matcher.cache_class">null</parameter> <!-- wyłączenie pamięci podręcznej routera-->
            <parameter key="router.options.matcher_class">
                Symfony\Component\Routing\Matcher\ApacheUrlMatcher
            </parameter>
        </parameters>

    .. code-block:: php

        // app/config/config_prod.php
        $container->setParameter('router.options.matcher.cache_class', null); // wyłączenie pamięci podręcznej routera
        $container->setParameter(
            'router.options.matcher_class',
            'Symfony\Component\Routing\Matcher\ApacheUrlMatcher'
        );

.. tip::

    Należy pamiętać, że :class:`Symfony\\Component\\Routing\\Matcher\\ApacheUrlMatcher`
    rozszerza :class:`Symfony\\Component\\Routing\\Matcher\\UrlMatcher`, zatem
    nawet gdy nie przegeneruje się url_rewrite_rules, wszystko będzie wciąż
    działać (ponieważ na koniec metody ``ApacheUrlMatcher::match()`` wołana jest
    metoda rodzica ``parent::match()``).

Generowanie reguł mod_rewrite
-----------------------------

Aby sprawdzić czy to działa, należy stworzyć podstawową trasę dla pakietu demo:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        hello:
            path:  /hello/{name}
            defaults: { _controller: AcmeDemoBundle:Demo:hello }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <route id="hello" path="/hello/{name}">
            <default key="_controller">AcmeDemoBundle:Demo:hello</default>
        </route>

    .. code-block:: php

        // app/config/routing.php
        $collection->add('hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeDemoBundle:Demo:hello',
        )));

Teraz należy wygenerować reguły **url_rewrite**:

.. code-block:: bash

    $ php app/console router:dump-apache -e=prod --no-debug

Które powinny prezentować się następująco:

.. code-block:: apache

    # pomija "prawdziwe" żądania
    RewriteCond %{REQUEST_FILENAME} -f
    RewriteRule .* - [QSA,L]

    # hello
    RewriteCond %{REQUEST_URI} ^/hello/([^/]+?)$
    RewriteRule .* app.php [QSA,L,E=_ROUTING__route:hello,E=_ROUTING_name:%1,E=_ROUTING__controller:AcmeDemoBundle\:Demo\:hello]

Można teraz zaktualizować plik `web/.htaccess`, aby używał on nowo wygenerowanych
reguł, zatem w tym przypadku powinno to wyglądać tak:

.. code-block:: apache

    <IfModule mod_rewrite.c>
        RewriteEngine On

        # skip "real" requests
        RewriteCond %{REQUEST_FILENAME} -f
        RewriteRule .* - [QSA,L]

        # hello
        RewriteCond %{REQUEST_URI} ^/hello/([^/]+?)$
        RewriteRule .* app.php [QSA,L,E=_ROUTING__route:hello,E=_ROUTING_name:%1,E=_ROUTING__controller:AcmeDemoBundle\:Demo\:hello]
    </IfModule>

.. note::

   Aby w pełni skorzystać z tej konfiguracji, powyższą procedurę należy
   przeprowadzać za każdym razem, gdy dodaje się lub zmienia trasę

To jest to!
Wszystko zostało ustawione tak, aby można było korzystać z reguł tras na serwerze Apache.

Dodatkowe ulepszenia
--------------------

Aby zaoszczędzić nieco na czasie przetwarzania, należy zmienić wystąpienia
``Request`` na ``ApacheRequest`` w pliku ``web/app.php``::

    // web/app.php

    require_once __DIR__.'/../app/bootstrap.php.cache';
    require_once __DIR__.'/../app/AppKernel.php';
    //require_once __DIR__.'/../app/AppCache.php';

    use Symfony\Component\HttpFoundation\ApacheRequest;

    $kernel = new AppKernel('prod', false);
    $kernel->loadClassCache();
    //$kernel = new AppCache($kernel);
    $kernel->handle(ApacheRequest::createFromGlobals())->send();
