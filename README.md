##### 单点登录
##### 什么是单点登录：
单点登录（Single Sign On，简称为 SSO，是目前比较流行的企业业务整合的解决方案之一。SSO的定义是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。举例：上豆瓣，要登录豆瓣FM、豆瓣读书、豆瓣电影、豆瓣日记，如果我们访问豆瓣读书、豆瓣电影、豆瓣日记都需要进行一次登录认证，那么用户体验是非常不好的。所以引用了单点登录。只要一次登录就可以访问所有相互信任的应用系统。
###### 单点登陆解决的问题：
在企业发展初期，企业使用的系统很少，通常一个或者两个，每个系统都有自己的登录模块，运营人员每天用自己的账号登录，很方便。但随着企业的发展，用到的系统随之增多，运营人员在操作不同的系统时，需要多次登录，而且每个系统的账号都不一样，这对于运营人员来说，很不方便。于是，就想到是不是可以在一个系统登录，其他系统就不用登录了呢？这就是单点登录要解决的问题。
###### 单点登陆的实现方式：
![](https://github.com/yujiaweitobebetter/internship_yujiawei/blob/master/some%20image/%E5%8D%95%E7%82%B9%E7%99%BB%E9%99%86%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F.png)

上图实现的是如何使用cookie进行单点登录， 客户端持有ID，服务端持有session，两者一起用来保持登录状态。客户端需要用ID来作为凭证，而服务端需要用session来验证ID的有效性。但是session这东西一开始是每个server自己独有的，豆瓣FM有自己的session、豆瓣读书有自己的session，而记录ID的cookie又是不能跨域的。所以，我们要实现一次登录一次退出，只需要想办法让各个server的共用一个session的信息，让客户端在各个域名下都能持有这个ID就好了。再进一步讲，只要各个server拿到同一个ID，都能有办法检验出ID的有效性、并且能得到ID对应的用户信息就行了，也就是能检验ID。
###### 实现方法
- server端
以server群如何生成、验证ID的方式大致分为两种：
1. “共享Cookie”这个就是上面提到的共享session的方式，本质上cookie只是存储session-id的介质，session-id也可以放在每一次请求的url里。session这项机制一开始就是一个server一个session的，把session拿出来让所有server共享有点奇怪。
1. SSO-Token方式。因为共享session的方式不安全，所以我们不再以session-id作为身份的标识。我们另外生成一种标识，把它取名SSO-Token(或Ticket)，这种标识是整个server群唯一的，并且所有server群都能验证这个token，同时能拿到token背后代表的用户的信息。

- 浏览器端

   用最早的“共享session”的方式还是现在的“token”方式，身份标识到了浏览器端都要面临这样的一个问题：用户登录成功拿到token(或者是session-id)后怎么让浏览器存储和分享到其它域名下？同域名很简单，我们可以把token存在cookie里，把cookie的路径设置成顶级域名下，这样所有子域都能读取cookie中的token。这就是共享cookie的方式。
###### 技术实现机制
![](https://github.com/yujiaweitobebetter/internship_yujiawei/blob/master/some%20image/%E5%8D%95%E7%82%B9%E7%99%BB%E9%99%86%E6%8A%80%E6%9C%AF%E5%AE%9E%E7%8E%B0%E6%9C%BA%E5%88%B6.png)

 当用户第一次访问应用系统的时候，因为还没有登录，会被引导到认证系统中进行登录；根据用户提供的登录信息，认证系统进行身份校验，如果通过校验，应该返回给用户一个认证的凭据－－ticket（SSO-Token）；用户再访问别的应用的时候，就会将这个ticket（SSO-Token）带上，作为自己认证的凭据，应用系统接受到请求之后会把ticket送到认证系统进行校验，检查ticket的合法性。如果通过校验，用户就可以在不用再次登录的情况下访问应用系统2和应用系统3了。
要实现SSO，需要以下主要的功能：
   1、所有应用系统共享一个身份认证系统。统一的认证系统是SSO的前提之一。认证系统的主要功能是将用户的登录信息和用户信息库相比较，对用户进行登录认证；认证成功后，认证系统应该生成统一的认证标志（ticket），返还给用户。另外，认证系统还应该对ticket进行效验，判断其有效性。
   2、所有应用系统能够识别和提取ticket（SSO-Token）信息 　　要实现SSO的功能，让用户只登录一次，就必须让应用系统能够识别已经登录过的用户。应用系统应该能对ticket（SSO-Token）进行识别和提取，通过与认证系统的通讯，能自动判断当前用户是否登录过，从而完成单点登录的功能。

###### 单点登录的优缺点
- 优点
   1、提高用户的效率。 用户不再被多次登录困扰，也不需要记住多个 ID 和密码。另外，用户忘记密码并求助于支持人员的情况也会减少。
   2、提高开发人员的效率。 SSO 为开发人员提供了一个通用的身份验证框架。实际上，如果 SSO 机制是独立的，那么开发人员就完全不需要为身份验证操心。他们可以假设，只要对应用程序的请求附带一个用户名，身份验证就已经完成了。
   3、简化管理。 如果应用程序加入了单点登录协议，管理用户帐号的负担就会减轻。简化的程度取决于应用程序，因为 SSO 只处理身份验证。所以，应用程序可能仍然需要设置用户的属性（比如访问特权）。

- 缺点
   1、不利于重构 因为涉及到的系统很多，要重构必须要兼容所有的系统，可能很耗时。
   2、无人看守桌面 因为只需要登录一次，所有的授权的应用系统都可以访问，可能导致一些很重要的信息泄露。

##### 前后端分离
###### 前后端分离是什么？为什么要前后端分离？
就是把数据和页面分离开，后端不提供页面，只是纯粹的通过 Web API 来提供数据和业务交互能力，Web 前端就是纯粹的客户端角色，与 WinForm、移动终端应用属于同样的角色，可以把它们合在一起，统称为前端，分离开了后，后端不再考虑页面如何美化，前段也不需要了解后端采用的是什么样的技术实现方案，使得前后端的开发人员能够更加专注于自身业务的开发。
###### 一体式web与前后端分离架构对比
![](https://github.com/yujiaweitobebetter/internship_yujiawei/blob/master/some%20image/%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%E6%9E%84%E6%9E%B6.jpg)
以前的一体式 Web 架构示意
![](https://github.com/yujiaweitobebetter/internship_yujiawei/blob/master/some%20image/%E4%BB%A5%E5%89%8D%E7%9A%84%E4%B8%80%E4%BD%93%E5%BC%8F%E6%9E%84%E6%9E%B6.jpg)
现在的前后端分离构架示意图

###### 前后端分离主要技术切入点（重要）
前后端分离后，会出现以前web一体式构架中没有出现过得问题，比如认证，会话机制，签名验证等，既然是做对外的api接口，当然安全问题是我们需要认真考虑的问题了，那么webapi会存在那些安全隐患呢？
- 请求来源(身份)是否合法？
- 请求参数被篡改？
- 请求的唯一性(不可复制)，防止请求被恶意攻击

处理这些安全隐患可以采用token+signature认证的方式；原理是：
1. 做一个认证服务，提供一个认证的webapi，用户先访问它获取对应的token；
1. 用户拿着相应的token以及请求的参数和服务器端提供的签名算法计算出签名后再去访问指定的api；
1. 服务器端每次接收到请求就获取对应用户的token和请求参数，服务器端再次计算签名和客户端签名做对比，如果验证通过则正常访问相应的api，验证失败则返回具体的失败信息




##### 微服务：

###### 什么是微服务： 
微服务是一种架构风格，将一个大项目拆分为多个小的、独立的微服务（功能单元）。
###### 传统单体大项目的缺点：
- 系统较大、较复杂，开发难度大
- 部署速度慢
- 难以升级、维护

###### 微服务的特点：
- 小：微服务是体积较小的功能单元，将一个大项目拆分为多个微服务。
- 独：服务都是独立的，运行在单独的JVM进程中，需要单独部署、维护，服务可以使用不同的编程语言来写，可以使用不同的数据库。每个服务往往都要做集群（节点数视该服务的访问量而定）。
- 轻：服务之间的通信机制是轻量级的
- 松：服务之间是松耦合的

###### 微服务的优点：
- 易于开发、维护，扩展性好
- 启动快，修改局部无需重新部署整个项目
- 技术栈不受限制
- 使用微服务时，可以针对性地设置集群大小，比如电商网站，商品、订单模块负载大，集群节点多些；积分模块负载小，集群节点少些。

###### 微服务的缺点：
- 要部署、维护多个服务（服务治理），运维难度加大。
- 单体应用使不使用分布式都行，微服务一般都要使用分布式，对开发人员的技术要求较高
- 调整接口成本高，需要同时修改调用它的其它微服务。
- 代码重复多。单体应用把要重复调用代码封装为工具类，项目中直接调用即可；每个微服务都是独立的，不能调用其它微服务中的类，需要把要用的类copy到要本服务中。
- 事务难度增加，要使用分布式事务。比如购买商品获赠积分，订单服务创建订单+积分服务增加积分，这是一个事务，但不在同一个应用中，使用事务难度增大。

##### 容器
###### 什么是容器：
容器就是将软件打包成标准化单元，以用于开发、交付和部署，是一种虚拟化解决方案，相对于传统虚拟机不同。传统的虚拟机是用中间层，将一台或多台独立的机器虚拟运行在硬件之上。而容器则是直接运行操作系统之上的用户空间，因此容器虚拟又被称为操作系统虚拟化。由于容器依赖于操作系统的特性，因此容器只能运行相同或相似内核的操作系统。
##### 容器和虚拟机的对比：

- 容器虚拟化的是操作系统而不是硬件，容器之间是共享同一套操作系统资源的。
- 虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统。因此容器的隔离级别会稍低一些。
- 容器是一个应用层抽象，用于将代码和依赖资源打包在一起。 多个容器可以在同一台机器上运行，共享操作系统内核，但各自作为独立的进程在用户空间中运行 。与虚拟机相比， 容器占用的空间较少（容器镜像大小通常只有几十兆），瞬间就能完成启动 。
- 容器和虚拟机的对比如图所示
![](https://github.com/yujiaweitobebetter/internship_yujiawei/blob/master/some%20image/vm%E5%92%8C%E5%AE%B9%E5%99%A8.png)

##### Docker
###### 什么是Docker:
Docker是一套完整的容器管理系统，Docker提供了一组命令，让用户更加方便直接地使用容器技术，而不需要过多关心底层内核技术。Docker 是一个开源的应用容器引擎，基于Go 语言并遵从 Apache2.0 协议开源。Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
###### Docker的三个基本概念:
- 镜像（Image）
- 容器（Container）
- 仓库（Repository）

 - 镜像：
镜像是容器构建的基石，是基于联合文件系统的一种层式结构由一系列指令构建，是一种轻量级、可执行的独立软件包，用于打包软件运行环境和基于运行环境开发的软件（代码、运行时、库、环境变量和配置文件）。Docker镜像是一个只读模板，它包含创建 Docker容器的说明。它和系统安装光盘有点像，使用系统安装光盘可以安装系统，同理，使用Docker镜像可以运行 Docker镜像中的程序。
 - 容器（container）：
Docker利用容器（container）独立运行的一个或者一组应用。容器是利用镜像创建的运行实例。可以把容器看做是一个简易版的Linux环境 (包括root用户名权限、进程空间、用户空间和网络空间等)和运行在其中的应用程序。
 - 仓库（repository）:
 是集中存放镜像文件的场所 。最大的公开仓库是docker hub存放了数量庞大的镜像供用户下载。
 
###### 总结：
- Docker本身是一个容器运行载体或者称之为管理引擎。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就叫image镜像文件。 只有通过这个镜像文件才能生成docker容器。image文件可以看做是容器的模板。 Docker根据image文件生成容器的实例。 同一个image文件，可以生成多个同时运行的容器实例。
- image文件生成的容器实例，本身也是一个文件，称为镜像文件；
- 一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器；
- 至于仓库，就是放了一堆镜像的地方，我们可以把镜像发布到仓库中，需要的时候从仓库中拉下来就可以了。
- 一句话：去仓库将镜像拉取到本地，然后使用一条命令将镜像运行起来，变成容器。

###### Docker架构图：
![](https://github.com/yujiaweitobebetter/internship_yujiawei/blob/master/some%20image/Docker%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

###### Docker优点:
 - 相比于传统的虚拟化技术,容器更加简洁高效;
 - 传统虚拟机需要给每个VM安装操作系统;
 - 容器使用的共享公共库和程序;
 
###### Docker的用途
1. 提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
2. 提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
3. 组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构

##### 微服务的容器化
站在 Docker 的角度，软件就是容器的组合：业务逻辑容器、数据库容器、储存容器、队列容器等等。Docker 使得软件可以拆分成若干个标准化容器，然后像搭积木一样组合起来。这正是微服务（microservices）的思想：软件把任务外包出去，让各种外部服务完成这些任务，软件本身只是底层服务的调度中心和组装层。微服务很适合用 Docker 容器实现，每个容器承载一个服务。一台计算机同时运行多个容器，从而就能很轻松地模拟出复杂的微服务架构。

###### 微服务容器化的原因及优势：
1、资源成本：轻量级容器资源利用率更高，容器成本相对较低
2、管理成本：m个微服务n个实例，可能需要m * n个虚机，虚机服务器管理成本较高（当然一个虚机可能部署多个实例），而容器直接在物理机上运行，可直接通过云管平台管理，管理复杂度较低。微服务化程度越高，容器技术的管理成本越低。
3、应用部署成本：容器一般通过镜像（OS+发布包）进行部署，一键发布（集成云管或者Jenkins），部署快效率高;而虚机通过在OS上部署发布包进行部署，虽然可以通过自动化（Ansible）等手段发布，但总体上没有容器技术高效便捷。
4、扩缩容效率：在服务器资源充足的情况下，容器技术可分钟级扩容，而虚机则由于技术、流程问题可能需要小时级甚至几天时间，所以容器技术更加快速便捷。



