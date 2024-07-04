# DS8000-Copy Services Manager
&#8195;&#8195;IBM Copy Services Manager除了应用于IBM DS8000系列存储，还可以用于Spectrum Virtualize-based storage等。本内容主要学习及记录DS8000上应用。官方相关参考链接：
- [CSM6.2.12-Using Copy Services Manager on the DS8000](https://www.ibm.com/docs/en/csm/6.2.12?topic=6212-using-copy-services-manager-ds8000)
- [CSM6.2.12-Applying license files after installation or migration, or updating licenses](https://www.ibm.com/docs/en/csm/6.2.12?topic=icsm-applying-license-files-after-installation-migration-updating-licenses)

## 登录
### WEB GUI
使用WEB浏览器登录地址示例：
```
https://xxx.xxx.xxx.xxx:9559/CSM/
```
用户: `csmadmin`, 默认密码: `passw0rd`
### CSM CLI
Copy Services Manager command-line interface.
## Easy Tier Heat Map Transfer
&#8195;&#8195;IBM Copy Services Manager(CSM)中的Easy Tier Heat Map Transfer功能是用于在数据复制或迁移过程中传输Easy Tier热图信息的一项技术。Easy Tier是IBM存储系统中的一项自动存储分层技术，它可以根据访问频率动态调整数据在不同存储层（如 SSD、硬盘）之间的位置，以优化性能。      
&#8195;&#8195;Easy Tier Heat Map Transfer功能的主要作用是在使用复制服务（如 Metro Mirror、Global Mirror 等）进行数据复制或迁移时，将源存储系统中的热图信息同步到目标存储系统。热图信息包括哪些数据块的访问频率最高和最低，这对于保持目标存储系统的性能优化是非常重要的。    
&#8195;&#8195;当热图信息被传输到目标系统后，尽管数据布局可能发生了变化，目标存储系统也可以通过热图信息立即进行有效的存储分层调整，而不需要重新学习数据访问模式。

### 开关Easy Tier Heat Map Transfer优劣
开启Easy Tier Heat Map Transfer后效果如下：
- 性能优化：目标存储系统可以利用传输过来的热图信息，立即应用适当的存储分层策略，从而维持或提升性能。这对于需要快速恢复性能的灾难恢复场景特别关键。
- 减少学习时间：源系统的热图信息允许目标系统快速了解哪些数据是“热”的，从而节省重新学习数据访问模式所需的时间。
- 提高效率：数据复制或迁移完成后，目标系统上的工作负载性能将更加接近于源系统，从而减少性能调优的时间和手段。

关闭Easy Tier Heat Map Transfer后效果：
- 性能恢复延迟：目标存储系统需要通过自己的访问模式监测来重新学习哪些数据是“热”的。这可能会导致短时间内的性能不稳定或不优化。
- 额外开销减少：虽然传输热图信息对于目标系统性能有帮助，但它会增加额外的元数据传输开销。关闭这个功能可以减少这部分开销，尽管大多数情况下，这部分开销相对较小。
- 手动调优需求增加：在某些情况下，关闭热图传输后，可能需要管理员进行手动的性能调优，以达到最佳的存储分层效果。

### 设置Easy Tier Heat Map Transfer

&#8195;&#8195;该功能可以在创建或修改复制对话（session）时进行设置，一般在配置 Metro Mirror或 Global Mirror等复制技术时进行定义。

在 CSM GUI 中设置：
- 登录 CSM Web 界面
- 创建或编辑复制对话：
   - 选择相应的复制对话（session），如 Metro Mirror 或 Global Mirror
- 配置 Heat Map Transfer：
   - 在创建或编辑对话的过程中，会有选项 (如 Advanced Options) 供启用或禁用Easy Tier Heat Map Transfer

#### 在CLI中设置
假设要创建一个 Metro Mirror会话并启用Heat Map Transfer，可以使用类似如下的命令：
```sh
csmcli create session -name my_mirror_session -type mMirror -source my_source_system -target my_target_system -heatmaptransfer enable
```
要禁用 Heat Map Transfer，可以将命令中的 `-heatmaptransfer enable` 更改为 `-heatmaptransfer disable`。

以上内容主要来自AI。
## Easy Tier Heat Map Information
&#8195;&#8195;Easy Tier热图信息（Heat Map Information）是IBM Easy Tier技术中的一个关键概念。热图信息是指存储系统中关于数据访问频率的统计信息。通过这些统计信息，系统可以了解哪些数据块是“热”数据（即高频访问数据），哪些是“冷”数据（即低频访问数据）。具体来讲，热图信息包括：
- **数据块访问频率**：这是热图的核心部分，记录了每个数据块的读写频率。这些数据有助于确定哪些数据应该放在速度更快的存储层中
- **访问模式**：除了访问频率，热图信息还可能包括数据的访问模式（例如随机访问还是顺序访问），这有助于对数据块进行更细致的优化
- **历史趋势**：一些热图可能还包括访问频率的历史趋势，以帮助检测和适应数据访问行为的变化

### Easy Tier热图信息的作用
作用如下：
- **自动数据分层**：Easy Tier 使用热图信息自动调整数据在不同存储层之间的位置。高频访问的热数据会被移动到更快的存储层（如SSD），而低频访问的冷数据会被移动到较慢的存储层（如HDD）
- **提高性能**：通过这种自动化的数据分层，存储系统能够在不增加额外硬件成本的情况下显著提升性能。
- **优化资源利用**：热图信息可以最大化存储层资源的利用效率，使得高性能的存储资源能够被用在最需要的地方。
- **减少管理复杂性**：Easy Tier 的自动分层功能减少了管理员手动调整存储层之间数据的复杂性，降低了管理开销。

以上内容主要来自AI。
## 待补充