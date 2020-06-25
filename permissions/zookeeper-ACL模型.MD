##  ACL
ACL全称Access Control List，在ACL中，包含用户(user)、资源(Resource)、资源操作(Operation)三个关键要素。通过将资源以及资源操作授权给用户而使用户获取对资源进行操作的权限，如下所示：  

用户 | 功能 | 资源
---|---|---
root | 重启，删除 | a程序
admin | 启动 | a程序


### 权限特性
```
1. ZooKeeper 的权限控制是基于每个 znode 节点的，需要对每个节点设置权限      
2. 每个 znode 支持设置多种权限控制方案和多个权限   
3. 子节点不会继承父节点的权限，客户端无权访问某节点，但可能可以访问它的子节点    
```
### 权限类型

权限 | ACL简写 | 描述
---|---|---
CREATE | c | 可以创建子节点
DELETE | d | 可以删除子节点（仅下一级节点）
READ | r | 可以读取节点数据及显示子节点列表
WRITE | w | 可以设置节点数据
ADMIN | a | 可以设置节点访问控制列表权限

### 访问控制列表方案（ACL Schemes）
ZooKeeper 内置了一些权限控制方案，可以用以下方案为每个节点设置权限：

方案 | 描述 
---|---
world | 只有一个用户：anyone，代表所有人（默认） 
ip | 使用 IP 地址认证 
auth | 使用已添加认证的用户认证
digest |使用 “用户名:密码” 方式认证

### 权限相关命令
命令 | 	使用方式 | 	描述
---|---|---
getAcl | 	getAcl	| 读取 ACL 权限
setAcl | 	setAcl | 	设置 ACL 权限
addauth	| addauth| 	添加认证用户

### World 方案
设置方式
```
setAcl <path> world:anyone:<acl>

setAcl /node1 world:anyone:cdrwa

getAcl /node1

'world,'anyone
: cdrwa

```
### IP 方案
设置方式
```
setAcl <path> ip:<ip>:<acl>

```
<ip>：可以是具体 IP 也可以是 IP/bit 格式，即 IP 转换为二进制，匹配前 bit 位，如 192.168.0.0/16 匹配 192.168.*.*

### Auth 方案
设置方式
```
addauth digest <user>:<password> # 添加认证用户
setAcl <path> auth:<user>:<acl>
```

### Digest 方案
设置方式
```
setAcl <path> digest:<user>:<password>:<acl>
```
这里的密码是经过 SHA1 及 BASE64 处理的密文，在 SHELL 中可以通过以下命令计算：

```
echo -n <user>:<password> | openssl dgst -binary -sha1 | openssl base64
```
先来算一个密文密码：

echo -n yoonper:123456 | openssl dgst -binary -sha1 | openssl base64
UvJWhBril5yzpEiA2eV7bwwhfLs=