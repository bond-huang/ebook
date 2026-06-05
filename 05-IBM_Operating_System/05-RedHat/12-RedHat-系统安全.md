# RedHat-系统安全

## iptables
命令来自AI，部分经过验证，供参考。
### 基础命令
查看所有iptables规则：
```bash
iptables -L -n
iptables -L -n -v
```
命令说明：
- **-L**：列出所有规则
- **-n**：不解析IP和端口名称
- **-v**：显示数据包计数、网卡、端口等详细信息，能看到哪条规则被匹配过多少次

只看INPUT/OUTPUT/FORWARD链：
```bash
iptables -L INPUT -n -v   # 只看入站规则
iptables -L OUTPUT -n -v  # 只看出站规则
iptables -L FORWARD -n -v # 只看转发规则
```

查看带行号（方便删除规则），会给每条规则标上 **1、2、3...**，删除时直接用编号：
```bash
iptables -L INPUT --line-numbers -n
```
查看NAT表规则（端口映射必备）：
```bash
iptables -t nat -L -n -v
```
命令说明：
- **-t nat**：查看 NAT 表（端口转发、外网映射都在这里）

查看RAW/MANGLE表（高级）：
```bash
iptables -t raw -L -n
iptables -t mangle -L -n
```

清空规则（慎用）：
```bash
iptables -F    # 清空所有规则
iptables -t nat -F  # 清空 NAT 规则
```
### iptables策略
放行单个端口（例如 8080/tcp）：
```bash
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
```
放行一段端口：
```bash
iptables -A INPUT -p tcp --dport 30000:40000 -j ACCEPT
```
放行UDP端口：
```bash
iptables -A INPUT -p udp --dport 53 -j ACCEPT
```
保存规则（重启不失效）：
CentOS6/老系统：
```bash
service iptables save
```
CentOS7+/Ubuntu：
```bash
iptables-save > /etc/iptables/rules.v4
```
放行后进行验证：
```sh
# 查看本机监听端口
ss -tuln

# 查看 iptables 规则
iptables -L -n
```
放行一段 IP（例如 192.168.1.0/24）访问 3306：
```bash
iptables -A INPUT -p tcp --dport 3306 -s 192.168.1.0/24 -j ACCEPT
```
网段写法说明：
- `192.168.1.0/24` = 192.168.1.1 ~ 192.168.1.255
- `10.0.0.0/8` = 超大段
- `172.16.0.0/16` = 中段

放行一段 IP范围访问 3306：
```shell
iptables -A INPUT -p tcp --dport 3306 -m iprange --src-range 192.168.1.10-192.168.1.15 -j ACCEPT
```
在规则链的最后面，添加一条新规则：
```sh
iptables -A INPUT ...
```
说明：
  - 新加的规则**排在最后**
  - 如果前面有 `REJECT` / `DROP`，这条可能**不生效**

在规则链最前面插入一条规则（`-I` = Insert（插入））：
```sh
iptables -I INPUT ...
```
说明：
  - 优先匹配，**最容易生效**
  - 适合放行重要端口、信任IP

查看当前 iptables 规则（ `-L` = List（列出））：
```sh
iptables -L
iptables -L -n   # 常用，显示数字IP和端口，不解析域名
```
总结
- `-A`：**追加**到最后
- `-I`：**插入**到最前
- `-L`：**列出**所有规则

示例：
```sh
# 插入到最前面，优先允许 3306
iptables -I INPUT -p tcp --dport 3306 -j ACCEPT

# 追加到最后，一般用于兜底
iptables -A INPUT -j DROP

# 查看规则
iptables -L -n
```
删除规则（`-D` = Delete），示例：
```sh
iptables -D INPUT -p tcp --dport 3306 -j ACCEPT
```
按行号删，先查看，再删，示例：
```sh
iptables -L INPUT --line-numbers -n
```
会显示：
```sh
1  tcp  --  0.0.0.0/0  0.0.0.0/0  tcp dpt:3306 ACCEPT
2  ...
```
删掉第1条：
```sh
iptables -D INPUT 1
```
### iptables使用
#### CentOS/RHEL7/8/9
先停firewalld（必须）：
```sh
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo systemctl mask firewalld
```
安装 iptables-services（提供save功能）：
```sh
sudo yum install -y iptables-services
# 或 dnf
sudo dnf install -y iptables-services
```
启用并启动iptables：
```sh
sudo systemctl enable iptables
sudo systemctl start iptables
```
保存：
```sh
sudo service iptables save
# 或用 systemctl 方式
sudo systemctl save iptables
```
手动保存：
```
sudo iptables-save > /etc/sysconfig/iptables
```
重启后用 `iptables-restore < /etc/sysconfig/iptables` 恢复。

#### Ubuntu/Debian系列
Ubuntu不用 `service iptables save`，用`netfilter-persistent`。安装持久化工具
```sh
sudo apt update
sudo apt install -y netfilter-persistent iptables-persistent
```
保存规则:
```sh
sudo netfilter-persistent save
```
手动保存：
```sh
sudo iptables-save > /etc/iptables/rules.v4
sudo ip6tables-save > /etc/iptables/rules.v6
```

## 待补充