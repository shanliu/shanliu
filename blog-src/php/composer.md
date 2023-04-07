# composer

1.  常用： composer.json

    ```
    {
       "name": "lonely/system",
       "description": "lonely",
       "version": "0.1",
       "require": {
           "php": ">=5.5.0",
           "ext-yaf": ">=2.0.0",
           "monolog/monolog": "1.19.0",
           "phpdr.net/php-yaf-doc":"dev-master",
           "phpunit/phpunit":"4.*",
           "lox/xhprof":"dev-master",
           "firephp/firephp-core":"dev-master",
           "foolz/profiler":"dev-master"
       },
       "require-dev": {
           "php": ">=5.6.0"
       },
       "license": "MIT",
       "autoload": {
           "psr-4": {
               "SL\\": "src/SL"
           }
       },
       "authors": [
           {
               "name": "lonely",
               "email": "shan.liu@msn.com"
           }
       ],
       "support": {
           "email": "shan.liu@msn.com",
           "irc": "irc://xxxx"
       }
    }
    ```
2.  使用本地SVN库

    > * repositories 的 url 中需要有: composer.json
    > * 使用 trunk 的版本为: dev-trunk
    > * 如果只有 trunk 版本,minimum-stability 设置为: dev
    > * SVN未使用 https ，secure-http 需设置为: false

    ```
    {
       "name": "lonely/aaa",
       "description": "lonely",
       "version": "0.2",
       "config": {
           "secure-http":false,
           "process-timeout":30
       },
       "repositories": [
           {
               "type": "vcs",
               "url": "http://127.0.0.1/svnroot/lonely/sl/",
               "trunk-path": "trunk",
               "branches-path": "branches",
               "tags-path": "tags",
               "svn-cache-credentials": false
           },
           {
               "type": "git",
               "url": "git://github.com/firephp/firephp-core.git"
           }
       ],
       "http-basic": {
         "127.0.0.1": {
             "username": "username",
             "password": "password"
         }
       },
       "require": {
           "lonely/system": "dev-trunk"
       },
       "license": "MIT",
       "authors": [
           {
               "name": "lonely",
               "email": "shan.liu@msn.com"
           }
       ],
       "minimum-stability":"dev"
    }
    ```
3.  autoload

    > files 加载指定文件

    ```
    "autoload": {
       "files": ["src/MyLibrary/functions.php"]
    }
    ```

    > classmap 扫描指定目录下以.php或.inc结尾的文件或制定文件文件中的class ，在autoload\_classmap.php生成 class 与文件的关系

    ```
    {
     "autoload": {
         "classmap": ["src/", "lib/", "Something.php"]
     }
    }
    ```

    > PSR-0 按下划线或命名空间加载，参考 Zend 加载

    ```
    {
     "autoload": {
         "psr-0": {
             "SL\\": "src/SL"
         }
     }
    }
    ```

    > PSR-4 按命名空间加载，类似 PSR-0 但不会把下划线转为目录

    ```
    {
     "autoload": {
         "psr-0": {
             "SL\\": "src/SL"
         }
     }
    }
    ```

    > PSR-0 PSR-4 "SL\\": "src/SL" 表示在 src/SL 目录中的PHP文件内容大体如下示例,如A.php内容为:

    ```
    <?php
    namespace SL;
    class A{};
    ```
4.  require

    > * require 依赖是扩展，可以通过以下命令得到支持的扩展

    ```
    composer show --platform
    ```
5.  版本控制中去除 vendor 目录

    ```
    svn propedit svn:ignore .
    #添加以下目录和文件
    composer.lock
    vendor
    ```
