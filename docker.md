# 把环境作为程序的一部分——docker

## 引言

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为什么会突然想起用Docker？因为最近的在写的开源项目[goMonitor](https://github.com/phospher/goMonitor)里用到了spark作为数据处理的框架，而spark的MapReduce在单机下运行（所有任务在同一个JVM下运行）跟在真正分布式方式下运行的结果是不一样的。为了测试程序的正确性，虚拟化技术似乎能帮到我很大的忙。

## 虚拟机与容器

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;到了21世纪的第二个十年，虚拟化技术已经里我们不陌生了。VMWare、VirtualBox等虚拟机产品，能帮助我们非常方便地创建虚拟机，并在这些虚拟机上面安装各种各样不同的环境。托这些虚拟机的福，当我们需要使用一个我们不熟悉的操作系统时，我们再也不用胆战心惊地怕把环境弄坏，毕竟即使把环境弄坏了，新建一个虚拟机并重新安装操作系统也不是一件难事。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;虚拟机带给我们的另一个便利就是环境复制的便利。毕竟虚拟机对于宿主机器来说就是一系列的文件，把这些文件拷贝一个副本就等于复制了这个虚拟机。我们可以轻易地制造多个一模一样的环境，而不用再去担心测试环境与生产环境不一致的问题了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;虚拟机技术的本质是对计算机硬件的模拟，运行其上的操作系统本质上并不知道自己运行在虚拟机上，因此只要虚拟机产品提供了驱动程序，理论上虚拟机上是可以跑任意操作系统的。这使得虚拟机的使用范围很广，深入操作系统内核来调试，甚至是开发权限的操作系统都可以使用虚拟机。但这样做的缺点也非常明显，就是资源消耗非常高。一般的PC，开两三个虚拟机或许还能正常运行的话，开个五六个可能就受不了了。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;随着SOA、微服务、云计算、分布式计算等技术渐渐火热起来，一种古老的虚拟化技术重新进入了人们眼脸——容器技术。是的，尽管容器技术是近几年再火起来的，但其实它是一个早在1979年就被实现了的技术。容器技术的本质是操作系统内核为应用程序提供一个与宿主操作系统隔离的运行环境，它既不是虚拟机，也不能算是一个完整的操作系统，但它的的确确为运行其上的应用程序提供了操作系统的服务，包括IO读写、文件管理、进程调度等等，因此对于应用程序，它并不知道自己其实运行在一个容器之上。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于需要操作系统内核的支持，因此容器技术的可移植性远没有虚拟机技术高。比如Liunx是2000年左右开始支持容器技术，到2008年LXC发布才标志着Liunx容器技术的成熟，在此之前Liunx上是无法创建容器的。而Windows支持容器就更晚了，必须使用Windows Server 2016或Windows10 Pro以上版本才能使用Windows原生的容器技术。但内核级别的支持也是容器技术的优点之一，容器技术的性能远好于虚拟机技术，甚至可以达到宿主机器的水平（当然是在宿主机器只运行一个容器的情况下，呵呵！） 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;随着服务器操作系统的大统一（Unix和Liunx占据了服务器操作系统的大半江山），和对虚拟化技术越来越迫切的需求，容器技术慢慢取代虚拟机技术，成为了虚拟化技术的主流。当然虚拟机技术不可能就此封尘的，因为仿真机器无论如何都会有它的适用场景。

## LXC与docker
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现在的虚拟化技术中，docker算是最火的容器技术了，甚至可以没有之一。可是既然2008年发布的LXC已经非常成熟了，为什么五年后的docker会超越它？docker仅仅是LXC的再造一次轮子吗？ 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要回答这些问题，需要从容器的使用者（在其上开发应用程序的开发者）角度考虑问题了。容器技术相对于虚拟机仅仅是解决了性能问题，当然也方便了我们复制环境，但以前一些困扰我们已久的问题依然没有解决：繁琐的环境初始化以及追踪环境变更历史。 

![LXC & docker](https://phospher.github.io/20161014110312912.jpg)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上图对比了LXC与docker在使用上的不同。由于LXC提供的使用方式是一般操作系统的使用方式，因此人们也习惯于用一般操作系统的使用思路使用LXC，于是对于开发人员来说LXC与一般操作系统就没有很大区别了，维护一般操作系统遇到的问题在使用LXC时一样会遇到。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于应用开发者而言，最舒服的事情莫过于整个环境只运行一个应用，这样我们无需担心环境变量的配置是否与其他应用冲突，无需担心端口是否被其他应用占用，环境怎么折腾都可以无需考虑是否影响其他应用。docker正是基于这一目标设计出来的。应用开发者使用docker的容器时，可以不需要登录进容器。当启动容器时，docker会同时帮我们运行我们指定的应用程序，当应用程序运行结束时，docker会自动帮我们关闭容器。这一切都是以应用为中心的，容器的生命周期与应用的生命周期保持同步。仅仅是这样的改变，让我们管理容器变得更简单，也让我们更乐意构建更多的容器，实现一个应用独享一个容器。

## 把环境作为程序的一部分

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于一个中大型的开发团队而言，生产环境的管理往往不在开发人员手上，甚至不允许开发人员登录到生产环境中。因此环境问题往往成为了程序部署的一大噩梦。得益于强大的shell系统，我们可以把环境初始化的工作交给shell脚本来处理，但开发人员依然无法控制环境的初始状态，解决这个问题的方法，要么是编写大量的校验代码以防止结果出错，要么是向运维人员详细了解环境的状态。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;docker则通过容器技术加上Dockerfile机制为开发者提供了一个可控的运行环境。我打算通过一个实际项目的例子说明开发人员可以如何用docker构建一个运行环境。

```Dockerfile
FROM openjdk:8

ARG SPARK_PATH

COPY $SPARK_PATH /spark

COPY docker-entrypoint.sh /
RUN chmod a+x /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上的例子是[goMonitor](https://github.com/phospher/goMonitor)项目为构建Spark运行节点所使用的Dockerfile（Dockerfile是docker为构建docker镜像使用的说明文本，详细可参考[docker的官方文档](https://docs.docker.com/)）。第一行的FROM语句告诉docker，我们打算以一个安装了openjdk 8的Linux环境作为我们的起始环境。从这里开始，我们得到了一个状态对我们来说是透明的环境了，我们不用担心环境变量是否已经被修改了（比如Java需要的CLASSPATH），端口是否已经被占用了等等，一切都在我们的掌握之中。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;后面的语句就是开始把程序部署到容器环境里面了，包括安装程序依赖的第三方库、拷贝应用程序的可执行文件到容器环境里面、设置环境变量或其他配置文件，最后告诉docker容器启动时该运行什么命令等等。这里不细讲Dockerfile的写法了，有兴趣可以到docker的官方网站上阅读相关的文档。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;只要编写正确的Dockerfile，无论是测试环境还是生产环境，我们都可以构建出一模一样的运行环境，不用担心环境的初始状态会影响到应用部署过程，因为环境的初始状态也在我们的掌控之中。也正因为构建环境变得容易，而且每次构建出来的环境初始状态都是已知的，使得我们可以调试应用的运行环境了。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在使用docker之前，由于部署脚本依赖于环境的状态，同时一旦环境的状态发生改变可能导致部署脚本无法多次运行，部署脚本一般很难调试，甚至是无法调试的，所以部署脚本的bug往往到正式上线的时候才发现，导致我们的部署工作举步难行。docker可以非常方便的构建环境，并且构建出来的环境都是稳定可控的，这使得我们可以一遍又一遍地通过Dockerfile和shell脚本构建新的容器，直到容器环境符合我们的要求为止。这样，连应用运行环境都变得可测试了。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;docker使用文本文件作为基础构建容器，作为程序员的我们凭直觉就能感觉到，我们可以使用代码管理工具（git、SVN等等）管理环境了。尽管虚拟机、LXC，甚至是docker也提供了`docker commit`命令，为环境的当前状态打上快照，并可以把快照分享出去，但毕竟快照体积可能很大，分享依然不是很容易。同时文本文件可以借助代码管理工具，跟踪文件的整个修改过程，这是二进制的快照文件很难做到的。

## 总结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本文主要介绍了docker是什么，同时讲述了docker为开发人员带来了什么新的开发体验，如果想多了解docker是如何使用的，可以查阅docker的官方文档，里面有非常详尽的学习资料。同时docker的托管网站（[https://hub.docker.com/](https://hub.docker.com/)）有许许多多开源的docker镜像供我们使用和学习。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可能容器技术与docker并不是虚拟化技术的终点，但是docker的出现使得虚拟化技术又迎来了一个高峰，同时使得“微服务”这个概念得以实现。尽管我个人不是“微服务”的信奉者，但是这些概念与实践手段的提出，的的确确改变了我们架构设计思想与开发模式。docker的出现让我看到了一种新的可能性，也因此有了这篇文章。