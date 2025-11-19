# 双系统安装之后的问题解决

双系统安装随便搜个教程就行（Ubuntu24.04）

感谢张宗x的全程帮忙安装


## 环境问题解决
更新系统环境
```bash
sudo apt upgrade
```

## 软件安装
要么从App Center里下载，要么用Fire Fox打开百度搜索下载

用浏览器安装的软件（注意是Ubuntu对应的.deb结尾的安装包），请在terminal中切换到下载目录，使用如下命令安装：
```bash
sudo dpkg -i 软件安装包全称（快捷输入可先输入首字母后按Tab补全）
```

## clash安装
搜索clash for windows linux，按照教程安装即可，之后就和windows使用一样，从URL下载配置即可。

网址：https://docs.lefly.co/contents/linux/cfw.html

## 卷挂载（即可以通过linux操作windows的磁盘文件）

首先，搜索Disks，打开后，找到磁盘，设置挂载，之后重启系统即可。具体可以看以下视频：

【解决Ubuntu 24.04 无法挂载磁盘的小妙招-哔哩哔哩】 https://b23.tv/am8Ht9c

但是我们会发现，可能会遇到能查看但是不能编辑的只读状态。

这个问题通常是因为 **Windows的NTFS文件系统** 在 Windows 关机时没有完全解除挂载（unmount）造成的，特别是当 Windows 启用了 **“快速启动 (Fast Startup)”** 或处于**休眠 (Hibernation)** 状态时。

Linux 看到这种“不干净”的状态，为了防止数据损坏，会强制将其以**只读 (read-only)** 模式挂载。


-----

**🛠️ 解决方案**

### 1\. **在 Windows 中彻底关闭“快速启动”和“休眠”** (推荐首选)

这是最常见和最彻底的解决方案。

  * **关闭快速启动（Fast Startup）：**
    1.  进入 **控制面板 (Control Panel)** -\> **硬件和声音 (Hardware and Sound)** -\> **电源选项 (Power Options)**。
    2.  点击左侧的 **“选择电源按钮的功能 (Choose what the power buttons do)”**。
    3.  点击顶部的 **“更改当前不可用的设置 (Change settings that are currently unavailable)”** (可能需要管理员权限)。
    4.  在底部 **“关机设置 (Shutdown settings)”** 下，**取消勾选** **“启用快速启动 (推荐) (Turn on fast startup (recommended))”**。
    5.  点击 **“保存更改 (Save changes)”**。
  * **关闭休眠（Hibernation）：**
    1.  以管理员身份运行 **命令提示符 (Command Prompt)** 或 **PowerShell**。
    2.  输入命令并执行：
        ```bash
        powercfg /h off
        ```
    3.  这会删除休眠文件 (`hiberfil.sys`) 并彻底禁用休眠。

完成上述步骤后，**在 Windows 中执行一次彻底的关机 (Shutdown)**，然后重启进入 Linux，再次尝试访问 Windows 磁盘。

### 2\. **使用 `ntfsfix` 工具尝试修复**

如果上述方法不奏效，或者你不想禁用 Windows 的快速启动/休眠，可以在 Linux 中使用 `ntfsfix` 工具来“修复”NTFS分区，它会清除挂载标志（例如休眠文件），让分区可以读写挂载。

1.  **确定 Windows 分区设备名：**
      * 使用 `lsblk` 或 `fdisk -l` 命令找到你的 Windows 分区，例如它可能是 `/dev/sda4` 或 `/dev/nvme0n1p3`。
2.  **卸载分区：**
      * 如果分区已经被挂载，需要先卸载它。
        ```bash
        sudo umount /mnt/7E36A98736A940CF/homework  # 将路径替换为你的实际挂载点
        ```
3.  **运行 `ntfsfix`：**
      * 将 `/dev/sdXX` 替换为你的 Windows 分区设备名（例如 `/dev/sda4`）。
        ```bash
        sudo ntfsfix /dev/sdXX 
        ```
