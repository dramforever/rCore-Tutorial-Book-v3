实验环境配置
============

.. toctree::
   :hidden:
   :maxdepth: 4
   
本节我们将完成环境配置并成功运行 rCore-Tutorial-v3 。整个流程分为下面几个部分：

- 系统环境配置
- C/Rust 开发环境配置
- Qemu 模拟器安装
- 其他工具安装
- 运行 rCore-Tutorial-v3

系统环境配置
-------------------------------

目前实验内容可支持在 `Ubuntu操作系统 <https://cdimage.ubuntu.com/releases/>`_ 、 `openEuler操作系统 <https://repo.openeuler.org/openEuler-20.03-LTS-SP2/ISO/>`_ 、 `龙蜥操作系统 <https://openanolis.cn/anolisos>`_ 等上面进行操作。对于 Windows10/11 和 macOS 上的用户，可以使用WSL2、VMware Workstation 或 VirtualBox 等相关软件，通过虚拟机方式安装 Ubuntu18.04 / 20.04、openEuler操作系统、龙蜥操作系统等，并在上面进行实验。

对于Windows10/11 的用户可以通过系统内置的 WSL2 虚拟机（请不要使用 WSL1）来安装 Ubuntu 18.04 / 20.04 。步骤如下：

- 升级 Windows 10/11 到最新版（Windows 10 版本 18917 或以后的内部版本）。注意，如果
  不是 Windows 10/11 专业版，可能需要手动更新，在微软官网上下载。升级之后，
  可以在 PowerShell 中输入 ``winver`` 命令来查看内部版本号。
- 「Windows 设置 > 更新和安全 > Windows 预览体验计划」处选择加入 “Dev 开发者模式”。
- 以管理员身份打开 PowerShell 终端并输入以下命令：

  .. code-block::

     # 启用 Windows 功能：“适用于 Linux 的 Windows 子系统”
     >> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

     # 启用 Windows 功能：“已安装的虚拟机平台”
     >> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

     # <Distro> 改为对应从微软应用商店安装的 Linux 版本名，比如：`wsl --set-version Ubuntu 2`
     # 如果你没有提前从微软应用商店安装任何 Linux 版本，请跳过此步骤
     >> wsl --set-version <Distro> 2

     # 设置默认为 WSL 2，如果 Windows 版本不够，这条命令会出错
     >> wsl --set-default-version 2

-  `下载 Linux 内核安装包 <https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package>`_
-  在微软商店（Microsoft Store）中搜索并安装 Ubuntu18.04 / 20.04。

如果你打算使用 VMware 安装虚拟机的话，我们已经配置好了一个能直接运行 rCore-Tutorial-v3 的 Ubuntu18.04 镜像，它是一个 ``vmdk`` 格式的虚拟磁盘文件，只需要在 VMware 中新建一台虚拟机，在设置虚拟磁盘的时候选择它即可。`百度网盘链接 <https://pan.baidu.com/s/1JzKjWivy9GZKK8rc3WMJ0g>`_ （提取码 x5mf ）或者 `清华云盘链接 <https://cloud.tsinghua.edu.cn/d/a9b7b0a1b4724c3f9c66/>`_ 。已经创建好用户 oslab ，密码为一个空格。它已经安装了中文输入法和 Markdown 编辑器 Typora 还有作为 Rust 集成开发环境的 Visual Studio Code，能够更容易完成实验并撰写实验报告。如果想要使用 VMWare 安装 openEuler 虚拟机的话，可以在 `openEuler官网 <https://repo.openeuler.org/openEuler-20.03-LTS-SP2/ISO/>`_ 下载 ISO 自行安装，接着需要参考网络上的一些教程配置网络和安装图形界面。

.. _link-docker-env:

.. note::

   **Docker 开发环境**

   感谢 dinghao188 和张汉东老师帮忙配置好的 Docker 开发环境，进入 Docker 开发环境之后不需要任何软件工具链的安装和配置，可以直接将 tutorial 运行起来，目前应该仅支持将 tutorial 运行在 Qemu 模拟器上。
   
   使用方法如下（以 Ubuntu18.04 为例）：

   1. 通过 ``su`` 切换到管理员账户 ``root`` ；
   2. 在 ``rCore-Tutorial-v3`` 根目录下 ``make docker`` 进入到 Docker 环境；
   3. 进入 Docker 之后，会发现当前处于根目录 ``/`` ，我们通过 ``cd mnt`` 将当前工作路径切换到 ``/mnt`` 目录；
   4. 通过 ``ls`` 可以发现 ``/mnt`` 目录下的内容和 ``rCore-Tutorial-v3`` 目录下的内容完全相同，接下来就可以在这个环境下运行 tutorial 了。例如 ``cd os && make run`` 。    


