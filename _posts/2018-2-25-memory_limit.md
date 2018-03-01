在运行PHP程序，通常会遇到“Fatal Error: Allowed memory size of xxxxxx bytes exhausted”的错误, 这个意味着PHP脚本使用了过多的内存，并超出了系统对其设置的允许最大内存。解决这个问题，首先需要查看你的程序是否分配了过多的内存，在程序没有问题的情况下，你可以通过一下方法来增加PHP的内存限制（memory_limit)。

检查php的内存限制值

为了查看这个值，你需要建立一个空的php文件，比如view-php-info.php。然后将一下代码贴到里面。

<?php phpinfo(); ?>
将这个脚本放到你的PHP主机上，然后在浏览器中调用它。这时你可以看到你的PHP环境配置的信息，其中有一部分是关于"memory_limit"的, 如下图:

view-php-memory-limit

注：你可以用这种方法来查看php的其他参数设置，不仅仅是memory_limit

memory_limit应该设为多少？

这个完全依赖于你的应用的要求。比如Wordpress，运行起核心代码需要32MB。Drupal 6则要求这个值最小为16MB，并推荐设置为32MB。如果你又安装不少的插件（plugins），尤其是那些要进行图像处理的模块，那么你可能需要128MB或更高的内存。

如何设置memory_limit

方法1: php.ini

最简单或常用的方法是修改php.ini

首先找到对你的网站生效的php.ini文件
由于有多个地方都可以设置php的参数，找到正确的配置文件，并进行更改是首先要做的一步。如果你上面的方法建立了php文件来查看其配置参数，则你可以找到"Loaded Configuration File"这一项，以下是个例子：

php-ini-location

对于Linux用户，你可以通过执行"php -i | grep Loaded Configuration File"来找到对应的配置文件。而Windows用户，你可以尝试修改你的php安装目录下的php.ini。

编辑php.ini
在php.ini中，找到"memory_limit"这一项，如果没有，你可以在文件的尾部自己增加这个参数。以下是一些设置范例

memory_limit = 128M ; 可以将128M改为任何你想设置的值
保存文件

重启web 服务器
如果是web服务器使用Apache, 则执行:

httpd restart
有些情况下，你可能不被允许私修改php.ini。比如如果你购买了虚拟主机服务，但是你的服务商确禁止你修改这个文件。那么，你可以需要考虑用其他方法来增加memory_limit的值。

方法2: .htaccess

说明: 这种方法只有在php以Apache模块来执行时才生效.

在你的网站的根目录下找到".htaccess"文件，如果没有，可以自己创建一个。然后把以下配置放入其中

php_value memory_limit 128M ; 可以将128M改为任何你想设置的值
方法3: 运行时修改php的内存设置

在你的php代码中增加以下命令行即可。

ini_set('memory_limit', '128M');
memory_limit修改失败

如果你使用虚拟主机，有可能会出现memory_limit的值修改失败。这个需要联系你的服务商看怎么处理，通常他们限制了可以设置的最大值或者根本就不允许你修改。如果他们的环境真的无法满足你的要求，那么你可能要考虑换一个主机服务商。


查看php当前占用内存量:

PHP性能优化过程中需要获取PHP内存消耗，使用memory_get_usage()函数可获取当前的内存消耗情况，函数使用简单，这里讨论一下memory_get_usage()函数的用法与实例
三，基础用法与实例 
1，获取当前的内存消耗量 
复制代码代码如下:
<?php 
echo memory_get_usage(); 
$var = str_repeat("liuhui", 10000); 
echo memory_get_usage(); 
unset($var); 
echo memory_get_usage(); 
?> 
分别输出：62328 122504 62416 
说明：memory_get_usage()函数输出的数值为bytes单位 
2，格式化memory_get_usage()输出 
复制代码代码如下:
<?php 
function convert($size){ 
$unit=array('b','kb','mb','gb','tb','pb'); 
return @round($size/pow(1024,($i=floor(log($size,1024)))),2).' '.$unit[$i]; 
} 
echo convert(memory_get_usage(true)); 
?> 
输出：256 kb 
3，自定义函数获取数组或变量值大小 
复制代码代码如下:
<?php 
function array_size($arr) { 
ob_start(); 
print_r($arr); 
$mem = ob_get_contents(); 
ob_end_clean(); 
$mem = preg_replace("/\n +/", "", $mem); 
$mem = strlen($mem); 
return $mem; 
} 
$memEstimate = array_size($GLOBALS); 
?>

