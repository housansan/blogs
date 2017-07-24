 Mac下如何不借助第三方工具实现NTFS分区的可写挂载
问题背景

我想很多使用Mac的同学都会遇到读写NTFS磁盘的问题，因为默认情况下Mac OSX对NTFS磁盘的挂载方式是只读(read-only)的，因此把一个NTFS格式的磁盘插入到Mac上，是只能读不能写的，用起来很是不便。

因此也就出现了一些第三方工具，例如Tuxera NTFS for Mac、Paragon NTFS for MAC等，这些工具都可以实现Mac下NTFS的写操作，但是这些工具都是收费的，当然也有些破解的版本，但是破解软件毕竟存在安全风险，so，I don't really like that.

此外Tuxera也提供了开源的NTFS读写方案：NTFS-3G，基于GPL授权的NTFS-3G也可以通过用户空间文件系统在Mac OS X上实现NTFS分区的读写。这个方案也比较不错，只是需要简单的安装，本文不再展开。具体的可以参考官方链接：http://www.tuxera.com/community/open-source-ntfs-3g/

但其实，我们完全可以不借助任何第三方工具就能解决这个问题，因为其实OSX原生就是支持NTFS的。后来由于微软的限制，苹果把这个功能给屏蔽了，但是我们可以通过命令行手动打开这个选项。

 
How to do that?

mount查看磁盘挂载情况
1
2
3
	
thatsitdeMacBook-Pro:~ thatsit$ mount
/dev/disk4s2 on /Volumes/Untitled (ntfs, local, nodev, nosuid, read-only, noowners)
thatsitdeMacBook-Pro:~ thatsit$

 

这时候如果我们要实现/dev/disk4s1分区的可写挂载，我们只需要做做一下两个步骤：卸载、重新挂载

卸载
1
	
thatsitdeMacBook-Pro:~ thatsit$ sudo umount /Volumes/Untitled/

重新挂载
1
	
thatsitdeMacBook-Pro:~ thatsit$ sudo mount -t ntfs -o rw,auto,nobrowse /dev/disk4s2 /Volumes/Udisk/

　　

这时候/dev/disk4s2已经以读写的方式挂载到/Volumes/Udisk/了，下面我们来进行挂载结果的确认

确认挂载
1
2
3
	
thatsitdeMacBook-Pro:~ thatsit$ mount|grep ntfs
/dev/disk4s2 on /Volumes/Udisk (ntfs, local, noowners, nobrowse)
thatsitdeMacBook-Pro:~ thatsit$

测试写入
1
2
3
4
5
6
7
8
9
10
11
12
	
thatsitdeMacBook-Pro:~ thatsit$ cd /Volumes/Udisk/
thatsitdeMacBook-Pro:Udisk thatsit$ touch test_writing
thatsitdeMacBook-Pro:Udisk thatsit$ ll|grep test_writing
-rwxr-xr-x 1 thatsit staff 0 12 24 17:14 test_writing
thatsitdeMacBook-Pro:Udisk thatsit$
thatsitdeMacBook-Pro:Udisk thatsit$ echo heheda >> test_writing
thatsitdeMacBook-Pro:Udisk thatsit$ cat test_writing
heheda
thatsitdeMacBook-Pro:Udisk thatsit$
thatsitdeMacBook-Pro:Udisk thatsit$ ll|grep test_writing
-rwxr-xr-x 1 thatsit staff 7 12 24 17:15 test_writing
thatsitdeMacBook-Pro:Udisk thatsit$

上述挂载参数的详细说明：
1
2
3
4
5
6
7
8
9
10
11
12
13
	
<strong>mount -t ntfs -o rw,auto,nobrowse /dev/disk4s2 /Volumes/Udisk/</strong>
 
-t ntfs # 执行要挂载的分区文件系统格式
-o # 执行挂载的选项
rw # read-write，以读写的方式挂载
auto # 自动检测文件系统,此参数可以省略
nobrowse # 这个选项非常重要，因为这选项指明了在finder里不显示这个分区，只有打开了这个选项才能将磁盘以读写的方式进行挂载
/dev/disk4s2 # 要挂载的分区，也就是我们在mount命令中看到的盘符
/Volumes/Udisk/ # 挂载点
 
* /Volumes/Udisk这个目录是需要存在的，如果不存在需要手动创建下：sudo mkdir /Volumes/Udisk
* 如果目录不存在会收到如下报错：
mount: realpath /Volumes/Udisk: No such file or directory

　　

通过上面的测试大家也看到了，此时的NTFS分区已经是可写的了；但是这个时候还存在另一个小问题，因为我们在挂载的时候使用nobrowse选项，这个分区在finder里是不显示的。

总会有些同学不习惯一直使用命令行进行操作，所以需要高解决这个问题：
解决办法其实很简单，因为这个分区是挂/Volumes下的，我们把这个目录在桌面做一个软链接就OK了。

创建软链接
1
2
3
	
thatsitdeMacBook-Pro:Udisk thatsit$ sudo ln -s /Volumes/Udisk/ ~/Desktop/Udisk
Password: # <----输入用户密码
thatsitdeMacBook-Pro:Udisk thatsit$

效果如下：桌面上会显示如下的盘符，点击即可进入Finder

http://www.cnblogs.com/thatsit/p/6218117.html
