.. _user-guide.routing-and-controllers:

路由和控制器
=======================

我们将创建一个非常简单的库存系统来显示我们的专辑. 主页将显示列表之后允许我们添加，编辑和删除. 因此需要以下的页面:

+---------------+------------------------------------------------------------+
| 页面          | 描述                                                       |
+===============+============================================================+
| 主页          | 这将显示相册的列表,并提供链接来编辑和删除它们。			 |
|	            |  启用链接添加新专辑也将提供                                |
+---------------+------------------------------------------------------------+
| 添加新专辑    | 这个页面将提供一个表单来添加一个新的专辑.                  |
+---------------+------------------------------------------------------------+
| 编辑专辑      | 这个页面将提供一个表单编辑一个专辑.                        |
+---------------+------------------------------------------------------------+
| 删除专辑      | 本页将确认我们想删除专辑和然后删除它                       |
+---------------+------------------------------------------------------------+

我们设置文件之前,重要的是要了解如何组织框架页面. 每个应用程序页面被叫做 is known as an*action*
这些action在 *modules*的*controllers*中 . 因此, 你需要把一般相关的action组织在一个controller中; 
例如, 一个新的控制器中有 ``current``, ``archived`` 和 ``view``.

由于我们在专辑中有四个页面, 我们将把他们放在``Album``模块中的一个控制器中`AlbumController``. 四个action将是:

+---------------+---------------------+------------+
| Page          | Controller          | Action     |
+===============+=====================+============+
| Home          | ``AlbumController`` | ``index``  |
+---------------+---------------------+------------+
| Add new album | ``AlbumController`` | ``add``    |
+---------------+---------------------+------------+
| Edit album    | ``AlbumController`` | ``edit``   |
+---------------+---------------------+------------+
| Delete album  | ``AlbumController`` | ``delete`` |
+---------------+---------------------+------------+

在模型的``module.config.php``文件中定义了URL到特定的action 的映射. 我们将为我们的 album
actions添加路由. 模型配置文件中的更新代码高亮显示.

.. code-block:: php
   :linenos:
   :emphasize-lines: 4,9-27

    return array(
        'controllers' => array(
            'invokables' => array(
                'Album\Controller\Album' => 'Album\Controller\AlbumController',
            ),
        ),

        // The following section is new and should be added to your file
        'router' => array(
            'routes' => array(
                'album' => array(
                    'type'    => 'segment',
                    'options' => array(
                        'route'    => '/album[/:action][/:id]',
                        'constraints' => array(
                            'action' => '[a-zA-Z][a-zA-Z0-9_-]*',
                            'id'     => '[0-9]+',
                        ),
                        'defaults' => array(
                            'controller' => 'Album\Controller\Album',
                            'action'     => 'index',
                        ),
                    ),
                ),
            ),
        ),

        'view_manager' => array(
            'template_path_stack' => array(
                'album' => __DIR__ . '/../view',
            ),
        ),
    );

名称为 ‘album’ 有一个 ‘segment’类型. segment 类型的路由
允许我们在URL中指定占位符pattern (route)参数将会映射到参数中. 因此, 路由
**``/album[/:action][/:id]``** 将会匹配以``/album``开始的 URL
. 下面一个部分将是一个可选的控制器名称的参数, 最后一个部分将被映射到一个可选的id. 
方括号表示这段是可选的. 让我们确保我们所有约束的字符在一个部分, constraints 部分我们已经限制了
actions由一个字母开始之后后续字符必须将是数字或者字母,. 我们也限制了id必须是数字.

路由允许我们有以下的 URLs:

+---------------------+------------------------------+------------+
| URL                 | Page                         | Action     |
+=====================+==============================+============+
| ``/album``          | Home (list of albums)        | ``index``  |
+---------------------+------------------------------+------------+
| ``/album/add``      | Add new album                | ``add``    |
+---------------------+------------------------------+------------+
| ``/album/edit/2``   | Edit album with an id of 2   | ``edit``   |
+---------------------+------------------------------+------------+
| ``/album/delete/4`` | Delete album with an id of 4 | ``delete`` |
+---------------------+------------------------------+------------+

创建一个控制器
=====================

现在,我们可以建立我们的控制器. 在 Zend Framework 2中, 控制器是一个类通常叫做 ``{Controller name}Controller``.
 注意``{Controller name}`` 必须以一个大写字母开头.  这个类在一个叫做 ``{Controller name}Controller.php``的文件中 ，在模块的 ``Controller`` 目录下 . 
在我们的例子中是 ``module/Album/src/Album/Controller``.控制器的类中每个action必须是一个公共的方法叫做 ``{action name}Action``.
 ``{action name}`` 必须以一个小写字母开始.

.. 注意::

    按照惯例. Zend Framework 2 没有很多的限制他们必须实现``Zend\Stdlib\Dispatchable``接口. 
	框架提供了两个抽象的类为我们这样做: ``Zend\Mvc\Controller\AbstractActionController``
    和 ``Zend\Mvc\Controller\AbstractRestfulController``. 我们将使用标准的 ``AbstractActionController``,
	如果你打算做RESTful服务, ``AbstractRestfulController`` 可能会用到.

让我们创建我们的控制器类 ``AlbumController.php`` 在 ``zf2-tutorials/module/Album/src/Album/Controller`` :

.. code-block:: php
   :linenos:

    namespace Album\Controller;

    use Zend\Mvc\Controller\AbstractActionController;
    use Zend\View\Model\ViewModel;

    class AlbumController extends AbstractActionController
    {
        public function indexAction()
        {
        }

        public function addAction()
        {
        }

        public function editAction()
        {
        }

        public function deleteAction()
        {
        }
    }
    
.. 注意::

    确保注册了新的模块``Album`` 在``config/application.config.php``文件中的 "modules" 部分
    . 你还需要提供一个 :ref:`Module Class
    <zend.module-manager.module-class>` 为专辑模块能被the MVC认识.

.. 注意::

    我们已经在``module/Album/config/module.config.php``中配置了我们的‘controller’部分 .

我们现在设置四个我们想使用的actions. 在设置视图之前他们将不会工作. 每个action的URLS是:

+------------------------------------------------+----------------------------------------------------+
| URL                                            | Method called                                      |
+================================================+====================================================+
| ``http://zf2-tutorial.localhost/album``        | ``Album\Controller\AlbumController::indexAction``  |
+------------------------------------------------+----------------------------------------------------+
| ``http://zf2-tutorial.localhost/album/add``    | ``Album\Controller\AlbumController::addAction``    |
+------------------------------------------------+----------------------------------------------------+
| ``http://zf2-tutorial.localhost/album/edit``   | ``Album\Controller\AlbumController::editAction``   |
+------------------------------------------------+----------------------------------------------------+
| ``http://zf2-tutorial.localhost/album/delete`` | ``Album\Controller\AlbumController::deleteAction`` |
+------------------------------------------------+----------------------------------------------------+

现在我们有一个工作中的路由和设置好了action的页面.

是时候建立视图和模型层.

初始化视图脚本
---------------------------

我们需要创建一些视图文件在我们的应用中. 这些文件将通过 ``DefaultViewStrategy``来执行 之后所有的变量将会被通过
 or view models that are returned from the controller action
method. These view scripts are stored in our module’s views directory within a
directory named after the controller. Create these four empty files now:

* ``module/Album/view/album/album/index.phtml``
* ``module/Album/view/album/album/add.phtml``
* ``module/Album/view/album/album/edit.phtml``
* ``module/Album/view/album/album/delete.phtml``

我们现在可以从我们的数据库和模型中开始写代码.