.. chyyuu 下面的说法是面向github网页的，在书中应该有改变???

你也可以在 Windows10/11 或 macOS 原生系统或者其他 Linux 发行版上进行实验，基本上不会出现太大的问题。不过由于时间问题我们主要在 Ubuntu18.04 on x86-64上进行了测试，后面的配置也都是基于它的。如果遇到了问题的话，请在本节的讨论区中留言，我们会尽量帮助解决。

基于RISC-V硬件环境的配置
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

目前已经出现了可以在RISC-V 64（简称RV64）的硬件模拟环境（比如QEMU with RV64）和真实物理环境（如全志哪吒D1开发板、SiFive U740开发板）的Linux系统环境。但Linux RV64相对于Linux x86-64而言，虽然挺新颖的，但还不够成熟，已经适配和预编译好的应用软件包相对少一些，适合hacker进行尝试。如果同学有兴趣，我们也给出多种相应的硬件模拟环境和真实物理环境的Linux for RV64发行版，以便于这类同学进行实验：

- `Ubuntu for RV64的QEMU和SiFive U740开发板系统镜像 <https://cdimage.ubuntu.com/releases/20.04.3/release/ubuntu-20.04.3-preinstalled-server-riscv64+unmatched.img.xz>`_
- `OpenEuler for RV64的QEMU系统镜像 <https://repo.openeuler.org/openEuler-preview/RISC-V/Image/openEuler-preview.riscv64.qcow2>`_
- `Debian for RV64的D1哪吒开发板系统镜像 <http://www.perfxlab.cn:8080/rvboards/RVBoards_D1_Debian_img_v0.6.1/RVBoards_D1_Debian_img_v0.6.1.zip>`_

注：后续的配置主要基于Linux for x86-64系统环境，如果同学采用Linux for RV64环境，需要自己配置。不过在同学比较熟悉的情况下，配置方法类似且更加简单。可能存在的主要问题是，面向Linux for RV64的相关软件包可能不全，这样需要同学从源码直接编译出缺失的软件包。

C 开发环境配置
-------------------------------------------

在实验或练习过程中，也会涉及部分基于C语言的开发，可以安装基本的本机开发环境和交叉开发环境。下面是以Ubuntu 20.04为例，需要安装的C 开发环境涉及的软件：

.. code-block:: bash

   $ sudo apt-get update && sudo apt-get upgrade
   $ sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu

注：上述软件不是Rust开发环境所必须的。

Rust 开发环境配置
-------------------------------------------

首先安装 Rust 版本管理器 rustup 和 Rust 包管理器 cargo，这里我们用官方的安装脚本来安装：

.. code-block:: bash

   curl https://sh.rustup.rs -sSf | sh

如果通过官方的脚本下载失败了，可以在浏览器的地址栏中输入 `<https://sh.rustup.rs>`_ 来下载脚本，在本地运行即可。

如果官方的脚本在运行时出现了网络速度较慢的问题，可选地可以通过修改 rustup 的镜像地址（修改为中国科学技术大学的镜像服务器）来加速：

.. code-block:: bash
   
   export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
   export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
   curl https://sh.rustup.rs -sSf | sh

或者使用tuna源来加速 `参见 rustup 帮助 <https://mirrors.tuna.tsinghua.edu.cn/help/rustup/>`_：

.. code-block:: bash
   
   export RUSTUP_DIST_SERVER=https://mirrors.tuna.edu.cn/rustup
   export RUSTUP_UPDATE_ROOT=https://mirrors.tuna.edu.cn/rustup/rustup
   curl https://sh.rustup.rs -sSf | sh

或者也可以通过在运行前设置命令行中的科学上网代理来实现：

.. code-block:: bash 

   # e.g. Shadowsocks 代理，请根据自身配置灵活调整下面的链接
   export https_proxy=http://127.0.0.1:1080
   export http_proxy=http://127.0.0.1:1080
   export ftp_proxy=http://127.0.0.1:1080

安装完成后，我们可以重新打开一个终端来让之前设置的环境变量生效。我们也可以手动将环境变量设置应用到当前终端，只需要输入以下命令：

.. code-block:: bash

   source $HOME/.cargo/env

接下来，我们可以确认一下我们正确安装了 Rust 工具链：

.. code-block:: bash
   
   rustc --version

可以看到当前安装的工具链的版本。

.. code-block:: bash

   rustc 1.46.0-nightly (7750c3d46 2020-06-26)