4.  **重新挂载分区：**
      * 再次尝试访问（通过文件管理器或 `mount` 命令）你的分区。它现在应该以读写模式挂载了。


### 🚨 **重要提示**

  * **风险：** 强制挂载一个处于“不干净”状态（例如有休眠文件）的 NTFS 分区并进行写入操作，**存在损坏 Windows 文件系统或丢失数据的风险**。因此，最佳做法是先在 Windows 中禁用快速启动/休眠。
  * **Linux NTFS 驱动：** Linux 默认使用 `ntfs-3g` 或较新的内核 `ntfs3` 驱动来支持 NTFS 文件系统的读写。大多数现代 Linux 发行版都预装了这些驱动。如果你使用的是非常老的系统或者特殊配置，可能需要确认 `ntfs-3g` 是否已安装。

请先尝试在 Windows 中禁用快速启动并执行一次完全关机（步骤 1）。我执行了步骤一就完全解决了问题。


## 合上笔记本盖子后打开是黑屏

这是个很有意思的问题，每次遇到我都只能重启电脑。但是我找到了一下解决方案：

这是一个在 Linux 笔记本电脑用户中**非常常见**的问题，通常被称为“**唤醒失败 (Resume from Suspend Failure)**”或“**黑屏唤醒 (Black Screen on Wake)**”。

这个问题**并非双系统本身导致**，而是 Linux 内核或驱动程序在处理笔记本电脑的电源管理（特别是**挂起/休眠**功能）时，与特定硬件（尤其是**显卡**和 **BIOS/ACPI**）兼容性不良引起的。




### 永久修复方案 (Troubleshooting Steps)

由于导致问题的原因很多，你需要逐一尝试以下步骤：

#### 针对 显卡/驱动问题

  * **a. 更新/更换显卡驱动：**
      * 如果你是 **NVIDIA** 用户，请确保使用的是官方推荐的**最新专有驱动**，而不是开源的 `nouveau` 驱动。
      * 如果已经是最新驱动，尝试**降级**到之前已知稳定的版本。
  * **b. 禁用 NVIDIA Suspend/Resume 服务：**
      * 在某些情况下，NVIDIA 提供的系统服务可能会干扰正常的挂起/唤醒。尝试禁用它们：
        ```bash
        sudo systemctl stop nvidia-suspend.service
        sudo systemctl stop nvidia-hibernate.service
        sudo systemctl stop nvidia-resume.service
        sudo systemctl disable nvidia-suspend.service
        sudo systemctl disable nvidia-hibernate.service
        sudo systemctl disable nvidia-resume.service
        ```

#### 针对 ACPI/内核问题

  * **d. 切换 `mem_sleep` 模式：**
      * Linux 默认可能使用 `s2idle` 挂起模式，这可能在某些硬件上不稳定。你可以尝试切换到更传统的 **"deep"** 模式：
        1.  编辑 GRUB 配置文件：`sudo nano /etc/default/grub`
        2.  找到 `GRUB_CMDLINE_LINUX_DEFAULT=` 这一行，在引号内添加 `mem_sleep_default=deep`。
            **示例：** `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash mem_sleep_default=deep"`
        3.  保存并退出，然后更新 GRUB：`sudo update-grub`
        4.  重启系统。
  * **e. 检查/更新 Linux 内核：**
      * 如果你使用的是较旧的内核版本，新的硬件可能尚未得到良好支持。尝试**升级**到最新的稳定内核。
      * 反之，如果你使用的是最新的内核但出现了问题，尝试**降级**到上一个已知稳定的内核版本。
  * **f. BIOS 更新：**
      * 检查笔记本电脑制造商的网站，是否有针对你的型号的 **BIOS 更新**。许多制造商会在 BIOS 更新中加入对 Linux 的电源管理兼容性修复。


我执行了b，d就解决了问题（原来的c可能会影响windows的运行，我删去了）


