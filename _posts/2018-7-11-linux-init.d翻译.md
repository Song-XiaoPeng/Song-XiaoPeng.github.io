
###	Configuration of System V init under Debian GNU/Linux

### ubuntu
Debian GNU/Linux System V 系统 init 脚本的配置

>Most Unix versions have a file here that describes how the scripts
in this directory work, and how the links in the /etc/rc?.d/ directories
influence system startup/shutdown.

大多数Unix版本在这里有一个文件，描述这些脚本如何在这个目录工作，以及在 /etc/rc?.d/ 目录里的链接如何影响系统的 启动/关机。

>For Debian, this information is contained in the policy manual, chapter 
"System run levels and init.d scripts".  The Debian Policy Manual is 
available at:

对于Debian来说，这个信息包含在政策手册的中，章节“系统运行级别和init.d脚本”。Debian官方手册在[这个网址](http://www.debian.org/doc/debian-policy/#contents)可找到

>    http://www.debian.org/doc/debian-policy/#contents

>The Debian Policy Manual is also available in the Debian package
"debian-policy".  When this package is installed, the policy manual can be
found in directory /usr/share/doc/debian-policy. If you have a browser
installed you can probably read it at

Debian官方手册也可以再Debian包中‘debian-policy’获得。当包安装后，官方手册可以在目录/usr/share/doc/debian-policy中找到。如果你安装了一个浏览器，你大概可以在这里读到它。

>    file://localhost/usr/share/doc/debian-policy/

>Some more detailed information can also be found in the files in the

一些更详细的信息亦可以在/usr/share/doc/sysv-rc目录的文件中找到
```
/usr/share/doc/sysv-rc directory.
```
Debian Policy dictates that /etc/init.d/*.sh scripts must work properly
when sourced.  The following additional rules apply:

debian手册规定，/etc/init.d/*.sh脚本当起作用时必须合适的工作，应遵守以下的这些规则：

* /etc/init.d/*.sh scripts must not rely for their correct functioning
  on their being sourced rather than executed.  That is, they must work
  properly when executed too. They must include "#!/bin/sh" at the top.
  This is useful when running scripts in parallel.

脚本必须不依赖它们正确的功能，在被应用后而不是在执行时。也就是说，它们在执行过程中也必须正确的工作。它们必须在顶部包含"#!/bin/sh"。这在并行运行脚本时是非常有用的。

* /etc/init.d/*.sh scripts must conform to the rules for sh scripts as
  spelled out in the Debian policy section entitled "Scripts" (§10.4).

脚本必须遵守这些为sh脚本设置的规则，这些规则在德班规则中被命名为Script。

>Use the update-rc.d command to create symbolic links in the /etc/rc?.d
as appropriate. See that man page for more details.

适当的使用update-rc.d 命令在/etc/rc?.d中创建符号链接。看那个手册获得更详细的信息。

>All init.d scripts are expected to have a LSB style header documenting
dependencies and default runlevel settings.  The header look like this
(not all fields are required):

所有的init.d脚本期望有一个LSB风格的头文档依赖和默认的运行等级设置。这个头部说明就像这样（不是所有的字段都是需要的）：

### BEGIN INIT INFO
# Provides:          skeleton
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Should-Start:      $portmap
# Should-Stop:       $portmap
# X-Start-Before:    nis
# X-Stop-After:      nis
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# X-Interactive:     true
# Short-Description: Example initscript
# Description:       This file should be used to construct scripts to be
#                    placed in /etc/init.d.
### END INIT INFO

>More information on the format is available from insserv(8).  This
information is used to dynamicaly assign sequence numbers to the
boot scripts and to run the scripts in parallel during the boot.

更多格式的信息可以从insserv获得。这个信息被动态的用于给boot启动脚本分配顺序号码，在机器启动过程中并行的运行脚本。

See also /usr/share/doc/insserv/README.Debian.


### CENTOS

>You are looking for the traditional init scripts in /etc/rc.d/init.d,
and they are gone?

你在/etc/rc.d/init.d目录里找传统的（惯例的）init初始脚本，它们没有了吗？

>Here's an explanation on what's going on:

发生了什么？这里是解释：

>You are running a systemd-based OS where traditional init scripts have
been replaced by native systemd services files. Service files provide
very similar functionality to init scripts. To make use of service
files simply invoke "systemctl", which will output a list of all
currently running services (and other units). Use "systemctl
list-unit-files" to get a listing of all known unit files, including
stopped, disabled and masked ones. Use "systemctl start
foobar.service" and "systemctl stop foobar.service" to start or stop a
service, respectively. For further details, please refer to
systemctl(1).

你正在运行一个基于OS的系统，这里的惯例的（传统的）初始化脚本被本地的systemd服务文件替代了。服务文件为init脚本提供非常相似的功能。使用服务文件，只需要简单的调用（祈求，恳求，引起）‘systemctl’，将会输出一个所有当前运行服务的列表（或其它单元）。使用‘systemctl list-unit-files’获得一个所有知道的单元文件清单，包括停止的，禁用的和掩蔽的。使用“systemctl start foobar.service”和"systemctl stop foobar.service" 去开启或者停止一个服务，分别地。更深入的详情，请参照systemctl(1);

>Note that traditional init scripts continue to function on a systemd
system. An init script /etc/rc.d/init.d/foobar is implicitly mapped
into a service unit foobar.service during system initialization.

注意传统的init脚本继续在一个systemd系统上起作用。一个init脚本/etc/rc.d/init.d/foobar是暗中的映射到一个服务党员foobar.service在西贡的初始化过程中。

>Thank you!

>Further reading:
```
        man:systemctl(1)
        man:systemd(1)
        http://0pointer.de/blog/projects/systemd-for-admins-3.html
        http://www.freedesktop.org/wiki/Software/systemd/Incompatibilities

```
### 参考文档
[在Linux中利用Service命令添加系统服务及开机自启动](https://blog.csdn.net/u013554213/article/details/78792686)  
[Ubuntu 系统 Update-rc.d 命令](https://blog.csdn.net/lile777/article/details/76592509)