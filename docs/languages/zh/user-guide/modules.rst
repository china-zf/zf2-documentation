.. _user-guide.modules:

模块
=======

Zend Framework 2 使用一个模块系统在每个模块去组织你的主应用代码.提供的应用程序模块框架用于提供引导,错误和路由配置对整个应用程序. 
它通常用于提供应用程序级别的控制器,应用程序的主页, 但是我们不会使用本教程中提供的默认的，我们想要在我们的模块中构建我们的专辑列表主页.

我们将要把包含有控制器，表单，视图，单独的配置的所有代码放进专辑模块中. 我们也会根据需要调整我们的应用程序.

让我们先从需要的目录开始.

设置相册模块
---------------------------

在``module``目录下创建一个 ``Album`` 文件夹，用下面的子目录来保存模块中的文件:

.. code-block:: text
   :linenos:

    zf2-tutorial/
        /module
            /Album
                /config
                /src
                    /Album
                        /Controller
                        /Form
                        /Model
                /view
                    /album
                        /album

正如你看到的 ``Album`` 模块下有不同的目录来保存不同类型的文件. 在``src/Album``目录下，有Album的命名空间里面存放着php的类文件，
如果有需要我们可以在这个模型下有多个命名空间. 视图目录下也有一个叫做``album``的子目录来存放模块的视图文件.

为了加载和配置一个模块, Zend Framework 2 有一个叫做
``ModuleManager``. 这将在模块的根目录(``module/Album``)下寻找 ``Module.php``  期望在这个文件中能找到 ``Album\Module``类. 
也就是说, 在模块的类中将有一个用模块目录名命名的命名空间.

为``Album``模块创建 ``Module.php``文件:
在``zf2-tutorial/module/Album``目录下创建一个名称为 ``Module.php`` 的文件 :

.. code-block:: php
   :linenos:

    namespace Album;

    use Zend\ModuleManager\Feature\AutoloaderProviderInterface;
    use Zend\ModuleManager\Feature\ConfigProviderInterface;

    class Module implements AutoloaderProviderInterface, ConfigProviderInterface
    {
        public function getAutoloaderConfig()
        {
            return array(
                'Zend\Loader\ClassMapAutoloader' => array(
                    __DIR__ . '/autoload_classmap.php',
                ),
                'Zend\Loader\StandardAutoloader' => array(
                    'namespaces' => array(
                        __NAMESPACE__ => __DIR__ . '/src/' . __NAMESPACE__,
                    ),
                ),
            );
        }

        public function getConfig()
        {
            return include __DIR__ . '/config/module.config.php';
        }
    }

 ``ModuleManager`` 将会为我们自动调用 ``getAutoloaderConfig()`` 和 ``getConfig()``.

Autoloading files
^^^^^^^^^^^^^^^^^

Our ``getAutoloaderConfig()`` method returns an array that is compatible with
ZF2’s ``AutoloaderFactory``. We configure it so that we add a class map file to
the ``ClassMapAutoloader`` and also add this module’s namespace to the
``StandardAutoloader``. The standard autoloader requires a namespace and the
path where to find the files for that namespace. It is PSR-0 compliant and so
classes map directly to files as per the `PSR-0 rules
<https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>`_.

As we are in development, we don’t need to load files via the classmap, so we provide an empty array for the
classmap autoloader. Create a file called ``autoload_classmap.php`` under ``zf2-tutorial/module/Album``:

.. code-block:: php
   :linenos:

    return array();

As this is an empty array, whenever the autoloader looks for a class within the
``Album`` namespace, it will fall back to the to ``StandardAutoloader`` for us.

.. note::

    If you are using Composer, you could instead just create an empty
    ``getAutoloaderConfig() { }`` and add to composer.json:

    .. code-block:: javascript
       :linenos:

        "autoload": {
            "psr-0": { "Album": "module/Album/src/" }
        },

    If you go this way, then you need to run ``php composer.phar update`` to update 
    the composer autoloading files.

Configuration
-------------

Having registered the autoloader, let’s have a quick look at the ``getConfig()``
method in ``Album\Module``.  This method simply loads the
``config/module.config.php`` file.

Create a file called ``module.config.php`` under ``zf2-tutorial/module/Album/config``:

.. code-block:: php
   :linenos:

    return array(
        'controllers' => array(
            'invokables' => array(
                'Album\Controller\Album' => 'Album\Controller\AlbumController',
            ),
        ),
        'view_manager' => array(
            'template_path_stack' => array(
                'album' => __DIR__ . '/../view',
            ),
        ),
    );

The config information is passed to the relevant components by the
``ServiceManager``.  We need two initial sections: ``controllers`` and
``view_manager``. The controllers section provides a list of all the controllers
provided by the module. We will need one controller, ``AlbumController``, which
we’ll reference as ``Album\Controller\Album``. The controller key must
be unique across all modules, so we prefix it with our module name.

Within the ``view_manager`` section, we add our view directory to the
``TemplatePathStack`` configuration. This will allow it to find the view scripts for
the ``Album`` module that are stored in our ``view/`` directory.

Informing the application about our new module
----------------------------------------------

We now need to tell the ``ModuleManager`` that this new module exists. This is done
in the application’s ``config/application.config.php`` file which is provided by the
skeleton application. Update this file so that its ``modules`` section contains the
``Album`` module as well, so the file now looks like this:

(Changes required are highlighted using comments.)

.. code-block:: php
   :linenos:
   :emphasize-lines: 4

    return array(
        'modules' => array(
            'Application',
            'Album',                  // <-- Add this line
        ),
        'module_listener_options' => array(
            'config_glob_paths'    => array(
                'config/autoload/{,*.}{global,local}.php',
            ),
            'module_paths' => array(
                './module',
                './vendor',
            ),
        ),
    );

As you can see, we have added our ``Album`` module into the list of modules
after the ``Application`` module.

We have now set up the module ready for putting our custom code into it.
