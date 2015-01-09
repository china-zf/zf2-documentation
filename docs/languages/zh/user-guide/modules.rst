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

在ZF2中我们使用 ``getAutoloaderConfig()``方法兼容返回一个``AutoloaderFactory``数组 .我们配置配置文件添加一个类的列表到``ClassMapAutoloader``,配置模块的命名空间到``StandardAutoloader``
 
.当需要的文件在命名空间下的时候标准加载会引入一个命名空间路径. 这些都是符合``psr-0``规范的
<https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>`_.

正如发展中, 我们不需要通过classmap加载文件, 所以我们在classmap提供一个空数组自动加载 创建一个叫做 ``autoload_classmap.php``文件在 ``zf2-tutorial/module/Album``下:

.. code-block:: php
   :linenos:

    return array();

因为这是一个空数组, 当 autoloader 寻找``Album`` 命名空间下的类的时候
, 他将为我们返回 ``StandardAutoloader``.

.. 注意::

    如果你使用Composer, 你可以创建一个空的``getAutoloaderConfig() { }`` 之后添加在 composer.json中:

    .. code-block:: javascript
       :linenos:

        "autoload": {
            "psr-0": { "Album": "module/Album/src/" }
        },

    如果你使用这种方法, 你需要运行 ``php composer.phar update`` 来更新  composer 自动加载文件.

配置
-------------

已经注册了 autoloader, 让我们快速看看在``Album\Module``中的 ``getConfig()``方法.  这个方法简单的加载了
``config/module.config.php`` 文件.

在 ``zf2-tutorial/module/Album/config`` 下创建一个叫做``module.config.php`` 的文件 :

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

配置信息通过``ServiceManager``传递给相关组件.  我们需要初始部分: ``controllers`` 和``view_manager``. 
 controllers 部分 提供一个控制器列表. 我们需要一个, ``AlbumController``,引用为``Album\Controller\Album``的控制器. 控制器的键在所有的模型中必须唯一
, 所以我们用模块名为它的前缀.

在 ``view_manager`` 部分, 我们添加我们的视图目录到``TemplatePathStack`` 配置中. 他将允许我们找到存储在``view/``目录下的``Album``模块的视图文件.

为我们的新的模块通知应用程序
----------------------------------------------

现在我们需要告诉 ``ModuleManager`` 有一个新的模块存在了. 这个在应用程序中的``config/application.config.php``来完成它. 
更新这个文件，以便``modules``部分能够包含``Album``模块，现在这个文件看起来应该是这样:

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

正如你看到的, 我们已经在模块列表添加了 ``Album`` 模块在``Application``之后 .

现在我们可以把我们的代码放在我们已经设置好的模块中了.
