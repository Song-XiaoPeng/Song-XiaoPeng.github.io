```
$ composer install
Loading composer repositories with package information
Updating dependencies (including require-dev)
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - Installation request for codeguy/upload ^1.3.2 -> satisfiable by codeguy/upload[1.3.2].
    - codeguy/upload 1.3.2 requires `ext-fileinfo` * -> the requested PHP extension fileinfo is missing from your system.

  To enable extensions, verify that they are enabled in your .ini files:
    - D:\amp\php\php-7.0.12-nts\php.ini
  You can also run `php --ini` inside terminal to see which files are used by PHP in CLI mode.
```
解决方法： 开启phpinfo扩展