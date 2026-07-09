# GPFS-常见问题
## 集群启动问题
### 证书问题
集群启动报错示例：
```shell
/usr/lpp/mmfs/bin/tsgskkkm error: could not insert the key in the keystore (error 48)
```
命令查看证书过期：
```shell
openssl x509 -in /var/mmfs/ssl/id_rsa_committed.cert -test|grep "Not after"
```
&#8195;&#8195;结果显示的是过期的时间。先改时间，然后把CCR disable，启动集群，生成新证书，调回时间，系统恢复。处理官方参考：
- [The SSL certificate has expired](https://www.ibm.com/docs/en/storage-scale/5.1.2?topic=issues-ssl-certificate-has-expired)
- [IBM Storage Scale: An expired cluster key prevents nodes from connecting to the Storage Scale cluster resulting in admin commands failing to update the cluster configuration.](https://www.ibm.com/support/pages/ibm-storage-scale-expired-cluster-key-prevents-nodes-connecting-storage-scale-cluster-resulting-admin-commands-failing-update-cluster-configuration)
- [mmauth command](https://www.ibm.com/docs/en/storage-scale/6.0.0?topic=reference-mmauth-command)

进入到指定目录：
```shell
cd /var/mmfs/ssl​​​​​​
```
查看committed key的有效期：
```shell
/usr/lpp/mmfs/bin/mmcommon run mmgskkm print --cert id_rsa_committed.cert | grep Valid
```
查看新 key的有效期：
```shell
/usr/lpp/mmfs/bin/mmcommon run mmgskkm print --cert id_rsa_new.cert | grep Valid
```
如果新key还在有效期内，提交：
```shell
mmauth genkey commit
```
如果没有，创建后再提交：
```shell
mmauth genkey new
```

## 待补充