.. warning::
   目前用于操作系统实验开发的 rustc 编译器的版本不局限在 1.46.0 这样的数字上，你可以选择更新版本的 rustc 编译器。但注意只能用 rustc 的 nightly 版本。


可通过如下命令安装 rustc 的 nightly 版本，并把该版本设置为 rustc 的缺省版本。

.. code-block:: bash
   
   rustup install nightly
   rustup default nightly


我们最好把软件包管理器 cargo 所用的软件包镜像地址 crates.io 也换成中国科学技术大学的镜像服务器来加速三方库的下载。我们打开（如果没有就新建） ``~/.cargo/config`` 文件，并把内容修改为：

.. code-block:: toml

   [source.crates-io]
   registry = "https://github.com/rust-lang/crates.io-index"
   replace-with = 'ustc'
   [source.ustc]
   registry = "git://mirrors.ustc.edu.cn/crates.io-index"

同样，也可以使用tuna源 `参见 crates.io 帮助 <https://mirrors.tuna.tsinghua.edu.cn/help/crates.io-index.git/>`_：

.. code-block:: toml

   [source.crates-io]
   replace-with = 'tuna'

   [source.tuna]
   registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

接下来安装一些Rust相关的软件包

.. code-block:: bash

   rustup target add riscv64gc-unknown-none-elf
   cargo install cargo-binutils --vers =0.3.3
   rustup component add llvm-tools-preview
   rustup component add rust-src

.. warning::
   如果你换了另外一个rustc编译器（必须是nightly版的），需要重新安装上述rustc所需软件包。rCore-Tutorial 仓库中的 ``Makefile`` 包含了这些工具的安装，如果你使用 ``make run`` 也可以不手动安装。

至于 Rust 开发环境，推荐 JetBrains Clion + Rust插件 或者 Visual Studio Code 搭配 rust-analyzer 和 RISC-V Support 插件。

.. note::

   * JetBrains Clion是付费商业软件，但对于学生和教师，只要在 JetBrains 网站注册账号，可以享受一定期限（半年左右）的免费使用的福利。
   * Visual Studio Code 是开源软件，不用付费就可使用。
   * 当然，采用 VIM，Emacs 等传统的编辑器也是没有问题的。

Qemu 模拟器安装
----------------------------------------

我们需要使用 Qemu 5.0.0 版本进行实验，而很多 Linux 发行版的软件包管理器默认软件源中的 Qemu 版本过低，因此我们需要从源码手动编译安装 Qemu 模拟器。下面以 Ubuntu 18.04 平台上的安装流程为例进行说明：

.. warning::

   注意，如果安装了 Qemu 6.0+ 版本，则目前需要将项目目录下的 bootloader（也即 RustSBI）更新为最新的 0.2.0-alpha.6 版本。它们目前可以在 ``chX-dev`` 分支中找到。


首先我们安装依赖包，获取 Qemu 源代码并手动编译：

.. code-block:: bash

   # 安装编译所需的依赖包
   sudo apt install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev \
                 gawk build-essential bison flex texinfo gperf libtool patchutils bc \
                 zlib1g-dev libexpat-dev pkg-config  libglib2.0-dev libpixman-1-dev git tmux python3 python3-pip
   # 下载源码包 
   # 如果下载速度过慢可以使用我们提供的百度网盘链接：https://pan.baidu.com/s/1z-iWIPjxjxbdFS2Qf-NKxQ
   # 提取码 8woe
   wget https://download.qemu.org/qemu-5.0.0.tar.xz
   # 解压
   tar xvJf qemu-5.0.0.tar.xz
   # 编译安装并配置 RISC-V 支持
   cd qemu-5.0.0
   ./configure --target-list=riscv64-softmmu,riscv64-linux-user
   make -j$(nproc)

.. note::
   
   注意，上面的依赖包可能并不完全，比如在 Ubuntu 18.04 上：

   - 出现 ``ERROR: pkg-config binary 'pkg-config' not found`` 时，可以安装 ``pkg-config`` 包；
   - 出现 ``ERROR: glib-2.48 gthread-2.0 is required to compile QEMU`` 时，可以安装 
     ``libglib2.0-dev`` 包；
   - 出现 ``ERROR: pixman >= 0.21.8 not present`` 时，可以安装 ``libpixman-1-dev`` 包。

   另外一些 Linux 发行版编译 Qemu 的依赖包可以从 `这里 <https://risc-v-getting-started-guide.readthedocs.io/en/latest/linux-qemu.html#prerequisites>`_ 找到。

