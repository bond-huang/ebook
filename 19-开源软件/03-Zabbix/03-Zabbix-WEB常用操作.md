# Zabbix-WEB常用操作
## 监控配置
### 自动发现配置
&#8195;&#8195;在Zabbix中配置自动发现，可以让系统自动识别和添加新主机、服务或资源，从而简化大规模监控环境的管理。以下是一般性的步骤示例：
- 登录到Zabbix Web界面：使用管理员账户登录到Zabbix Web界面。
- 选择自动发现规则：
  - 在左侧导航栏中，点击`配置`
  - 然后选择`动作`
  - 然后选择`发现动作`选项。
- 创建自动发现规则：
  - 点击页面右上角的`创建动作`按钮。
  - 在`名称`字段中输入规则的名称。
  - 在 "IP range" 或 "DNS name" 字段中指定待发现的 IP 范围或域名。
  - 配置其他发现规则的参数，如间隔时间、Agent 等。
  - 配置操作，如添加主机的组、模板等。
- 保存配置：完成自动发现规则和动作的配置后，点击 `添加`保存配置。
- 验证配置：
  - 系统将按照自动发现规则定期进行发现，识别并添加新的主机或服务。
  - 在"Monitoring"（监控）选项卡下的 "Latest data"（最新数据）中，可以看到新添加的主机和监控项。

### 开启自定义监控
开启自定义监控：
- 登录到Zabbix Web界面
- 选择主机：在左侧导航栏中，点击"Configuration"（配置），然后选择"Hosts"（主机）选项。
- 选择要配置的主机：从主机列表中选择你想要配置自定义监控的主机，然后点击主机名称进入主机详情页。
- 添加监控项：
  - 在主机详情页的上方，点击"Items"（监控项）选项卡。
  - 点击页面右上角的"Create Item"（创建监控项）按钮。
  - 在"Key"字段中输入一个唯一的标识符，用于在主机上标识这个自定义监控项。
  - 选择"Type"（类型），例如数字、文本等，根据你希望监控的数据类型。
  - 配置其他选项，如单位、值类型、更新间隔等。
- 配置触发器：
  - 在主机详情页的上方，点击"Triggers"（触发器）选项卡。
  - 点击页面右上角的"Create Trigger"（创建触发器）按钮。
  - 在"Name"字段中输入触发器的名称。
  - 在"Expression"字段中定义触发器的表达式，根据你的自定义监控项的值设置触发条件。
- 设置报警规则：
  - 在触发器配置页面中，设置触发器的报警规则，如报警优先级、通知方式等。
- 保存配置：完成监控项、触发器和报警规则的配置后，点击"Add"或"Save"保存配置。
- 验证配置：返回主机详情页，可以看到你刚刚创建的自定义监控项和触发器。在数据开始上传后，这些项将会显示相关的监控数据和报警状态。

### 配置自动注册
元素据位置及示例：
```
[root@centos82 zabbix_agentd.d]# pwd
/etc/zabbix/zabbix_agentd.d
[root@centos82 zabbix_agentd.d]# cat hostmetadata.conf
HostMetadata=|centos_linux|nginx|redis-single|
```
主页左侧导航栏
- 选择`配置`
- 然后选择`动作`
- 再选择`自动注册动作`
- 右上角点击`创建动作`，添加项目如下：
  - 名称：写入名称
  - 条件：添加条件，例如主机元数据包含nginx
  - 勾选或不勾选`已启用`
- 点击`添加`进行添加确认

## 待补充
