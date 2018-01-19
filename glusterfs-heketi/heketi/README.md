# heketi
Heketi提供了rest风格的api接口去管理GlusterFS中volume的生命周期。通过Heketi，OpenStack Manila, Kubernetes, OpenShift等云服务可以动态的申请GlusterFS的volume。而且，Heketi还可以支持同时管理多个GlusterFS集群。Heketi的工作流程如下：
- 接收申请volume的请求
- 向集群申请存储资源
- 磁盘分区，挂载磁盘，创建bricks
- 等待所有bricks创建成功之后，创建volume

## heketi 安装
### (1)yum 安装
```
yum install heketi heketi-client
```
### (2)环境配置
heketi要求ssh的免密登陆，因此需要设置
- 在heketi server上生成签名，并发送给glusterfs所在的机器
```
sh-keygen -f /etc/heketi/heketi_key
chown heketi:heketi /etc/heketi/heketi_key*
ssh-copy-id -i /etc/heketi/heketi_key.pub root@server1.com
ssh-copy-id -i /etc/heketi/heketi_key.pub root@server2.com
```
- 注释掉/etc/sudoers中的requiretty
### (3)heketi server配置
配置文件在/etc/heketi/heketi.json，[`heketi.json示例`](heketi.json)

### (4)启动
```
systemctl enable heketi
systemctl start heketi
systemctl status heketi
```
### (5)验证
```
curl http://heketi-server:prot/hello
```
### (6)topolog.json
topolog.json文件描述了GlusterFS集群情况，[`topolog.json示例`](topolog.json)
设置heketi-server地址到环境变量里
```
export HEKETI_CLI_SERVER=http://heketi-server:port
```
上传topolog中的配置到heketi server
```
heketi-cli topology load --json=topology.json
```
### (7)创建volume
```
heketi-cli volume create -size=100  -replica=3

```
### (8)查看volume
```
heketi-cli volume list
```
### (9)删除volume
```
heketi-cli volume delete <volume-id>
```
### (10)日志
```
journalctl -u heketi
or
vi /var/log/messages
```
