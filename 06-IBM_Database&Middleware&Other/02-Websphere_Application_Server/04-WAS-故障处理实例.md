# WAS-故障处理实例
记录几个故障处理实例。
## Out Of Memory
发生内存溢出后，根据之前说明收集相应的数据。
### 示例一：Java堆OOM
&#8195;&#8195;分析javacore，从javacore里看到Java堆已经扩展到最大堆4G，只剩11m，几乎用尽，此种类型属于Java堆OOM，javacore里面信息示例：
```
/javacore.20210108.200305.7955.0003.txt  |  8,022,601 bytes
Server Name: fxqapp02Node01Cell\fxqapp02Node01\server1
Dump Event "systhrow" (00040000) Detail "java/lang/OutOfMemoryError" "Java heap space" received
JRE 1.7.0 Linux amd64-64 build 20130421_145945 (pxa6470sr4fp1ifix-20130423_02(SR4 FP1+IV38579+IV38399+IV40208) )
OS Level : Linux 2.6.32-573.26.1.el6.x86_64
How Many : 8
-Xms512m
-Xmx4096m
-verbose:class,gc
-verbose:jni
Total memory free: 11,826,296 (0x0000000000B47478) = 11M
Total memory in use: 4,283,075,464 (0x00000000FF4A8B88) = 4085M
Total memory: 4,294,901,760 (0x00000000FFFF0000) = 4096M
```
分析heapdump​，可以看到内存使用情况，示例如下：
```
Problem Suspect 1
One instance of "org.apache.ibatis.executor.result.DefaultResultHandler" occupies 3,513,427,680 (87.77%) bytes. The memory is accumulated in one instance of "java.lang.Object[]".
Class Name                                                                 | Shallow Heap | Retained Heap | Percentage
-----------------------------------------------------------------------------------------------------------------------
org.apache.ibatis.executor.result.DefaultResultHandler @ 0x352e55c0 Unknown|           16 | 3,513,427,680 |     87.77%
'- java.util.ArrayList @ 0x352e7bd0                                        |           24 | 3,513,427,664 |     87.77%
   '- java.lang.Object[1215487] @ 0xb5e7e830                               |    4,861,968 | 3,513,427,640 |     87.77%
      |- java.util.HashMap @ 0x9466a448                                    |           48 |         3,120 |      0.00%
      |- java.util.HashMap @ 0x66296e10                                    |           48 |         3,104 |      0.00%
      |- java.util.HashMap @ 0x66296e40                                    |           48 |         3,104 |      0.00%
      |- java.util.HashMap @ 0x946592b8                                    |           48 |         3,104 |      0.00%
      |- java.util.HashMap @ 0x94669458                                    |           48 |         3,104 |      0.00%
......
      |- java.util.HashMap @ 0x946694e8                                    |           48 |         3,072 |      0.00%
      '- Total: 25 of 1,166,800 entries; 1,166,775 more                    |              |               |  
-----------------------------------------------------------------------------------------------------------------------
```
Javacore里看到有多个线程有类似的堆栈，例如：​
```
=====
3XMTHREADINFO      "WebContainer : 9" J9VMThread:0x00000000054B9200, j9thread_t:0x00007F516C0D11E0, java/lang/Thread:0x00000000129862D0, state:R, prio=5
3XMTHREADINFO3           Java callstack:
4XESTACKTRACE                at java/net/SocketInputStream.socketRead0(Native Method)
4XESTACKTRACE                at java/net/SocketInputStream.read(SocketInputStream.java:161(Compiled Code))
4XESTACKTRACE                at java/net/SocketInputStream.read(SocketInputStream.java:132(Compiled Code))
4XESTACKTRACE                at oracle/net/ns/Packet.receive(Bytecode PC:31(Compiled Code))
4XESTACKTRACE                at oracle/net/ns/DataPacket.receive(Bytecode PC:1(Compiled Code))
4XESTACKTRACE                at oracle/net/ns/NetInputStream.getNextPacket(Bytecode PC:48(Compiled Code))
......
```
&#8195;&#8195;在上面示例中，从heapdump分析结果来看是ibatis​相关"org.apache.ibatis.executor.result.DefaultResultHandler"里保存了大量的对象，占用较多内存，需要应用检查此对象为何占用内存比较多。       
&#8195;&#8195;遇到oom之后，Java的垃圾回收会尝试回收不使用的对象，释放资源；如果对象不再使用，可以被回收，有足够的资源来进行新对象的分配就可以恢复；但是如果Java堆里面的对象都是被占用的，那么垃圾回收是不会释放这些资源，​结果就是频繁的global gc，性能会比较差，server无法响应请求，可以查看垃圾回收日志查看使用曲线。       
​&#8195;&#8195;出现oom的时候数据库和网络方面是否正常？如果这么大的对象都是正常需求，比如高业务请求时需要申请这么多对象，那么可以通过增加server，或者增加Java堆大小的方式来规避oom。​    
&#8195;&#8195;另外，如果开启了开启了class 和jni的trace，如果在生成环境，并且问题已经解决，建议关闭，对性能会有一定的影响。​

## 待补充
