# ZF2 Restful API Example

This is an example application showing how to create a RESTful JSON API using PHP and [Zend Framework 2](http://framework.zend.com/). It starts from a clean base rather than the skeleton app as that includes a load of files which are unnecessary for an API (language and html view files). Matches the [ZF2 Album example](http://framework.zend.com/manual/2.2/en/user-guide/overview.html) but without the DB logic for simplicity. Examples of how to test the application will be included in part 2.

## Requirements

* PHP 5.3+
* Web server [setup with virtual host to serve project folder](http://framework.zend.com/manual/2.2/en/user-guide/skeleton-application.html#virtual-host)
* [Composer](http://getcomposer.org/) (manage dependencies)

## Creating the API

1. Get composer:

    ```
    curl -sS https://getcomposer.org/installer | php
    ```

2. Create the composer.json file to get ZF2:

    ```
    {
        "require": {
            "php": ">=5.3.3",
            "zendframework/zendframework": ">=2.2.4"
        }
    }
    ```

3. Install the dependencies:

    ```
    php composer.phar install
    ```

4. public/index.php (for directing calls to Zend and static)

    ```
    <?php

    chdir(dirname(__DIR__));

    if (php_sapi_name() === 'cli-server' && is_file(__DIR__ . parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH))) {
        return false;
    }

    require 'init_autoloader.php';
    Zend\Mvc\Application::init(require 'config/application.config.php')->run();
    ```

5. init_autoloader.php (for loading Zend)

    ```
    <?php

    $loader = include 'vendor/autoload.php';
    $zf2Path = 'vendor/zendframework/zendframework/library';
    $loader->add('Zend', $zf2Path);

    if (!class_exists('Zend\Loader\AutoloaderFactory')) {
        throw new RuntimeException('Unable to load ZF2. Run `php composer.phar install` or define a ZF2_PATH environment variable.');
    }
    ```

6. config/application.config.php (application wide configuration)

    ```
    <?php
    return array(
        'modules' => array(
            'AlbumApi',
        ),
        'module_listener_options' => array(
            'module_paths' => array(
                './module',
                './vendor',
            ),
            // local/global config location when needed
            //'config_glob_paths' => array(
            //    'config/autoload/{,*.}{global,local}.php',
            //),
        ),
    );
    ```

7. module/AlbumApi/Module.php (module setup)

    ```
    <?php

    namespace AlbumApi;

    use Zend\Mvc\ModuleRouteListener;
    use Zend\Mvc\MvcEvent;

    class Module
    {
        public function onBootstrap(MvcEvent $e)
        {
            $eventManager        = $e->getApplication()->getEventManager();
            $moduleRouteListener = new ModuleRouteListener();
            $moduleRouteListener->attach($eventManager);
        }

        public function getConfig()
        {
            return include __DIR__ . '/config/module.config.php';
        }

        public function getAutoloaderConfig()
        {
            return array(
                'Zend\Loader\StandardAutoloader' => array(
                    'namespaces' => array(
                        __NAMESPACE__ => __DIR__ . '/src/' . __NAMESPACE__,
                    ),
                ),
            );
        }
    }

    ```

8. module/AlbumApi/config/module.config.php (module configuration)

    ```
    <?php

    return array(
        'router' => array(
            'routes' => array(
                'home' => array(
                    'type' => 'Zend\Mvc\Router\Http\Literal',
                    'options' => array(
                        'route'    => '/',
                        'defaults' => array(
                            'controller' => 'AlbumApi\Controller\Index',
                        ),
                    ),
                ),
            ),
        ),
        'controllers' => array(
            'invokables' => array(
                'AlbumApi\Controller\Index' => 'AlbumApi\Controller\IndexController'
            ),
        ),
        'view_manager' => array(
            'strategies' => array(
                'ViewJsonStrategy',
            ),
        ),
    );
    ```

9. module/AlbumApi/src/AlbumApi/Controller/IndexController.php (basic RESTful controller)

    ```
    <?php
    namespace AlbumApi\Controller;

    use Zend\Mvc\Controller\AbstractRestfulController;
    use Zend\View\Model\JsonModel;

    class IndexController extends AbstractRestfulController
    {
        public function getList()
        {
            return new JsonModel(array('data' => "Welcome to the Zend Framework Album API example"));
        }
    }
    ```