之后我们可以在同目录下 ``sudo make install`` 将 Qemu 安装到 ``/usr/local/bin`` 目录下，但这样经常会引起冲突。个人来说更习惯的做法是，编辑 ``~/.bashrc`` 文件（如果使用的是默认的 ``bash`` 终端），在文件的末尾加入几行：

.. code-block:: bash

   # 请注意，qemu-5.0.0 的父目录可以随着你的实际安装位置灵活调整
   export PATH=$PATH:$HOME/qemu-5.0.0
   export PATH=$PATH:$HOME/qemu-5.0.0/riscv64-softmmu
   export PATH=$PATH:$HOME/qemu-5.0.0/riscv64-linux-user

随后即可在当前终端 ``source ~/.bashrc`` 更新系统路径，或者直接重启一个新的终端。

此时我们可以确认 Qemu 的版本：

.. code-block:: bash

   qemu-system-riscv64 --version
   qemu-riscv64 --version

在其他缺少预编译 QEMU with RV64 软件包的Linux x86-64 平台（如openEuler操作系统）上，首先需要从 openEuler 社区维护的 Qemu 的 `riscv分支 <https://gitee.com/src-openeuler/qemu/tree/riscv/>`_ 下载 QEMU 源码，并直接通过 rpmbuild 进行构建。

K210 真机串口通信
------------------------------

为了能在 K210 真机上运行 Tutorial，我们还需要安装基于 Python 的串口通信库和简易的串口终端。

.. code-block:: bash

   pip3 install pyserial
   sudo apt install python3-serial

GDB 调试支持
------------------------------

在 ``os`` 目录下 ``make debug`` 可以调试我们的内核，这需要安装终端复用工具 ``tmux`` ，还需要基于 riscv64 平台的 gdb 调试器 ``riscv64-unknown-elf-gdb`` 。该调试器包含在 riscv64 gcc 工具链中，工具链的预编译版本可以在如下链接处下载：

- `Ubuntu 平台 <https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.1-x86_64-linux-ubuntu14.tar.gz>`_
- `macOS 平台 <https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.1-x86_64-apple-darwin.tar.gz>`_
- `Windows 平台 <https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.1-x86_64-w64-mingw32.zip>`_
- `CentOS 平台 <https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.1-x86_64-linux-centos6.tar.gz>`_


解压后在 ``bin`` 目录下即可找到 ``riscv64-unknown-elf-gdb`` 以及另外一些常用工具 ``objcopy/objdump/readelf`` 等。

在其他缺少预编译 riscv64 gcc 工具链的Linux x86-64 平台（如openEuler操作系统、龙蜥操作系统等）上，则需要 clone `riscv 工具链仓库 <https://github.com/riscv-collab/riscv-gnu-toolchain>`_ 并参考其说明手动构建。

运行 rCore-Tutorial-v3
------------------------------------------------------------

在 Qemu 平台上运行
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

如果是在 Qemu 平台上运行，只需在 ``os`` 目录下 ``make run`` 即可。在内核加载完毕之后，可以看到目前可以用的
应用程序。 ``usertests`` 打包了其中的很大一部分，所以我们可以运行它，只需输入在终端中输入它的名字即可。

.. image:: qemu-final.gif

之后，可以先按下 ``Ctrl+A`` ，再按下 ``X`` 来退出 Qemu。

在 K210 平台上运行
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

如果是在 K210 平台上运行则略显复杂。

首先，我们需要将 MicroSD 插入 PC 来将文件系统镜像拷贝上去。

.. image:: prepare-sd.gif

.. warning::

   在 ``os/Makefile`` 中我们默认设置 MicroSD 在当前操作系统中可以用设备 ``SDCARD=/dev/sdb`` 访问。你可以使用 ``df -hT`` 命令来确认在你的环境中 MicroSD 是哪个设备，并在 ``make sdcard`` 之前对 ``os/Makefile`` 的 ``SDCARD`` 配置做出适当的修改。不然，这有可能导致 **设备 /dev/sdb 上数据丢失**！
   
随后，我们将 MicroSD 插入 K210 开发板，将 K210 开发板连接到 PC ，然后进入 ``os`` 目录 ``make run BOARD=k210`` 在 K210 开发板上跑 rCore Tutorial 。 

.. image:: k210-final.gif

之后，可以按下 ``Ctrl+]`` 来退出串口终端。

由于教程的 ch1~ch5 分支还没有文件系统，在 K210 上运行这些分支无需 MicroSD 卡也不需要进行文件系统镜像烧写工作，直接切换到 `os` 目录下 `make run BOARD=k210` 即可。

到这里，恭喜你完成了实验环境的配置，可以开始阅读教程的正文部分了！