PHP memory_get_usage()管理内存

我们在实际编码中，要想实现对内存的查看和操作，许多程序员们第一个想到的就是PHP memory_get_usage()这个PHP脚本内存函数。

下面是PHP memory_get_usage()使用示例：
echo memory_get_usage(), '<br />'; // 313864  
$tmp = str_repeat('http://blog.huachen.me/', 4000);  
echo memory_get_usage(), '<br />'; // 406048  
unset($tmp);  
echo memory_get_usage(); // 313952 
上面的程序后面的注释代表了它们的输出（单位为 byte(s)），也就是当时 PHP 脚本使用的内存（不含 memory_get_usage() 函数本身占用的内存）

由上面的例子可以看出，要想减少内存的占用，可以使用 PHP unset() 函数把不再需要使用的变量删除。类似的还有：PHP mysql_free_result() 函数，可以清空不再需要的查询数据库得到的结果集，这样也能得到更多可用内存。

PHP memory_get_usage()还可以有个参数，$real_usage，其值为布尔值。默认为 FALSE，表示得到的内存使用量不包括该函数（PHP 内存管理器）占用的内存；当设置为 TRUE 时，得到的内存为不包括该函数（PHP 内存管理器）占用的内存。

所以在实际编程中，可以用PHP memory_get_usage()比较各个方法占用内存的高低，来选择使用哪种占用内存小的方法。

常用的检测：

用microtime函数就可以分析程序执行时间
memory_get_usage可以分析内存占用空间 
SQL的效率可以使用打开慢查询查看日志分析
SQL 找到有瓶颈的使用EXPLAIN 来分析

当你在做一个抓取程序的时候，php空白了好长一段时间然后报出现如下这个错误提示：Fatal error: Maximum execution time of 30 seconds exceeded in ......很简单，意思是说脚本执行时间超过了30秒的上限。这个错误以前经常碰到，一般都是直接在页面头部加个 set_time_limit(0) 处理，今天特意将这个错误的处理方法做一下总结。经过查阅相关资料，对于PHP入门教程的学者们处理这个错误的方法基本上有三种。
 
 
（1）修改php的配置文件 php.ini 文件
找到 php.ini 这个文件，然后在这个文件中找到：max_execution_time = 30 ;这一行，将数字 30 设置成你想要的值，单位是秒。（也可以直接修改为：max_execution_time=0;//无限制）注意这样修改完后需要重启一下服务器。
 
（2）使用 ini_set() 函数
对于那些不能够修改 php.ini 的朋友来说，你可以使用ini_set()这个函数来改变你的最大执行时间限制值，在程序的顶部加入如下代码：ini_set('max_execution_time','100');以上设置的为100秒，你也可以设置为0，那么就是不限制执行的时间。
 
（3）使用set_time_limit() 函数
在程序的顶部加入：set_time_limit(100);则表示最大执行时间设置为了100秒，当然也可以将参数设置为0，意思同上。set_time_limit 函数特别说明：void set_time_limit ( int $seconds ) 该函数的作用是设置允许脚本运行的时间，单位为秒。如果超过了此设置，脚本返回一个致命的错误。默认值为30秒，或者是在php.ini的max_execution_time被定义的值，如果此值存在。当此函数被调用时， set_time_limit()会从零开始重新启动超时计数器。换句话说，如果超时默认是30秒，在脚本运行了了25秒时调用 set_time_limit(20)，那么，脚本在超时之前可运行总时间为45秒。当php运行于安全模式下时，此功能不能生效。除了关闭安全模式（在 php.ini 中将 safe_mode 设置为 off）或改变 php.ini 中的时间限制，没有别的办法。案例：如果没有打开安全模式，设置程序运行时间为25秒。例如：
 
