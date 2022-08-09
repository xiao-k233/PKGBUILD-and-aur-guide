# PKGBUILD完全指南
PKGBUILD 是一个 **shell 脚本**，包含 Arch Linux 在构建软件包时需要的信息。
简而言之就是一个打包脚本，类似于Makefile，但是打成软件包而非直接安装到系统
## 优点
 1. 交给包管理器管理，方便升级，卸载，安装
 2. 不是所有用户都有工夫从源码编译，通过PKGBUILD可以加快软件分发
 3. 方便打包，无需手动配置环境与其他，自动化完成
## 缺点
 1. 在AUR中缺少监管，可能会有恶意软件与代码
 2. Arch系独占（也许可以移植  

## 教程开始
### 依赖的安装
```Bash
sudo pacman -S --needed base-devel
```
> 首先，确定你已安装必须的工具包。安装 `base-devel`应该足够了；它包含make和其它一些从源码编译时所需要的工具。  
 创建包的一个很重要的工具是`makepkg`（由**pacman**提供），它主要做以下工作：
> 1. 检查相关依赖是否安装。
> 2. 从指定的服务器下载源文件。
> 3. 解压源文件。
> 4. 编译软件并将它安装于伪root环境下。
> 5. 删除二进制文件和库文件的符号连接。
> 6. 生成包的meta文件。
> 7. 将伪root环境压缩成一个包文件
> 8. 将生成的包文件保存到配置的文件夹中（默认为当前工作目录）。

### 源代码
你需要知道你将要打包的程序的源代码的编译方式
> 下载你想打包的软件的源代码压缩包，解压，按照作者所说的步骤安装它。记录下在编译和安装软件过程中需要的所有命令或步骤。你将要在PKGBUILD文件中重复这些命令和步骤。  
大多数软件作者遵循三步走的安装惯例：  
> ```
> ./configure   
> make  
> make install  
> ```
> 这是一个能确保程序正常运行的好时机。
### PKGBUILD的编辑
>当你运行makepkg时，它会在当前工作目录寻找一个PKGBUILD文件。如果找到PKGBUILD文件，它会下载该软件的源代码，根据PKGBUILD文件中的指令编译它。<b>PKGBUILD中的指令必须能完全被Bash解释。</b>成功完成后，最后的二进制文件和包的元信息（即包的版本、依赖）被一起打包在`pkgname.pkg.tar.zst`文件包中，这个文件包可以使用`pacman -U <package file>`来安装。  
#### 变量部分
<details>
  <summary>展开查看</summary>
  
```bash
# Maintainer: Your Name <youremail@domain.com>
pkgname=NAME  #软件包名
pkgver=VERSION #软件包的版本号，应该与软件上游发布的版本号一致。变量的值可以由字母、数字和英文句点 .，下划线 _ 组成，但不能包含连字符(-)。如果上游版本号中使用了连字符，则应该用下划线 _ 来替代。
pkgrel=1 #软件的发布号。这通常是一个正整数，用来区分同一版本软件的多次构建。
pkgdesc="一个介绍用PKGBUILD" #软件包介绍
arch=('x86_64') #软件的架构
url="" #上游url，通常是仓库地址
license=('GPL') #许可证，如果不知道或者懒得填就写custom
groups=() #软件包所在的包组。例如：当你安装 base，它会安装 base 组里的所有包。
depends=() #运行时依赖，即必须依赖，可指定版本号
makedepends=() #编译时依赖，即运行时不需要但是用来编译需要
optdepends=() #可选依赖，用来拓展功能时依赖，没有但不影响的依赖，格式:  包名：作用。如'cups: printing support'
checkdepends=() #检查时依赖，可选，用于check（）步骤
provides=() #这个序列说明当前包能提供的功能，只要没有在 conflicts 序列中被标记，提供相同功能的软件包可以同时安装。
conflicts=() #与当前软件包发生冲突的包与功能的列表。安装此软件时，所有这个列表中的软件包，和提供这个功能的软件包都会被删除。可以像 depends 那样指定冲突包的版本号。
replaces=() #会因安装当前包而取代的过时的包的列表。
backup=() #当包被升级或卸载时，应当备份的文件（的路径）序列。使用没有绝对路径标识(/)的相对路径(如 etc/pacman.conf)
options=() #这个序列允许你重载 makepkg 的部分定义在 /etc/makepkg.conf 中的默认行为。
install=  #.install 脚本的名称。
changelog= #软件包的更新日志的文件名。
source=($pkgname-$pkgver.tar.gz) #构建软件包时需要的文件列表。它必须包含软件源的位置，大多数情况下是一个完整的 HTTP 或 FTP 地址。文件也可以放到与 PKGBUILD 文件相同目录，并将文件名添加到这个列表。
noextract=() #一个在 source 中列出，但不应该在运行 makepkg 时被解包的文件列表。这通常包括那些压缩文件不能被 /usr/bin/bsdtar 处理，或者本来就不需要解压、按照原样提供的文件。对于前者，需要将额外的解包工具（如unzip、p7zip，lrzip等）加入 makedepends 序列，并用 prepare() 函数手动解压。
sha256sums=() #sha256校验，通过updpkgsums填充
```
</details>  
各位可以根据自己的需要来使用变量，不需要的可以删去
详见 [Arch Wiki]("https://wiki.archlinux.org/title/PKGBUILD_(简体中文)")  