## 查看隐藏文件
在 Linux 的图形化界面（如 GNOME, KDE, XFCE 等）中，查看以点（`.`）开头的文件（即**隐藏文件**）非常简单。

几乎所有的文件管理器（例如 Nemo, Nautilus, Dolphin, Thunar 等）都提供了**快捷键**和**菜单选项**来完成这个操作。

### 📁 **操作步骤：显示隐藏文件**

您可以通过以下两种最常用的方法来显示隐藏文件：

#### **方法一：使用快捷键 (推荐)**

这是最快、最通用的方法。

1.  打开您的文件管理器（例如，打开您的“主文件夹”或“文件”应用）。
2.  按下键盘上的组合键：
    $$\text{Ctrl} + \text{H}$$

**效果：** 所有以 `.` 开头的隐藏文件和文件夹（例如 `.config`, `.bashrc`）都会立即显示出来。再次按下 $\text{Ctrl} + \text{H}$ 即可将其隐藏。

#### **方法二：使用菜单选项**

如果忘记了快捷键，您可以通过文件管理器的菜单栏找到该选项：

1.  打开文件管理器。
2.  查找菜单栏或工具栏中的 **“查看” (View)** 选项。
3.  在下拉菜单中，勾选或点击名为 **“显示隐藏文件” (Show Hidden Files)** 或 **“显示隐藏文件和文件夹” (Show Hidden Files and Folders)** 的选项。

> **💡 提示：** 在某些文件管理器中，该选项可能位于一个**汉堡包菜单图标**（三条横线）或**齿轮图标**的设置菜单下。

### 🌐 **如何在终端（命令行）中查看？**

如果您偶尔需要使用命令行查看，也非常简单：

1.  打开终端。
2.  使用 `ls` 命令，并带上 `-a` 选项：
    $$\text{ls -a}$$

**效果：** 这会列出当前目录下**所有**的文件和文件夹，包括正常的（非隐藏）文件、隐藏文件，以及特殊的目录 `.`（当前目录）和 `..`（上级目录）。

---

## git时遇到的安全目录问题
您收到的这个错误信息 `fatal: detected dubious ownership in repository` 是 **Git 的一个安全机制** 在起作用。

### 🚨 **错误原因：可疑的所有权 (Dubious Ownership)**

这个错误通常发生在您使用 **Linux 子系统 (WSL)** 或 **挂载的 Windows/NTFS 文件系统** 来操作 Git 仓库时。

**Git 担心的问题是：**

  * 您的 Git 仓库目录 `/mnt/7E36A98736A940CF/linux_ubuntu` 的所有者（`yinchao`）与实际文件系统的所有者 **不匹配**，或者该目录位于一个**非标准文件系统**（如 NTFS 分区，它通过 `/mnt/7E36A98736A940CF` 挂载）上。
  * Git 认为如果一个仓库的目录所有者不是当前用户，那么运行 Git 命令可能会被**恶意利用**，从而在您不知情的情况下执行代码。

### ✅ **解决方法：将目录添加到安全列表**

如 Git 提示中所说，最简单、最安全的解决方法是**将您的仓库目录添加到 Git 的全局安全目录列表**。这告诉 Git，您信任这个目录，并确认它是安全的，即使它位于挂载的文件系统上。

请在您的终端中运行 Git 提示的这条命令：

```bash
git config --global --add safe.directory /mnt/7E36A98736A98736A940CF/linux_ubuntu
```

#### **步骤说明：**

1.  **`git config`**: Git 的配置命令。
2.  **`--global`**: 将此设置应用到您当前用户的所有 Git 仓库，而不是仅限于当前仓库。
3.  **`--add`**: 添加一个新的配置项。
4.  **`safe.directory`**: 指定要添加的安全目录类型。
5.  **`/mnt/7E36A98736A98736A940CF/linux_ubuntu`**: 您的 Git 仓库的完整路径。

-----

### 🚀 **后续操作**

