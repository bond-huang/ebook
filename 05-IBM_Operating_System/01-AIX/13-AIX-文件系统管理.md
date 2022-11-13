# AIX-文件系统管理
## 文件系统其它
### superblock
#### superblock查看
官方参考链接：
- [IBM Support Where are the superblocks?](https://www.ibm.com/support/pages/node/670133?mhsrc=ibmsearch_a&mhq=superblock)

&#8195;&#8195;每个超级块的前4个字节包含该类型文件系统的`Magic`。这就是`fsck`和`mount`实用程序如何知道它们是什么类型的文件系统。JFS和JFS2类型的文件系统`Magic`：
- Hex: 4A324653 Char: J2FS = JFS2 filesystem
- Hex: 43218765 = Regular JFS filesystem
- Hex: 65872143 = Large-file enabled JFS filesystem

##### JFS2文件系统
JFS2的Primary superblock块位于lv(0x8000)的32768字节(32kb)处：
```
# lquerypv -h /dev/fslv05 8000 100
00008000 4A324653 00000001 00000000 4845D182 |J2FS........HE..|
```
JFS2的Secondary superblock块位于lv(0xF000)的61440字节(60kb)处：
```
# lquerypv -h /dev/fslv05 F000 100
00008000 4A324653 00000001 00000000 4845D182 |J2FS........HE..|
```
##### JFS文件系统
JFS文件系统中，它们位于块1(4096 dec=0x1000)：
```
# lquerypv -h /dev/lv00 1000 10
00001000 65872143 00000000 00001000 00000001 |e.!C............|
```
Secondary superblock在块31处(4096*31=126976 dec=0x1F000)：
```
# lquerypv -h /dev/lv00 1F000 10
0001F000 65872143 00000000 00001000 00000000 |e.!C............|
```
## 待补充