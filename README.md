# git 管理二进制文件
本文档将逐步带你体验 git 的大文件管理方式。

- 环境: windows10 64位 cmd
- git版本: git version 2.18.0.windows.1

## 创建到推送
创建二进制文件，修改并推送到远程。

``` batch

: 初始化项目
git init

: 创建 100k 大小文件模拟二进制文件 和普通文本
fsutil file createnew isbin.bin 102400
cd.>text.txt

: 开启 lfs 功能及文件追踪
git lfs install
git lfs track "isbin.bin"
: --------------- 大小 119 KB ---------------  [.git .gitattributes isbin.bin text.txt]

: 提交文件
git add isbin.bin text.txt .gitattributes
git commit -m "初始化"
: --------------- 大小 221 KB ---------------

: 修改文件
echo a>>isbin.bin
echo a>>text.txt
git add isbin.bin text.txt
git commit -m "添加a"
: --------------- 大小 321 KB ---------------

: 修改文件
echo b>>isbin.bin
echo b>>text.txt
git add isbin.bin text.txt
git commit -m "添加b"
: --------------- 大小 422 KB ---------------  通过上面的大小发现，每次提交，二进制文件体积还是会增大一倍。


: 推送到远端
git.exe push --progress "https://github.com/wll8/bin" master

```

## 克隆到推送
从远程克隆，修改并推送。

``` batch

: 从远端克隆
git clone https://github.com/wll8/bin bintest
cd bintest
: --------------- 大小 222 KB ---------------  上传 git 远程的二进制文件只保存最新的一份， 其他的在 lfs 服务器上


: 测试回退到上个版本
git reset --hard HEAD~1
: --------------- 大小 322 KB ---------------  这时候的大小说明回退版本时在 lfs 上下载该版本的文件到本地， 本地体积增加
git reset --hard HEAD~1
: --------------- 大小 422 KB ---------------


: 修改文件
git reset --hard origin/HEAD
echo c>>isbin.bin
echo c>>text.txt
git add isbin.bin text.txt
git commit -m "添加c"
: --------------- 大小 524 KB ---------------

: 推送到远端
git.exe push --progress "https://github.com/wll8/bin" master
echo c>>isbin.bin

: 再来一次修改
echo d>>isbin.bin & echo d>>text.txt & git add isbin.bin text.txt & git commit -m "添加d" & git.exe push --progress "https://github.com/wll8/bin" master
: --------------- 大小 624 KB ---------------

```

## 再次克隆
验证经过多次推送后的大小变化。

``` batch

: 从远端克隆
git clone https://github.com/wll8/bin bintest2
cd bintest2
: --------------- 大小 223 KB ---------------  看得出来多次推送也只在 git 上保存最后一次二进制文件

```

## 总结
经测确实是有效减少了 git 仓库的体积。

LFS(Large File Storage) 大文件存储是 git 的一个扩展。

在 git `拉取` 或 `推送` 时在 lfs 服务器 `下载` 或 `上传`， 也就是没有把每个版本保存在 github 服务器上。

其实违背了 git 的理念， 它依赖 lfs 胳， `本地并不是一个完整的仓库， 而且上传到 lfs 服务器 比 github 还慢` 。
