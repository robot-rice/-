# #***Linux学习笔记***

## ##Linux基础

### ###Linux基本命令

**我创建了一个learn-linux来做示例**

![img](https://robot-squid.oss-cn-beijing.aliyuncs.com/undefined%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-12-05%20172145.png)

1.cd：改变当前目录，这里cd /home使当前目录变为home

2.ls：列出当前目录内容

3.mkdir：创建新目录，这里创建了一个名为learn-linux的目录。使用ls命令列出home下面的目录（又是执行命令需要权限，提示permission denied 这时输入密码不会显示但其实已经输入了，输完密码后获得权限就可以执行了）

![img](https://robot-squid.oss-cn-beijing.aliyuncs.com/undefined%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-12-05%20173156.png)

4.这里切换到learn-linux下面，并且用vim learn 用vim编辑器打开一个名叫learn的文件。这样就可以使用vim编辑器编辑learn的内容

![img](https://robot-squid.oss-cn-beijing.aliyuncs.com/undefined%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-12-05%20173744.png)

![img](https://robot-squid.oss-cn-beijing.aliyuncs.com/undefined%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-12-05%20174633.png)

5.cat：查看文件内容。用：wq保存更改并推出vim

6.rm：删除文件。用这个命令删除learn文件

7.rmdir：删除空目录。这个命令只能删除空目录，如果目录里有文件，必须先删除目录里的文件

8.mv：移动或重命名文件或目录   例如要将file从home下的file-operation移动到home下的tmp就可以用mv

​    mv home/operation/file home/tmp

9.tar：解压缩命令    tar -czvf b.fasta.tar.gz -8 /home/file-operation/b.fasta     -c表示创建一个新的归档文件 z    	表示用gzip压缩文件 8表示压缩级别为8  v表示在过程中显示详细信息  f表示指定归档文件的名称，我这里用文	件径指向文件   

### ###Linux软件包管理器

1.apt update：更新软件包列表

2.apt upgrade：升级软件包

3.apt install <package_name>：安装指定软件包

## ##vim基本使用

### ###vim指令

1.x删除光标所在的字符

2.i开始插入字符，esc退出编辑

3.A添加，在句子末开始添加esc退出

4.dw从光标处删除至一个单词的末尾，此时光标在下一个单词的第一个第一个字母

​	de从光标处删除至第一个单词末尾，此时光标在下一个单词的第一个字母的前一格

​	d$从当前光标删除到行末

5.2w向前移动两个单词

​	3e使光标向前移动到第三个单词末尾

​	上面这两个指令数字可变

6.u撤销指令

7.r加要替换的字符替代光标所在字符

8.cw或ce改变文本直到一个单词的末尾，删除并插入
	c$从光标处删除至末尾

### ###vim插件

下载插件管理器，如vim-plug

在~/.vimrc中配置（顶部）

> call plug#begin('~/.vim/plugged')
>
> Plug 'xxxxxxx'call
>
> plug#end()
>
> > 不同插件管理器格式有一点差别

在vim中输入

> :PlugStatus     查看插件状态
>
> ：PlugInstall     安装在vimrc声明的插件

![img](https://robot-squid.oss-cn-beijing.aliyuncs.com/undefined%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202024-12-05%20201423.png)

（auto-pairs下了好几次没下下来，还有下面的vim-im select(其实我不记得这个是哪个插件了)orz）