12if(!ini_get('safe_mode')){
3    set_time_limit(25);
4}

php安全模式：safe_mode=on|off
启用safe_mode指令将对在共享环境中使用PHP时可能有危险的语言特性有所限制。可以将safe_mode是指为布尔值on来启用，或者设置为 off禁用。它会比较执行脚本UID（用户ID）和脚本尝试访问的文件的UID，以此作为限制机制的基础。如果UID相同，则执行脚本；否则，脚本失败。
具体地，当启用安全模式时，一些限制将生效。
1、所有输入输出函数（例如fopen()、file()和require()）的适用会受到限制，只能用于与调用这些函数的脚本有相同拥有者的文件。例如，假定启用了安全模式，如果Mary拥有的脚本调用fopen(),尝试打开由Jonhn拥有的一个文件，则将失败。但是，如果Mary不仅拥有调用 fopen()的脚本，还拥有fopen()所调用的文件，就会成功。
2、如果试图通过函数popen()、system()或exec()等执行脚本，只有当脚本位于safe_mode_exec_dir配置指令指定的目录才可能。
3、HTTP验证得到进一步加强，因为验证脚本用于者的UID划入验证领域范围内。此外，当启用安全模式时，不会设置PHP_AUTH。
4、如果适用MySQL数据库服务器，链接MySQL服务器所用的用户名必须与调用mysql_connect()的文件拥有者用户名相同。
安全模式和禁用的函数
下面是启用safe_mode指令时受影响的函数、变量及配置指令的完整列表：
apache_request_headers() backticks()和反引号操作符 chdir()
chgrp()     chmode()    chown()
copy()     dbase_open()    dbmopen()
dl()     exec()     filepro()
filepro_retrieve()   filepro_rowcount()   fopen()
header()    highlight_file()   ifx_*
ingres_*    link()     mail()
max_execution_time()   mkdir()     move_uploaded_file()
mysql_*     parse_ini_file()   passthru()
pg_lo_import()    popen()     posix_mkfifo()
putenv()    rename()    zmdir()
set_time_limit()   shell_exec()    show_source()
symlink()    system()    touch()

以下是一些和安全模式相关的配置选项
safe_mode_gid=on|off
次指令会修改安全模式的行为，即从执行前验证UID改为验证组ID。例如，如果Mary和John处于相同的用户组，则Mary的脚本可以对John的文件调用fopen()。
safe_mode_include_dir=string
可以使用指令safe_mode_include_dir指示多个路径，启用安全模式时在这些路径中将忽略安全模式。例如，你可以使用此函数指定一个包含不同模板的目录，致谢模板可能会继成到一些用户网站。可以指定多个目录，在基于UNIX的系统各目录用冒号分隔，在Windows中用分号分隔。
注意，如果指定某个路径但未包含最后的斜线，则该路径下的所有目录都会忽略安全模式设置。例如，如果设置次指令为/home /configuration，表示/home/configuration/templates/和/home/configureation /passwords都排除在安全模式限制之外。因此，如果只是要排除一个目录或一组目录不受安全模式设置的限制，要确保每个目录都包括最后的斜线。
safe_mode_env_vars=string
当启用安全模式时，可以只用次指令允许执行用户的脚本修改某些环境变量。可以允许修改多个变量，每个变量之间用逗号分隔。
safe_mode_exec_dir=string
次指令指定一些目录，其中的系统程序可以通过诸如system()、exec()或passthru()等函数执行。为此必须启用安全模式。此指令有一个奇怪的地方，在所有操作系统中(包括Windows)，都必须使用斜线(/)作为目录的分隔符。
safe_mode_protected_env_vars=string
此指令保护某些环境变量不能被putenv()函数修改。默认情况下，变量LD_LIBRARY_PATH是受保护的，因为如果在运行时修改这个变量可能导致不可预知的结果。关于此环境变量的更多信息，请参考搜索引擎或Linux手册。注意，本届中声明的所有便来弄个都覆盖 safe_mode_allowed_env_vars指令中声明的变量。

 

另外，由于启用了安全模式后，由于会对比文件的拥有者和文件的执行者是否是一个人，所以会减慢执行效率。


 