在您成功运行上述安全配置命令后，您可以**重新运行**您最初尝试的命令来添加远程仓库：

```bash
git remote add origin git@github.com:CarterYin/Ubuntu_Windows.git
```

> **注意：** 我将您尝试添加的远程仓库名改为了 **`origin`**，这是 Git 中约定俗成的默认主远程仓库名。如果您想使用其他名称，请替换 `origin`。

-----


## 代理配置
配置 Linux 系统的代理有几种常见的方法，具体取决于您想设置的代理类型和作用范围。

### 1\. 临时环境变量代理（仅对当前 Shell 会话有效）

这种方式最简单，通常用于临时的网络请求，如 `wget`、`curl` 或 `apt` 等命令行工具。

  * **HTTP 代理：**
    ```bash
    export http_proxy="http://用户名:密码@代理服务器地址:端口"
    # 如果没有用户名和密码
    export http_proxy="http://代理服务器地址:端口"
    ```
  * **HTTPS 代理：**
    ```bash
    export https_proxy="http://用户名:密码@代理服务器地址:端口"
    # 注意：HTTPS 代理通常也使用 http:// 开头的 URL 格式，除非代理本身是 SOCKS/HTTPS 代理。
    ```
  * **FTP 代理：**
    ```bash
    export ftp_proxy="http://代理服务器地址:端口"
    ```
  * **不使用代理的地址（可选）：**
    ```bash
    export no_proxy="localhost,127.0.0.1,.example.com"
    ```

**提示：** 记住所有变量名（如 `http_proxy`）应使用**小写**，同时为了兼容性，建议也设置**大写**版本（`HTTP_PROXY`）。

### 2\. 永久环境变量代理（系统级或用户级）

若要永久生效，可以将上述 `export` 命令添加到配置文件中：

  * **用户级（仅对当前用户有效）：**

      * 添加到 `~/.bashrc` 或 `~/.zshrc` 文件末尾。
      * 例如：
        ```bash
        # 在文件末尾添加以下内容
        export http_proxy="http://192.168.1.100:8080"
        export https_proxy="$http_proxy"
        export no_proxy="localhost,127.0.0.1"
        ```
      * 保存后，运行 `source ~/.bashrc` 或重新登录使其生效。

  * **系统级（对所有用户有效）：**

      * 在 `/etc/profile.d/` 目录下创建一个新的 `.sh` 脚本文件，例如 `/etc/profile.d/proxy.sh`，并将上述 `export` 命令写入其中。
      * 给脚本添加执行权限：`sudo chmod +x /etc/profile.d/proxy.sh`。
      * 重新登录或重启系统使其生效。

### 3\. 配置 APT 包管理器的代理（针对 Debian/Ubuntu 等）

对于 `apt` 或 `apt-get` 下载包时需要使用代理，可以单独配置：

  * 编辑或创建文件 `/etc/apt/apt.conf.d/proxy.conf`（如果不存在）。
  * 添加以下行：
    ```
    Acquire::http::Proxy "http://用户名:密码@代理服务器地址:端口/";
    Acquire::https::Proxy "http://用户名:密码@代理服务器地址:端口/";
    ```

### 4\. 桌面环境网络设置代理（适用于 GUI 环境）

如果您使用带有图形界面的 Linux 发行版（如 Ubuntu、Fedora、Mint），可以通过系统设置来配置代理：

  * **Ubuntu (GNOME):** 通常在 **设置 (Settings) -\> 网络 (Network) -\> 网络代理 (Network Proxy)** 中进行配置。您可以选择“手动”并填入 HTTP、HTTPS 或 SOCKS 代理的详细信息。

-----

**请注意：**

  * **SOCKS 代理**：如果您使用的是 SOCKS 代理（例如 SOCKS5），需要将变量设置为 `export all_proxy="socks5://代理服务器地址:端口"`。
  * **确认**：配置完成后，您可以使用 `env | grep -i proxy` 命令来检查环境变量是否已正确设置。