#### 函数部分
> 一共有五个函数, 以下按照它们执行的先后顺序列出。`package()` 函数是每个 PKGBUILD 中必须的函数,其余不存在的函数可以跳过。  
 
##### prepare()
此函数会执行用于预处理源文件以进行构建的命令, 例如 patching. 此函数执行在 `build()` 之前, 软件包解压之后. 如果解压过程被跳过 `(makepkg -e)`, 那么 `prepare()` 函数就不会被执行.
***该函数运行在 bash -e 模式下, 意味着任何以非零状态退出的命令都会造成该函数中止.***  

<details>
<summary>例：</summary>

```bash
prepare(){
    7z x "${_pkgname}-${pkgver}.exe" -o"tmp/bili" "\$PLUGINSDIR/app-64.7z" #手动解压noextract=()变量中的文件
    7z x "tmp/bili/\$PLUGINSDIR/app-64.7z" -o"tmp/bili" "resources"#手动解压noextract=()变量中的文件
    rm -rf "tmp/bili/\$PLUGINSDIR/app-64.7z" "tmp/bili/resources/elevate.exe"#删除不需要的文件
    mkdir -p tools #建立tools文件夹存放补丁脚本
    for file in *.sh app-decrypt.js bridge-decode.js js-decode.js; do
        mv $file tools;
    done #将注入脚本移动至tools文件夹
    mkdir -p res/scripts #建立res/scripts文件夹
    mv inject*.js res/scripts #放入注入用脚本
}
```
</details>

##### pkgver()
用于制作构建过程相同, 但源文件可能每天甚至每小时更新一次的软件包的时候  
多见于git等VCS包（即 什么什么什么-git）  
相对复杂，不讲了（用release不香吗）
##### build()
>现在你需要编写PKGBUILD文件中的build()函数。这个函数使用通用的shell命令来自动编译软件并创建软件的安装目录。这允许makepkg无需详查你的文件系统就可以打包你的软件。  
***在build()函数中第一步就是进入由解压源码包所生成的目录。 makepkg 会在执行 build() 函数之前更改当前目录为 $srcdir; 因此, 大多数情况下第一条命令是这样的(参考示例文件/usr/share/pacman/PKGBUILD.proto)：***  

>`cd "$srcdir/$pkgname-$pkgver"`  

>`cd "$srcdir/BBDown"`

以上两种进入源码目录的方式因源代码而异。
如：`source=("https://github.com/nilaoda/BBDown/archive/refs/tags/1.5.3.tar.gz")`
这是通过下载1.5.3版本发布时的源代码压缩包，makepkg默认自动解压tar格式。这个解压出来是`BBDown-1.5.3`这个文件夹，那么就是`cd "$srcdir/BBDown-$pkgver"`了
***切记，在PKGBUILD中，不要硬编码，用变量！！否则跟随上游时全是问题！！***
>现在，你需要把你当时手动编译软件时用到的命令一一列上。`build()`基本上会自动运行你当时手动输入的命令并在伪root环境下编译该软件。如果你要打包的软件使用了一个配置脚本，最好在配置中加上`--prefix=/usr`。许多软件都将自己安装到`/usr/local`下，我们仅仅推荐当你手动从源码安装时这么做。所有的Arch Linux软件包都应当使用`/usr`目录。

例：
```
./configure --prefix=/usr
make
```
***注意： 如果你的软件不需要构建任何东西, 请不要使用 build() 函数. 但package() 函数依然是必须的.***
##### check()
> 用来执行make check和其他一些例行测试的地方。如果不需要可以通过在 PKGBUILD/makepkg.conf 中使用 BUILDENV+=('!check') 或者给 makepkg 传入参数 --nocheck 来禁用它。

大部分时候用不到它，参考Arch Wiki
##### package()
> 最后一步就是把编译好的文件放到pkg文件夹——一个简单的伪root环境。pkg目录复制了根目录下软件安装路径的继承关系。如果你需要手动把文件放到根目录下，那么在这里你需要把文件放在pkg下相同的文件层级结构中。比如，你想把一个文件安装到/usr/bin，那么在伪root环境中对应的路径为$pkgdir/usr/bin。极少情况下的安装步骤需要用户手动复制大量的文件到某个地方。大部分软件安装时只需要调用make install即可。为了将软件安装到正确的路径，最后一行一般应该这样写：
> 
> `make DESTDIR="$pkgdir/" install`  
> ***注意： 有时候在`Makefile`里没有使用`DESTDIR`；你可能需要使用`prefix`来替代。如果软件包是用autoconf/automake来创建的，那就使用`DESTDIR`；如果DESTDIR不起作用，试试`make prefix="$pkgdir/usr/" install`。如果这还不起作用的话，你就需要深入检查软件的安装命令了。***

