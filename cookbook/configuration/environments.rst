.. index::
   single: środowiska

Jak osiągnąć mistrzostwo w tworzeniu nowych środowisk
=====================================================

Każda aplikacja to kombinacja kodu i zestawu konfiguracji, które mają wpływ
na to w jaki sposób dany kod powinien funkcjonować. Konfiguracja może definiować
aktualnie używaną bazę danych, może także określać czy coś powinno być buforowane
lub jak dokładnie coś zalogować. W Symfony2, idea "środowisk" opiera się na
przekonaniu, że ten sam kod źródłowy może być uruchamiany za pomocą wielu różnych
konfiguracji. Dla przykładu, środowisko ``dev`` powinno używać konfiguracji,
która sprawia, że rozwój aplikacji jest łatwy i przyjazny, podczas gdy środowisko
``prod`` powinno używać zestawu konfiguracji zoptymalizowanego pod kątem wydajności.

.. index::
   single: środowiska; pliki konfiguracyjne

Różne środowiska, różne pliki konfiguracyjne
--------------------------------------------

Typowa aplikacja Symfony2 zaczyna z trzema środowiskami: ``dev``,
``prod`` i ``test``. Jak wspomniano, każde "środowisko" oznacza po prostu
sposób na wykonanie tego samego kodu źródłowego z użyciem innej konfiguracji.
Nie powinno być zaskoczeniem, że każde z nich ładuje swój indywidualny plik
konfiguracyjny. Jeśli używano formatu YAML, następujące pliki są używane:

* dla środowiska ``dev``: ``app/config/config_dev.yml``
* dla środowiska ``prod``: ``app/config/config_prod.yml``
* dla środowiska ``test``: ``app/config/config_test.yml``

Działa to za pomocą prostego standardu, który jest używany domyślnie przez
klasę ``AppKernel``:

.. code-block:: php

    // app/AppKernel.php

    // ...
    
    class AppKernel extends Kernel
    {
        // ...

        public function registerContainerConfiguration(LoaderInterface $loader)
        {
            $loader->load(__DIR__.'/config/config_'.$this->getEnvironment().'.yml');
        }
    }

Jak można zauważyć, gdy Symfony2 jest uruchamiane, korzysta wówczas z danego
środowiska aby określić, który plik konfiguracyjny załadować. Realizuje to cel
wielu środowisk w sposób elegancki, potężny i przejrzysty.

Oczywiście, w rzeczywistości, każde środowisko różni się tylko nieznacznie od
innych. Ogólnie rzecz biorąc, wszystkie środowiska będą dzielić dużą cześć
zwykłej konfiguracji. Otwierając plik konfiguracyjny "dev" można zobaczyć
jak łatwo i przejrzyście jest to realizowane:

.. configuration-block::

    .. code-block:: yaml

        imports:
            - { resource: config.yml }
        # ...

    .. code-block:: xml

        <imports>
            <import resource="config.xml" />
        </imports>
        <!-- ... -->

    .. code-block:: php

        $loader->import('config.php');
        // ...

By dzielić zwykłą konfigurację, każdy plik konfiguracyjny danego środowiska
na początku importuje wszystko z centralnego pliku konfiguracyjnego (``config.yml``).
Pozostała część pliku może różnić się od domyślnej konfiguracji poprzez zastąpienie
poszczególnych parametrów. Dla przykładu pasek narzędzi ``web_profiler`` jest
domyślnie wyłączony. Jednak w środowisku ``dev``, pasek ten jest aktywowany dzięki
modyfikacji domyślnej wartości w pliku konfiguracyjnym ``dev``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        imports:
            - { resource: config.yml }

        web_profiler:
            toolbar: true
            # ...

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->
        <imports>
            <import resource="config.xml" />
        </imports>

        <webprofiler:config
            toolbar="true"
            ... />

    .. code-block:: php

        // app/config/config_dev.php
        $loader->import('config.php');

        $container->loadFromExtension('web_profiler', array(
            'toolbar' => true,

            // ...
        ));

.. index::
   single: środowiska; uruchamianie różnych środowisk

Uruchamianie aplikacji w różnych środowiskach
---------------------------------------------

Aby uruchomić aplikację w każdym środowisku, należy załadować aplikację z
użyciem albo ``app.php`` (dla środowiska ``prod``) albo ``app_dev.php``
(dla środowiska ``dev``) w kontrolerze wejściowym:

.. code-block:: text

    http://localhost/app.php      -> środowisko *prod*
    http://localhost/app_dev.php  -> środowisko *dev*

