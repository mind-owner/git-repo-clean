## 介绍

`git repo-clean`是用Golang开发的具备Git仓库大文件扫描，清理，并重写commit提交记录功能的Git拓展工具。

## 依赖环境：
+ Golang >= 1.15

+ Git >= 2.24.0

## 安装

+ 二进制包安装

下载链接：https://gitee.com/oschina/git-repo-clean/releases/

+ 源码编译安装

```bash
$ git clone https://gitee.com/oschina/git-repo-clean
# 进入源码目录，编译
$ cd git-repo-clean
$ make
```

+ 安装

对于Linux环境
> sudo cp git-repo-clean $(git --exec-path)

类似的，对于Windows环境，将可执行文件`git-repo-clean`的路径放到系统$PATH路径中，
或者复制该可执行文件到`C:\Windows\system32`目录下即可。

安装完成后，执行如下命令检测是否安装成功：
> git repo-clean --version

注：在`Mac OS`上进行配置之后可能无法执行，需要授权，具体方式为：
**System Preferences** -> **Security & Privacy**
点击 Allow Anyway 始终允许即可:
![mac](docs/images/mac_setting.png)


## 使用

有两种使用方式，一种是命令行，一种是交互式。

目前选项有如下：
```bash
  -v, --verbose		show process information
  -V, --version		show git-repo-clean version number
  -h, --help		show usage information
  -p, --path		Git repository path, default is '.'
  -s, --scan		scan the Git repository objects
  -b, --branch		set the branch to scan, default is current branch
  -l, --limit		set the file size limitation, like: '--limit=10m'
  -n, --number		set the number of results to show
  -t, --type		set the file type to filter from Git repository
  -i, --interactive 	enable interactive operation
  -d, --delete		execute file cleanup and history rewrite process
```

**命令行式用法:**

![命令行式用法](docs/images/git-repo-clean-command-line.gif)

`git repo-clean --scan --limit=1G --type=tar.gz --number=1`
> 在仓库中使用命令行，扫描仓库当前分支的文件，文件最小为1G，类型为tar.gz，显示前1个结果

`git repo-clean --scan --limit=1G --type=tar.gz --number=1 --delete`
> 加上`--delete`选项，则会批量删除当前分支扫描出的文件，并重写相关提交历史

以上操作是假设在当前目录提交了大文件，然后需要在该分支进行删除。这个时候扫描的是当前分支的数据，而不是全部分支的数据，
这样做是为了加快扫描速度。如果想要清理其他分支的数据或者所有分支的数据，可以使用`--branch`选项，如`--branch=all`则
可以进行全扫描，会把所有分支上筛选出的数据清理掉。

`git repo-clean --scan --limit=1G --type=tar.gz --number=1 --delete --branch=all`
> 加上`--branch`选项，则会扫描所有分支的文件再执行删除，并重写相关提交历史

**交互式用法:**

![交互式用法](docs/images/git-repo-clean-interactive.gif)

`git repo-clean -i[--interactive]`
> 使用`-i` 选项进入交互式模式，此模式下，默认打开的开关有`--sacn`, `--delete`, `--verbose`

> 输入`git repo-clean`也可以直接进入交互模式

<!-- 进入交互模式后，首先提示如下：
```bash
$ git repo-clean -i
? 选择要扫描的文件的类型，如：zip, png:
? 选择要扫描文件的最低大小，如：1M, 1g:
? 选择要显示扫描结果的数量，默认3:
```
第一个问题，文件类型选择，默认`*`表示所有类型
第二个问题，指定文件大小的限制，默认1M。 注意，需要有单位，可选单位有B, K, M, G，不区分大小写
第三个问题，选择显示结果的数量，默认显示前三个结果。

用户选择好了三个条件后，便开始扫描仓库，对于较大的仓库，这可能会花一段时间。

```bash
开始扫描(如果仓库过大，扫描时间会比较长，请耐心等待)...
扫描完成!
注意，同一个文件因为版本不同可能会存在多个，这些是占用 Git 仓库存储的主要原因
请根据需要，通过其对应的ID进行选择性删除，如果确认文件可以全部删除，全选即可。
079266398882a970242daaab4c53956da2a3f2b6  954371 字节  po/bg.po
29ba1c82fed9ee7837b6d84f4966fce2724f5c1f  940063 字节  po/bg.po
26998105879cc2113cb8e5dfed2bdec02820ab48  920035 字节  po/bg.po
? 请选择你要删除的文件(可多选):  [Use arrows to move, space to select, <right> to all, <left> to none, type to filter, ? for more help]
> [ ]  079266398882a970242daaab4c53956da2a3f2b6: po/bg.po
  [ ]  29ba1c82fed9ee7837b6d84f4966fce2724f5c1f: po/bg.po
  [ ]  26998105879cc2113cb8e5dfed2bdec02820ab48: po/bg.po
```
箭头`>`指向当前可选的文件，通过键盘的上下方向键可选择需要删除的文件，向右按键可以全选，相反向左按键则是全取消。

选择后便进入执行阶段，同样地，根据仓库大小情况，可能需要等待一段时间。



扫描结果显示了文件的ID，大小和文件名。
第四个问题，对结果进一步选择。该选择可以多选，使用向上、向下按键可以指定不同文件，使用向右按键可以全选，使用向左按键可取消全选。

