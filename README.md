# resources_packer 资源打包工具

resources_packer是一个简单实用的游戏资源更新包制作工具。


## 应用场景

玩游戏的时候经常遇到这样的一个场景，打开游戏，提示正在下载游戏资源更新包。

假设本地游戏版本1.0.0， 提示下载1.0.0-1.zip更新包，那么这个zip包当中，包含的是的1.0.0版本与1.0.1版本差异的文件；

假设本地游戏版本1.0.1， 很久没有玩游戏，提示正在更新1.0.1-10.zip更新包，这个zip包当中，包含了从1.0.1版本到1.0.10版本的所有差异文件。

那么，像1.0.1-10.zip这种差异包，是如何生成？人工挑出差异文件，制作zip包？有没有自动化的方法？

resources_packer就是一个生成版本差异资源包自动化工具。

## Example

假设, 所需打包的目录testdir

- testdir
	- test1.txt
	- test2.txt

执行命令, 操作行为check
```shell
resources_packer -paths testdir -package-name TestPackage -artifact-dir testdirOutput -action check

```

这时候，与testdir目录平级的testdirOutput目录下，将生成:

- testdirOutput            			--> 输出目录
	- TestPackage.0.zip 				--> 生成的0号资源包，包含test1.txt, test2.txt两个文件
	- TestPackage.0.zip.md5 			--> 0号资源包的md5文本
	- TestPackage.resource_version.txt  --> TestPackage这个系列包的最新版本（比如当前内容为0）


这时候，我们已经拥有了0号资源包了。我们做一些差异文件：往testdir里新加文件test3.txt。
testdir目录的结构将变化成：
- testdir
 	- test1.txt
 	- test2.txt
	- test3.txt

执行命令, 操作行为pack：
```shell
resources_packer -paths testdir -package-name TestPackage -artifact-dir testdirOutput -action pack
```


这时候，与testdir目录平级的testdirOutput目录下，将变成:

- testdirOutput            			--> 输出目录
 	- TestPackage.0.zip 				--> 生成的0号资源包全量包，包含test1.txt, test2.txt两个文件
 	- TestPackage.0.zip.md5 			--> 0号资源包的md5文本
	- TestPackage.0-1.zip 				--> 0号资源包与1号资源包的差异包
	- TestPackage.0-1.zip.md5
	- TestPackage.1.zip 				--> 1号资源包的全量包
	- TestPackage.1.zip.md5
 	- TestPackage.resource_version.txt  --> TestPackage这个系列包的最新版本（现在内容为1）

好了，一个差异资源包0-1就这样生成了。

为什么同时有全量包和差异包？
因为差异包的生成，是基于两个全量包之间进行比较的。

## 生成的zip包，有什么特别？

- .manifest        记录全量的文件（不论差异与否，所有的目录文件）的大小、MD5等文件信息
- OTHER1
- OTHER2
- OTHER3/OTHER4.txt

简而言之，生成的资源zip，只有根路径的.manifest文件是由resources_packer生成的，其余的所有文件，都是您所设置的资源文件。


## 命令行如何获取帮助?

你可以同通过命令行获取到详细帮助:

![screenshot.png](screenshot.png)

Linux/Unix下：
```shell
resources_packer --help
```

Windows下，需要预先安装Python环境：
```shell
resources_packer.bat --help
```
或
```shell
python resources_packer --help
```

## 打包操作模式

resources_packer有几种不同的操作模式，来适应不同的实际情况。

### check检查或创建
常用。检查是否存在当前版本的资源包，不存在则init创建。存在则不做任何事。

假设，当前没有生成任何资源包，则生成0号zip包。
假设，当前已有0号zip包，则不做任何事。

### pack差异打包
核心。生成差异资源包时，提升资源版本号。

假设当前已0号资源包， 执行pack行为，将会识别0号资源包的文件，与当前所有资源文件进行比较。差异的文件，生成1号资源包。


### init初始化

强制重新生成当前版本的差异包。

假设当前资源版本10， 执行init后，重新生成10版本，覆盖之前的10号资源包。

init一般很少手动执行，一般在你不想修改当前资源包版本号，但又想重新添加差异文件的时候使用。

### clean清理

清理旧版本的zip包。
假设当前最新版本3，那么其实只会用到1-3, 2-3, 0-3几个zip包。像0-2, 1-2这种旧版本的zip包，已经没有用处了。执行clean，它们将被归档到archives目录，考虑到稳健性，不做删除操作。