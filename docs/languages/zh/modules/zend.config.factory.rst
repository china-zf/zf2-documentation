.. _zend.config.factory:

工厂方法
=======

工厂方法给予了导入一个配置文件到 ``Zend\Config\Config`` 的功能.
工厂方式有两个目的

- 导入配置文件
- 存储一个配置文件

.. note::

   存储配置到一个文件. 工厂方法是不会将合并的配置信息存储到多个文件中的.如果你需要存储到多个文件中去需要你自己手动操作.

导入配置文件
----------

接下来举例说明如何加载一个单独的配置文件.

.. code-block:: php
   :linenos:
   
   //Load a php file as array
   $config = Zend\Config\Factory::fromFile(__DIR__.'/config/my.config.php');

   //Load a xml file as Config object
   $config = Zend\Config\Factory::fromFile(__DIR__.'/config/my.config.xml', true);

合并多个配置文件

.. code-block::php
   :linenos:

    $config = Zend\Config\Factory::fromFiles(
        array(
            __DIR__.'/config/my.config.php',
            __DIR__.'/config/my.config.xml',
        )
    );

存储配置文件
----------

有时候你需要存储配置文件到文件中去, 在这里你可以轻松的实现.

.. code-block::php
   :linenos:
   
   $config = new Zend\Config\Config(array(), true);
   $config->settings = array();
   $config->settings->myname = 'framework';
   $config->settings->date	 = '2012-12-12 12:12:12';
   
   //Store the configuration
   Zend\Config\Factory::toFile(__DIR__.'/config/my.config.php', $config);
   
   //Store an array
   $config = array(
       'settings' => array(
           'myname' => 'framework',
           'data'   => '2012-12-12 12:12:12',
       ),
    );

    Zend\Config\Factory::toFile(__DIR__.'/config/my.config.php', $config);

	