选中完成后，会有二次确认：
```bash
? 请选择你要删除的文件(可多选):
079266398882a970242daaab4c53956da2a3f2b6: po/bg.po
29ba1c82fed9ee7837b6d84f4966fce2724f5c1f: po/bg.po
26998105879cc2113cb8e5dfed2bdec02820ab48: po/bg.po
? 以上是你要删除的文件，确定要删除吗?
(y/N)
```
可以选择y, yes, 或者n, no, 均不区分大小写。 -->

<!--
**LFS使用流程**
+ download and install
> https://github.com/git-lfs/git-lfs/releases
+ set up in machine
> git lfs install
+ track file
> git lfs track "*.mp4"
+ modify .gitattributes
> git add .gitattributes
+ normal git operation and the tracked file will upload to LFS server
> git add && git commit && git push


git lfs可以跟踪仓库中新加入的文件，而不会追踪历史提交中的文件
已经存在于提交历史中的大文件，如果想要使用LFS，需要用迁移：
> https://help.aliyun.com/document_detail/206890.html?spm=a2c4g.11186623.0.nextDoc.778d3f107TbPkx
+ mirate existing file in history
> git lfs migrate import --include="*.psd" --everything
+ push to remote
> git push origin main

使用LFS将历史中的某个文件纳入到LFS的追踪管理，此时会生成`.git/lfs/objects`保存该文件对象
然后对仓库过滤，删除文件
然后强制推送到远程仓库 -->


## 代码结构

+ main.go       | 程序主入口
+ options.go    | 程序选项参数处理
+ cmd.go        | 交互式命令处理
+ repository.go | 仓库扫描相关处理
+ fastexport.go | 启动git-fast-export进程
+ fastimport.go | 启动git-fast-import进程
+ parser.go     | 仓库数据解析
+ filter.go     | 仓库数据过滤
+ git.go        | Git对象相关
+ utils.go      | 一些有用帮助函数


## TODO
- [ ] 支持在同一个选项中有多个选择，如：--type=jpg, png, mp4
- [ ] 支持指定更准确范围的文件大小
- [ ] 对特殊文件名的处理
- [ ] 增加处理过程的进度提示信息，时间消耗信息等
- [ ] 对用户提供的仓库做进一步检测，如检测`.git`与工作目录是否分离
- [x] 提供选项给git-fast-export，对特定分支进行筛选，而不是所有分支
- [ ] 考虑重写历史对签名的影响
- [ ] 考虑重写历史对PR的影响

## BUGs
+ 如果仓库中存在nested tag, 则在清理过程中会出现错误，如：`error: multiple updates for ref 'refs/tags/v1.0.1-pointer' not allowed`, 这会导致文件删除失败。暂时处理方式是一旦检测到这种情况，就退出程序，并显示警告信息。

## NOTE
+ 目前只关注文件本身，所以扫描时只关注blob类型对象
+ 从Git 2.32.0起，`git-rev-list`具备`--filter=object:type`选项，在扫描时能够过滤特定类型，这样能够加快处理过程，后续考虑使用较新的Git版本。
+ 考虑到有种情况是扫描出来的大文件(blob)只存在历史中，此时如果想删除，指定文件名是找不到该文件的。因此，实际在做文件删除时，应该指定为blob hash值，也就是虽然看起来用户选择的是文件名，实际上使用它对应的blob hash ID。
+ 为了防止用户没有对仓库进行备份，当用户在进行删除重、写历史操作时，有一种策略是首先进行`fresh clone safety check`，即检查正在操作的仓库是不是刚克隆的，如果不是新鲜克隆的，则很有可能该仓库是单例仓库，没有副本，没有原始仓库，所以进行一些操作是非常危险的，此时需要提示用户正在非副本仓库中进行危险操作。不是完全拒绝的，用户还可以使用选项`--force`强制进行操作 ([参考](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html#FRESHCLONE))。
虽然这种检测不是绝对准确的，但它是有用的。
具体检测方法是查看`git reflog`命令的结果，是否只包含一项。如果超过一项，则会被认为不是刚克隆的仓库。

## 技术原理
见 [docs/technical.md](docs/technical.md)



## 测试

+ 普通仓库

- [x] 单分支，最末端的文件及其commit
- [x] 单分支，中间单个文件及其commit
- [x] 单分支，中间连续多个文件及其commit
- [x] 单分支，中间间断的文件及其commit
- [x] 多分支，first parent(and its blob)(from-ref)
- [x] 多分支，second parent(and its blob)(merge-ref)
- [x] 多分支，all parents
- [x] 多分支，after merge point
- [x] 可执行文件
- [x] 压缩文件
- [x] 多媒体文件

+ 大型仓库

极端情况下，在仓库中加入一个文件大小为1216179567 byte(1.2G)的压缩文件, 作为仓库最近一次提交(最后被扫描)，从仓库删除，最快不到10s。
```bash
$ time git repo-clean --scan --verobse --limit=1g --number=3 --delete
Start to scan repository:
[0]: 449a189d6fb67b3dc0cfcce086847fc93ac86fd0 1216179567 gitaly-dev.tar.gz
git repo-clean -s -v --limit=1g -n=3  9.87s user 7.62s system 150% cpu 11.651 total
```
以上是理想情况，即在仓库历史中没有加入其它二进制文件，否则过程也会比较长，这取决于仓库中的数据大小。


## Licensing
git repo-clean is licensed under [Mulan PSL v2](LICENSE)