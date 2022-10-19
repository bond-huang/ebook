# SVC-池 mdisk 卷
## Pool
## Mdisk
## Volume
### Image mode volumes
&#8195;&#8195;`Image mode volumes`映像方式卷是与一个`MDisk`有直接关系的特殊卷。`Image`模式通过使用虚拟化提供从`MDisk`到卷的直接逐块转换。最常见的用例是将数据从旧的（通常是非虚拟化的）存储迁移到基于SVC的虚拟化基础架构。此模式是为了满足以下主要使用场景:
- 映像模式支持对已包含直接写入而非通过SVC写入的数据的MDisk进行虚拟化。相反，它是由直接连接的主机创建的
- 此模式使客户端能够以最短的停机时间将SVC插入现有存储卷或LUN的数据路径
- 映像模式使由SVC管理的卷能够与底层RAID控制器提供的本机复制服务功能一起使用。为避免在以这种方式使用 SVC时丢失数据完整性，禁用卷的 SVC缓存非常重要
- SVC提供迁移到映像模式的能力，这使SVC能够导出卷并直接从路径中没有SVC的主机访问它们

官方参考链接：[SVC Image mode volumes](https://www.ibm.com/docs/en/sanvolumecontroller/8.4.x?topic=volumes-image-mode)

## 待补充