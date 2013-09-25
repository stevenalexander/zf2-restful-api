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
5. public/.htaccess (for redirecting non-asset requests to index.php)

    ```
    RewriteEngine On
    # The following rule tells Apache that if the requested filename
    # exists, simply serve it.
    RewriteCond %{REQUEST_FILENAME} -s [OR]
    RewriteCond %{REQUEST_FILENAME} -l [OR]
    RewriteCond %{REQUEST_FILENAME} -d
    RewriteRule ^.*$ - [NC,L]
    # The following rewrites all other queries to index.php. The
    # condition ensures that if you are using Apache aliases to do
    # mass virtual hosting, the base path will be prepended to
    # allow proper resolution of the index.php file; it will work
    # in non-aliased environments as well, providing a safe, one-size
    # fits all solution.
    RewriteCond %{REQUEST_URI}::$1 ^(/.+)(.+)::\2$
    RewriteRule ^(.*) - [E=BASE:%1]
    RewriteRule ^(.*)$ %{ENV:BASE}index.php [NC,L]
    ```

6. init_autoloader.php (for loading Zend)

    ```
    <?php

    $loader = include 'vendor/autoload.php';
    $zf2Path = 'vendor/zendframework/zendframework/library';
    $loader->add('Zend', $zf2Path);

    if (!class_exists('Zend\Loader\AutoloaderFactory')) {
        throw new RuntimeException('Unable to load ZF2. Run `php composer.phar install` or define a ZF2_PATH environment variable.');
    }
    ```

7. config/application.config.php (application wide configuration)

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

8. module/AlbumApi/Module.php (module setup)

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

9. module/AlbumApi/config/module.config.php (module configuration)

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

10. module/AlbumApi/src/AlbumApi/Controller/IndexController.php (basic RESTful controller)

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

11. You should now be able to request the API URL and receive a JSON response with the welcome message

12. module/AlbumApi/src/AlbumApi/Controller/AlbumController.php (album controller with CRUD REST actions)

    ```
    <?php
    namespace AlbumApi\Controller;

    use Zend\Mvc\Controller\AbstractRestfulController;
    use Zend\View\Model\JsonModel;

    class AlbumController extends AbstractRestfulController
    {
        public function getList()
        {   // Action used for GET requests without resource Id
            return new JsonModel(
                array('data' =>
                    array(
                        array('id'=> 1, 'name' => 'Mothership', 'band' => 'Led Zeppelin'),
                        array('id'=> 2, 'name' => 'Coda',       'band' => 'Led Zeppelin'),
                    )
                )
            );
        }

        public function get($id)
        {   // Action used for GET requests with resource Id
            return new JsonModel(array("data" => array('id'=> 2, 'name' => 'Coda', 'band' => 'Led Zeppelin')));
        }

        public function create($data)
        {   // Action used for POST requests
            return new JsonModel(array('data' => array('id'=> 3, 'name' => 'New Album', 'band' => 'New Band')));
        }

        public function update($id, $data)
        {   // Action used for PUT requests
            return new JsonModel(array('data' => array('id'=> 3, 'name' => 'Updated Album', 'band' => 'Updated Band')));
        }

        public function delete($id)
        {   // Action used for DELETE requests
            return new JsonModel(array('data' => 'album id 3 deleted'));
        }
    }
    ```

13. Update module/AlbumApi/config/module.config.php to add controller and routing

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
                'album' => array(
                    'type'    => 'segment',
                    'options' => array(
                        'route'    => '/album[/:id]',
                        'constraints' => array(
                            'id'     => '[0-9]+',
                        ),
                        'defaults' => array(
                            'controller' => 'AlbumApi\Controller\Album',
                        ),
                    ),
                ),
            ),
        ),
        'controllers' => array(
            'invokables' => array(
                'AlbumApi\Controller\Index' => 'AlbumApi\Controller\IndexController',
                'AlbumApi\Controller\Album' => 'AlbumApi\Controller\AlbumController',
            ),
        ),
        'view_manager' => array(
            'strategies' => array(
                'ViewJsonStrategy',
            ),
        ),
    );
    ```

14. You can now make specific HTTP requests to the album resource to interact with it in a RESTful manner, e.g. GET /album to see the list of albums, PUT /album/3 to update album 3. Use a REST client, like Chromes Postman to test it out

## Tricks and traps

### Error handling for application exceptions and 404s

If your application experiences an error or can't find a requested resource you want to return a meaningful response. Currently if an error occurs you'll get a Zend\View\Exception\RuntimeException 'Unable to render template "error"', cause it can't find the html error view. To fix this you need to hook into the ZF2 eventmanager events and intercept errors, returning an appropiate response.

You do this in the module Module.php file:

```
<?php

namespace AlbumApi;

use Zend\Mvc\ModuleRouteListener;
use Zend\Mvc\MvcEvent;
use Zend\View\Model\JsonModel;

class Module
{
    public function onBootstrap(MvcEvent $e)
    {
        $eventManager        = $e->getApplication()->getEventManager();
        $moduleRouteListener = new ModuleRouteListener();
        $moduleRouteListener->attach($eventManager);

        $eventManager->attach(MvcEvent::EVENT_DISPATCH_ERROR, array($this, 'onDispatchError'), 0);
        $eventManager->attach(MvcEvent::EVENT_RENDER_ERROR, array($this, 'onRenderError'), 0);
    }

    public function onDispatchError($e)
    {
        return $this->getJsonModelError($e);
    }

    public function onRenderError($e)
    {
        return $this->getJsonModelError($e);
    }

    public function getJsonModelError($e)
    {
        $error = $e->getError();
        if (!$error) {
            return;
        }

        $response = $e->getResponse();
        $exception = $e->getParam('exception');
        $exceptionJson = array();
        if ($exception) {
            $exceptionJson = array(
                'class' => get_class($exception),
                'file' => $exception->getFile(),
                'line' => $exception->getLine(),
                'message' => $exception->getMessage(),
                'stacktrace' => $exception->getTraceAsString()
            );
        }

        $errorJson = array(
            'message'   => 'An error occurred during execution; please try again later.',
            'error'     => $error,
            'exception' => $exceptionJson,
        );
        if ($error == 'error-router-no-match') {
            $errorJson['message'] = 'Resource not found.';
        }

        $model = new JsonModel(array('errors' => array($errorJson)));

        $e->setResult($model);

        return $model;
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

You can extend this approach to return error responses with specific [HTTP status codes](http://www.restapitutorial.com/httpstatuscodes.html) for exceptions by changing the response status code based on the exception (custom exceptions with appropiate codes/messages).