.. note::

   Podane adresy URL zakładają, że serwer został skonfigurowany do używania
   katalogu ``web/`` jako głównego katalogu aplikacji. Więcej informacji można
   znaleźć w :doc:`Installing Symfony2</book/installation>`.

Jeśli by otworzyć jeden z plików, można szybko zauważyć, że używane środowiska
są jawnie ustawiane za pomocą:

.. code-block:: php
   :linenos:

    <?php

    require_once __DIR__.'/../app/bootstrap_cache.php';
    require_once __DIR__.'/../app/AppCache.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppCache(new AppKernel('prod', false));
    $kernel->handle(Request::createFromGlobals())->send();

Jak widać, klucz ``prod`` określa, że aplikacja będzie działać w środowisku
``prod``. Warto wspomnieć, że aplikacja Symfony2 może być wykonywana w dowolnym
środowisku za pomocą tego samego kodu, z wyjątkiem zmiany nazwy środowiska.

.. note::

   Środowisko ``test`` jest używane do testów funkcjonalnych i nie jest bezpośrednio
   dostępne z poziomu przeglądarki przez kontroler wejściowy. Innymi słowy,
   w przeciwieństwie do innych środowisk, nie ma wejściowego pliku kontrolera
   ``app_test.php``.

.. index::
   single: konfiguracja; tryb debugowania

.. sidebar:: Tryb *Debugowania*

    Ważna, lecz nie związana z tematem *środowisk*, jest flaga ``false`` w
    linii 8 kontrolera wejściowego powyżej. Określa ona czy aplikacja powinna
    działać w "trybie debugowania". Niezależnie od środowiska, aplikacja Symfony2
    może zostać uruchamiana w trybie debugowania poprzez ustawianie flagi na
    ``true`` albo na ``false``. Ma to wpływ na wiele rzeczy w aplikacji, na przykład
    na to czy pliki pamięci podręcznej będą dynamicznie przebudowywane przy każdym
    żądaniu. Choć nie jest to wymogiem, tryb debugowania jest zazwyczaj ustawiony
    na ``true`` w środowiskach ``dev`` i ``test`` oraz na ``false`` w środowisku
    ``prod``.

    Wewnętrznie wartość trybu debugowania staje się parametrem ``kernel.debug``
    używanym w :doc:`service container </book/service_container>`.
    Jeśli spojrzeć do pliku konfiguracyjnego aplikacji, z łatwością można zauważyć,
    że parametr ten jest używany do ustawiania bądź wyłączania logowania podczas
    korzystania z Doctrine DBAL:

    .. configuration-block::

        .. code-block:: yaml

            doctrine:
               dbal:
                   logging:  "%kernel.debug%"
                   # ...

        .. code-block:: xml

            <doctrine:dbal logging="%kernel.debug%" ... />

        .. code-block:: php

            $container->loadFromExtension('doctrine', array(
                'dbal' => array(
                    'logging'  => '%kernel.debug%',
                    // ...
                ),
                // ...
            ));

    Począwszy od Symfony 2.3, pokazywanie błędów nie zależy już od
    trybu debugowania. Należy włączyć to w kontrolerze wejściowym poprzez
    wywołanie metody :method:`Symfony\\Component\\Debug\\Debug::enable`.

.. index::
   single: środowiska; tworzenie nowego środowiska

Tworzenie nowego środowiska
---------------------------

Domyślnie aplikacja Symfony2 składa się z trzech środowisk, które obsługują
większość przypadków. Ponieważ środowisko jest niczym innym jak nazwą, która
odpowiada zestawowi konfiguracji, tworzenie nowego nie powinno przysporzyć
żadnych problemów.

Proszę rozważyć sytuację, kiedy to przed wdrożeniem aplikacji należy wykonać test
wydajności. Jeden ze sposobów to test aplikacji z ustawieniami zbliżonymi do
tych z produkcji, lecz z włączonym ``web_profiler``, co umożliwia Symfony2
rejestrowanie informacji o aplikacji podczas jej testów wydajnościowych.

Najlepsze rozwiązanie to utworzenie nowej nazwy środowiska, na przykład ``benchmark``.
W tym celu proszę rozpocząć od wygenerowania pliku konfiguracyjnego:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_benchmark.yml
        imports:
            - { resource: config_prod.yml }

        framework:
            profiler: { only_exceptions: false }

    .. code-block:: xml

        <!-- app/config/config_benchmark.xml -->
        <imports>
            <import resource="config_prod.xml" />
        </imports>

        <framework:config>
            <framework:profiler only-exceptions="false" />
        </framework:config>

    .. code-block:: php

        // app/config/config_benchmark.php
        $loader->import('config_prod.php')

        $container->loadFromExtension('framework', array(
            'profiler' => array('only-exceptions' => false),
        ));

