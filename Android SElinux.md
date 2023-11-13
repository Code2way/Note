# SElinux 是什么

安全增强型 Linux（Security-Enhanced Linux）简称 SELinux，它是一个 Linux 内核模块，也是 Linux 的一个安全子系统。

SELinux 主要由美国国家安全局开发。2.6 及以上版本的 Linux 内核都已经集成了 SELinux 模块。

# SELinux 能干啥

SELinux 主要作用就是最大限度地减小系统中服务进程可访问的资源（最小权限原则）。

设想一下，如果一个以 root 身份运行的网络服务存在 0day 漏洞，黑客就可以利用这个漏洞，以 root 的身份在您的服务器上为所欲为了。是不是很可怕？

SELinux 就是来解决这个问题的。



- ##### LINUX权限控制机制

###### DAC(Discretionary Access Control)自主式权限控制

在没有使用 SELinux 的操作系统中，决定一个资源是否能被访问的因素是：某个资源是否拥有对应用户的权限（读、写、执行）。

只要访问这个资源的进程符合以上的条件就可以被访问。

而最致命问题是，root 用户不受任何管制，系统上任何资源都可以无限制地访问。

这种权限管理机制的主体是用户，也称为自主访问控制（DAC）

- ##### MAC （**Mandatory Access Control** ）

在使用了 SELinux 的操作系统中，决定一个资源是否能被访问的因素除了上述因素之外，还需要判断每一类进程是否拥有对某一类资源的访问权限。

这样一来，即使进程是以 root 身份运行的，也需要判断这个进程的类型以及允许访问的资源类型才能决定是否允许访问某个资源。进程的活动空间也可以被压缩到最小。

即使是以 root 身份运行的服务进程，一般也只能访问到它所需要的资源。即使程序出了漏洞，影响范围也只有在其允许访问的资源范围内。安全性大大增加。

这种权限管理机制的**主体是进程**，也称为强制访问控制（MAC）

SELinux 属于MAC的具体实现，增强了Linux系统的安全性。MAC机制的特点在于，资源的拥有者，并不能决定谁可以接入到资源。具体决定是否可以接入到资源，是基于安全策略。而安全策略则是有一系列的接入规则组成，并仅有特定权限的用户有权限操作安全策略。

一个简单的例子，则是一个程序如果要写入某个目录下的文件，在写入之前，一个特定的系统代码，将会依据进程的Context和资源的Context查询安全策略，并且根据安全策略决定是否允许写入文件。

**MAC 又细分为了两种方式，一种叫类别安全（MCS）模式，另一种叫多级安全（MLS）模式。**



- ##### MAC与DAC对比

在 DAC 模式下，只要相应目录有相应用户的权限，就可以被访问。而在 MAC 模式下，还要受进程允许访问目录范围的限制。



# Linux Security Module

SELinux的实现是依赖于Linux提供的Linux Security Module框架简称为LSM。其实LSM的名字并不是特别准确，因为他并不是Linux模块，而是一些列的hook，同样也不提供任何的安全机制。LSM的的重要目标是提供对linux接入控制模块的支持。

![img](https://pic4.zhimg.com/v2-223cde664509658132da7b937d21d6fb_b.webp?consumer=ZHI_MENG)

LSM 在内核数据结构中增加了安全字段，并且在重要的内核代码（系统调用）中增加了hook。可以在hook中注册回调函数对安全字段进行管理，以及执行接入控制。

# SELinux

Security Enhanced Linux(SELinux) 为Linux 提供了一种增强的安全机制，其本质就是回答了一个“Subject是否可以对Object做Action?”的问题，例如 Web服务可以写入到用户目录下面的文件吗？其中Web服务就是Subject而文件就是Object，写入对应的就是Action。

- Subject: 在SELinux里指的就是进程，也就是操作的主体。
- Object： 操作的目标对象，例如 文件（linux 中一切皆为文件）
- Action： 对Object做的动作，例如 读取、写入或者执行等等
- Context： Subject和Object都有属于自己的Context，也可以称作为Label。Context有几个部分组成，分别是SELinux User、SELinux Role、SELinux Type、SELinux Level，