例：
1. ```Bash
   package() {
	install -Dm755 "BBDown" "$pkgdir/usr/bin/BBDown"
   }
   ```
2. ```Bash
   package() {
    install -Dm644 "${_pkgname}.desktop" "${pkgdir}/usr/share/applications/${_pkgname}.desktop"
    install -Dm644 "${_pkgname}.png" "$pkgdir/usr/share/icons/hicolor/512x512/apps/${_pkgname}.png"
    install -Dm644 "${srcdir}/app/app.asar" "${pkgdir}/usr/share/${_pkgname}/${_pkgname}.asar"
    install -Dm644 "${srcdir}/app/app-update.yml" "${pkgdir}/usr/share/${_pkgname}/app-update.yml"
    install -Dm755 "${_pkgname}" "${pkgdir}/usr/bin/${_pkgname}"
    cp -r "${srcdir}/app/extensions" "${pkgdir}/usr/share/${_pkgname}/extensions"
   }
   ```
   >makepkg --repackage 命令只运行package()函数,它只是将文件打包成软件包，并不运行编译过程。如果你只是更改了PKGBUILD中的依赖，用这个命令来打包可以节省很多时间。
## 测试PKGBUILD文件
>你在写PKGBUILD的 build()方法时，会想频繁的测试你所做的改动以确保没有bug。你可以在包含 PKGBUILD的目录下运行makepkg命令来确保没有问题。如果PKGBUILD没有错误，将会生成一个包，但是如果PKGBUILD被破坏或未完成，它将抛出一个错误。

推荐安装namcap这个包，以来检测PKGBUILD中的错误
```Bash
sudo pacman -S namcap
```
>如果运行makepkg 成功，在你工作的目录下将会生成一个名为`$pkgname-$pkgver.pkg.tar.gz`的新文件。这个文件可以使用`pacman -U` 或 `pacman -A`安装，你也可以将它加到本地或网上的软件仓库中。注意，一个包被构建并不代表你的工作就完成了！只有当所有文件的结构都正确才能确保完成，例如你给了一个不正确的前缀就不行。你可以使用pacman的查询功能显示软件包包含的文件及依赖的文件，然后将它于你认为正确的对比。`pacman -Qlp <package file>` 和`pacman -Qip <package file>` 可以完成这项工作。

>如果包看起来是正确的，那你的工作就完成了。但是如果你打算发布这个包或PKGBUILD，你就需要确认确认再确认包的依赖关系。  

>同样要确保安装的软件确实很完美的运行！如果你释放了一个包括所有必需文件的包，但是由于一些配置选项使它不能很好的工作，这真是让人恼火。如果你只是为你自己的系统安装这个软件，你就不必做这个质量保证了，因为只有你一个人需要忍受这些错误。

以上是Arch Wiki中对于检查编译出来的包的过程
>检查包的逻辑性  
确定包可以正常使用后，再使用namcap来检查错误：
```Bash
$ namcap PKGBUILD
$ namcap <package file name>.pkg.tar.zst
```
Namcap将会做以下工作：

1. 检查PKGBUILD文件里的一些常见错误
2. 用ldd扫描包中所有的ELF文件，自动报告缺失或可去除的依赖。
3. 启发式搜寻缺失或冗余的依赖。

>**要养成用namcap检查包的习惯，以避免提交包后再做修复的麻烦。**  

***但是namcap也不能全信，部分依赖它无法检测到！！！！在上游中明确要求的依赖可能检测不出来，所以以上游为准！！！***

### 注意事项
> 1. 在开始自动打包之前，请确保你至少已成功手动打包一次，除非你“很清楚”你正在做什么。不幸的是，虽然大多数软件作者遵循了三步走的安装惯例：`./configure; make; make install`，但事情并不都是这样的，有时候你不得不自己打补丁才能安装成功。经验是：如果你手动无法编译成功或者无法将软件安装到指定子目录下，那你就不必费心打包了。makepkg没有任何魔力能消除源代码的问题让你编译成功。  
> 2. 在一些情况下，你可能无法直接得到包的源码，可能需要使用`sh installer.run`这样的东西来工作。这时就需要你自己做很多工作了（比如读READMEs，安装指导，手册，或者Gentoo的ebuilds等等）。在一些很变态的情况下，你需要自己编辑源码才能正常安装。但是，makepkg需要完全自主运行，不能有用户的干预。因此，如果你想修改`makefiles`，你需要随`PKGBUILD`附上一个定制的补丁，然后在`prepare()`函数里安装这个补丁；或者你可以在`prepare()`函数里通过sed来修改。


# 下一期视频讲解AUR