Dzięki temu prostemu dodatkowi, aplikacja obsługuje teraz nowe środowisko o
nazwie ``benchmark``.

Nowy plik konfiguracyjny importuje konfiguracje ze środowiska ``prod``, po
czym nieznacznie ją modyfikuje. Daje to gwarancje, że nowe środowisko będzie
identyczne ze środowiskiem ``prod``, za wyjątkiem wszelkich zmian wyrażnie
dokonanych w pliku konfiguracyjnym ``benchmark``.

Ponieważ to środowisko powinno być dostępne przez przeglądarkę, należy dla
niego utworzyć kontroler wejściowy. Proszę skopiować plik ``web/app.php``
do ``web/app_benchmark.php``, a następnie zmienić nazwę środowiska na ``benchmark``:

.. code-block:: php

    <?php

    require_once __DIR__.'/../app/bootstrap.php';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('benchmark', false);
    $kernel->handle(Request::createFromGlobals())->send();

Nowo utworzone środowisko jest teraz dostępne przez::

    http://localhost/app_benchmark.php

.. note::

   Niektóre środowiska, takie jak ``dev``, nigdy nie powinny zostać udostępnione
   ogółowi społeczeństwa na którymkolwiek serwerze, gdzie nastąpiło wdrożenie.
   Jest tak dlatego, gdyż niektóre środowiska, w celach debugowania, mogą prezentować
   zbyt wiele informacji o aplikacji jak i jej infrastrukturze. Aby upewnić się, że
   żadne z tych środowisk nie jest dostępne publicznie, kontroler wejściowy jest
   zazwyczaj zabezpieczony przed dostępem z zewnętrznych adresów IP poprzez
   następujący kod na początku kontrolera:

    .. code-block:: php

        if (!in_array(@$_SERVER['REMOTE_ADDR'], array('127.0.0.1', '::1'))) {
            die('Nie posiadasz dostępu do tego pliku. Sprawdź '.basename(__FILE__).', aby uzyskać więcej informacji.');
        }

.. index::
   single: środowiska; katalog pamięci podręcznej

Środowiska i katalog pamięci podręcznej
---------------------------------------

Symfony2 wykorzystuje buforowanie na wiele sposobów: konfiguracja aplikacji,
konfiguracja trasowania, szablony Twig jak i wiele innych zostają zamieniane na
obiekty PHP i przechowywane w plikach w globalnym systemie plików.

Zazwyczaj te zbuforowane pliki są w dużej mierze przechowywane w katalogu ``app/cache``.
Proszę pamiętać, że każde środowisko tworzy swój odrębny zestaw plików i katalogów:

.. code-block:: text

    app/cache/dev   - katalog dla środowiska *dev*
    app/cache/prod  - katalog dla środowiska *prod*

Czasami, podczas debugowania, może być pomocne, aby sprawdzić pliki pamięci
podręcznej w celu zrozumienia jak coś działa. Należy przy tym pamiętać, aby przeglądać
katalog środowiska, które jest aktualnie używane (najczęściej ``dev`` podczas
rozwoju lub debugowania aplikacji). Choć może ulec to zmianie, katalog ``app/cache/dev``
zawiera następujące elementy:

* ``appDevDebugProjectContainer.php`` - "pojemnik usług", który reprezentuje
  zbuforowaną konfigurację aplikacji;

* ``appdevUrlGenerator.php`` - klasa PHP generowana z konfiguracji trasowania
  używana podczas generowania adresów URL;

* ``appdevUrlMatcher.php`` - klasa PHP używana do dopasowywania tras - warto
  tu zajrzeć, aby zobaczyć logikę skompilowanych wyrażeń regularnych, które
  odpowiadają za dopasowania przychodzących adresów URL do różnych tras;

* ``twig/`` - ten katalog zawiera wszystkie zbuforowane szablony Twig.

.. note::

    Można łatwo zmienić lokalizację katalogu i nazwę. Aby uzyskać więcej
    informacji, proszę przeczytać artykuł :doc:`/cookbook/configuration/override_dir_structure`.


Co dalej
--------


Proszę przeczytać artykuł :doc:`/cookbook/configuration/external_parameters`.
