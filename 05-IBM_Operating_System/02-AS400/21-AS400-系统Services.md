# AS400-系统Services
官方文档链接：[IBM i 7.5 IBM i Services](https://www.ibm.com/docs/en/i/7.5?topic=optimization-i-services)

## Performance Services
返回收集服务配置属性：
```sql
SELECT * FROM QSYS2.COLLECTION_SERVICES_INFO
```
&#8195;&#8195;`COLLECTION_SERVICES_INFO`视图返回收集服务的配置属性。视图中的列返回的值与`CFGPFRCOL`(Configure Perf Collection)命令和`QypsRtvColSrvAttributes`(Retrieve Collection Services Attributes)API返回的值密切相关。调用者必须对`QSYS/QYPSCOLL`服务程序有`*USE`权限或通过`QPMCCFCN`授权列表获得授权。
## 待补充