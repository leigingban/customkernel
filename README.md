# customkernel

**在大家安装 Project C 内核之前，请先安装以下两个包 dev-util/dwarves dev-libs/libbpf**

**请注意：从2020年10月2日开始，内核小版本号补丁跟随 Gentoo Genpatches，5.8.13 以后的内核将取消单独的小版本补丁，一方面能够保持和 Gentoo 的一致性，另一方面能够帮助我更好的维护。另外，在内核的 config 中的 CONFIG_LOCALVERSION="" 请大家还是给自己的版本取个名字，另一个源提供的 Xanmod 中是不提供 config 文件的，但是我的保留了原始的 .config 文件，一方面有助于使用者更好的编译，另一方面保留了这个内核配置文件中的优化部分，而我这个 overlay 中 config 文件夹中的 xanmod config 就删掉了，另外的两个保留，保留的是我自己使用的，各位根据具体情况酌情修改，5.9这次更新我发现我的 .config 配置文件不够全面，于是更换成 linux-tkg 的 .config 文件。最后，这个三个内核都可以安装 zfs-kmod，望周知**

## 三个定制 Linux 内核 ebuild

### Xanmod Cachy 内核

[Xanmod](https://xanmod.org/) 是 Linux 第三方定制内核中非常著名的。原始 Xanmod 内核并没有对 CPU 调度器做非常特殊定制的情况下，能够在 [Phoronix Benchmark](https://www.phoronix.com/scan.php?page=article&item=xanmod-2020-kernel&num=3) 中击败众多对手，比方说 [Liquorix](https://liquorix.net/)。并且 Xanmod 的稳定版本在其官方主页支持基于 Debian 的所有发行版，当然，还有 [Archlinux](https://aur.archlinux.org/packages/linux-xanmod/) 和 [Gentoo](https://gitlab.com/src_prepare/src_prepare-overlay/-/tree/master/sys-kernel/xanmod-sources) 。那么既然已经有 Genoo 的 overlay 了，我为什么还要做一个 overlay 呢？

1. [src_prepare overlay](https://gitlab.com/src_prepare/src_prepare-overlay) 只提供稳定版本的 Xanmod 内核
2. 跟随上游 Xanmod 进行更新，Linux 主线更新的某些版本会选择性跳过
3. 我选择的是 [Xanmod Cachy](https://sourceforge.net/projects/xanmod/files/experimental/cachy_cpu_sched/) 这个分支的主线，在 Xanmod 主页，基于 Cachy 调度器的内核属于实验性，然而我自己测试下来，响应非常快，所以我选择这个主线的内核到我的 overlay，这也是同 src_prepare 的差异
4. config 在内核文件夹中，.config 不会删除，内容同 Xanmod 官方提供的
5. 添加 UKSM 补丁，这个补丁 Xanmod 并没有添加到主线

关于 customkernel 内核中，Xanmod 将保持最小的变化给各位朋友。

### Project C 内核

[Project C](https://cchalpha.blogspot.com/) 是 Linux 内核中 CPU Scheduler 第三方补丁，Project C 主要包含两个 CPU 调度器，一个是 BMQ，另一个是 PDS，两者都是非常适合在非RT内核中使用，提高响应性是它们的共同目的，就目前测试的结果来看，当然是稍微老的内核，PDS 和 BMQ 都非常适合在 Linux 的游戏机中使用，参考这个[视频](https://youtu.be/phA-M1biogE)，PDS 效果更好。目前这个内核我只添加 Project C 补丁，fsgsbase 和 UKSM 补丁，其余的补丁将来在通过测试慢慢加入。这个内核将遵循以下原则更新：

1. 内核跟随 [Linux Kernel](https://www.kernel.org/) 官方主线，补丁跟随 Project C 主线，fsgsbase 将在 5.9 内核融合主线，届时这个补丁将不复存在，UKSM 尽可能保持加入状态
2. 2020年9月22日测试成功加入了 Lucjan 补丁集，具体包含的内容，可以参阅下面 Cachy 第6条，不过舍弃了关于 Intel pstate 补丁，原作者要去掉，我还是保留了
3. 加入 zstd 补丁
4. 下面的警告⚠还是需要注意，虽然现在从 r1 开始打上补丁，另外反映的情况是大部分 AMD 已经不会出现死机，但是大家还是要注意

**现阶段 Project C 补丁存在的 [bug](https://gitlab.com/alfredchen/linux-prjc/-/issues/8)，尤其是在 AMD 的 Ryzen 处理器上会出现死机的现象，无法通过 SSH 进入系统查看，建议 CPU 是 Ryzen 的朋友不要使用，等待后续 bug 解决再使用**

### Cachy 内核

[Cachy](https://github.com/hamadmarri/cachy-sched) 内核是 Linux 内核中 CPU Scheduler 的新成员(第三方)，就目前文档中能够获取的信息来看，这个调度器的最核心特点是：利用 CPU 缓存尽可能让任务在该 CPU 执行，直到另一个 CPU 空闲，任务被闲置 CPU 拉去（这里我理解的 CPU 是逻辑核心）。Cachy 调度器利用 CPU 缓存并且基于 Highest Response Ratio Next (HRRN) 策略。那么在 customkernel 源中，我将遵循以下原则持续更新：

1. 上游跟随 [Linux Kernel](https://www.kernel.org/) 官方主线，补丁跟随 Cachy 主线
2. 加上 fsgsbase 补丁
3. 加入 UKSM 补丁
4. 加入 zen 补丁
5. 加入 GCC 优化补丁
6. 加入了 LL 补丁集，这个系列的补丁包括：750Hz 时钟中断，放弃之前使用 500Hz 时钟中断，前者认为能够平衡笔记本的续航和性能；磁盘调度器 bfq 代替 mq-deadline，这个更换有点个人情节的感觉，不过其实也不用担心，因为如果硬盘是固态，都会切换成 none；256那个补丁原名叫做：0003-sched-core-nr_migrate-256-increases-number-of-tasks-.patch，我翻译的可能不准确，应该是平行任务从32个增加到256个，还更改了 rt_runtime 到980000，这个具体含义我不清楚；2048 这个补丁将 VM_READAHEAD_PAGES 从128K提升到2M；
7. 加入 zstd 补丁
8. 将来根据情况加入更多优化补丁

## 写在最后

以上就是该 overlay 现在和将来的目标，维护这三个 ebuild，持续更新，为了更好的性能，为了我们的爱——Linux，为了我的至爱——Wuxunmo

希望大家多多提意见，多多提 issue，多多点❤❤❤
