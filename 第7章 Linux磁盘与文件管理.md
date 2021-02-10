# 第7章 Linux磁盘与文件管理

## 1.磁盘的物理组成

- 圆形的磁盘盘(主要记录数据的部分)；
- 机械手臂，与在机械手臂上的磁盘读取头(可擦写磁盘盘上的数据)；
- 主轴马达，可以转动磁盘盘，让机械手臂的读取头在磁盘盘上读写数据。

## 2.文件系统特性

磁盘分区完毕后还需要进行格式化(format)，之后操作系统才能够使用这个分割槽。 为什么需要进行格式化呢？这是因为每种操作系统所配置的文件属性/权限并不相同， 为了存放这些文件所需的数据，因此就需要将分割槽进行格式化，以成为操作系统能够利用的文件系统格式(filesystem)。

## 3.inode表格记录的文件数据

- 该文件的存取模式(read/write/excute)；
- 该文件的拥有者与群组(owner/group)；
- 该文件的容量；
- 该文件创建或状态改变的时间(ctime)；
- 最近一次的读取时间(atime)；
- 最近修改的时间(mtime)；
- 定义文件特性的旗标(flag)，如 SetUID；
- 该文件真正内容的指向 (pointer)；

## 4.超级模块（Superblock）记录的信息

- block 与 inode 的总量；
- 未使用与已使用的 inode / block 数量；
- block 与 inode 的大小 (block 为 1, 2, 4K，inode 为 128 bytes)；
- filesystem 的挂载时间、最近一次写入数据的时间、最近一次检验磁盘 (fsck) 的时间等文件系统的相关信息；
- 一个 valid bit 数值，若此文件系统已被挂载，则 valid bit 为 0 ，若未被挂载，则 valid bit 为 1 。

## 5.filesystem 大小与磁盘读取效能：

另外，关于文件系统的使用效率上，当你的一个文件系统规划的很大时，例如 100GB 这么大时， 由于硬盘上面的数据总是来来去去的，所以，整个文件系统上面的文件通常无法连续写在一起(block 号码不会连续的意思)， 而是填入式的将数据填入没有被使用的 block 当中。如果文件写入的 block 真的分的很散， 此时就会有所谓的文件数据离散的问题发生了。

如前所述，虽然 ext2 在 inode 处已经将该文件所记录的 block 号码都记上了， 所以数据可以一次性读取，但是如果文件真的太过离散，确实还是会发生读取效率低落的问题。 因为磁盘读取头还是得要在整个文件系统中来来去去的频繁读取。 那么可以将整个 filesystme 内的数据全部复制出来，将该 filesystem 重新格式化， 再将数据复制回去即可解决这个问题。

此外，如果 filesystem 真的太大了，那么当一个文件分别记录在这个文件系统的最前面与最后面的 block 号码中， 此时会造成硬盘的机械手臂移动幅度过大，也会造成数据读取效能的低落。而且读取头在搜寻整个 filesystem 时， 也会花费比较多的时间去搜寻。因此， partition 的规划并不是越大越好， 而是要针对主机用途来进行规划。

## 6.Hard Link 

文件名只与目录有关，但是文件内容则与 inode 有关。那么想一想， 有没有可能有多个档名对应到同一个 inode 号码呢？有的！那就是 hard link 的由来。 所以简单的说：hard link 只是在某个目录下新增一笔档名链接到某 inode 号码的关连记录而已。

## 7.Symbolic Link

相对于 hard link ， Symbolic link 可就好理解多了，基本上， Symbolic link 就是在创建一个独立的文件，而这个文件会让数据的读取指向他 link 的那个文件的档名。

## 8.swap

- 1. 使用 dd 这个命令来新增一个 128MB 的文件在 /tmp 底下：

```
[root@www ~]# dd if=/dev/zero of=/tmp/swap bs=1M count=128 128+0 records in 128+0 records out 134217728 bytes (134 MB) copied, 1.7066 seconds, 78.6 MB/s [root@www ~]# ll -h /tmp/swap -rw-r--r-- 1 root root 128M Oct 28 15:33 /tmp/swap 
```



------

- 2. 使用 mkswap 将 /tmp/swap 这个文件格式化为 swap 的文件格式：

```
[root@www ~]# mkswap /tmp/swap Setting up swapspace version 1, size = 134213 kB # 这个命令下达时请『特别小心』，因为下错字节控制，将可能使您的文件系统挂掉！ 
```



------

- 3. 使用 swapon 来将 /tmp/swap 启动啰！

```
[root@www ~]# free             total       used       free     shared    buffers     cached Mem:        742664     450860     291804          0      45584     261284 -/+ buffers/cache:     143992     598672 Swap:      1277088         96    1276992 [root@www ~]# swapon /tmp/swap [root@www ~]# free             total       used       free     shared    buffers     cached Mem:        742664     450860     291804          0      45604     261284 -/+ buffers/cache:     143972     598692 Swap:      1408152         96    1408056 [root@www ~]# swapon -s Filename                 Type            Size    Used    Priority /dev/hdc5                partition       1020088 96      -1 /dev/hdc7                partition       257000  0       -2 /tmp/swap                file            131064  0       -3 
```



------

- 4. 使用 swapoff 关掉 swap file

```
[root@www ~]# swapoff /tmp/swap [root@www ~]# swapoff /dev/hdc7 [root@www ~]# free             total       used       free     shared    buffers     cached Mem:        742664     450860     291804          0      45660     261284 -/+ buffers/cache:     143916     598748 Swap:      1020088         96    1019992  <==回复成最原始的样子了！
```

