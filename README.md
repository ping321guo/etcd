# Running etcd  v3.2.12 under Docker
> Transport security & client certificates in a cluster

> 参考链接

> https://coreos.com/etcd/docs/latest/v2/docker_guide.html

> https://coreos.com/etcd/docs/latest/op-guide/security.html

> https://quay.io/repository/coreos/etcd?tag=latest&tab=tags 

![](http://xxx.png)

## 部署

机房 | 	IP	 | Zone	 | 系统版本 |  域名 
:-: | :-: | :-: | :-: | :-: | 
xxg	| 10.xx.xx.227	 |  xxg  | 7.3  | etcd0.test.cn
xxg | 10.xx.xx.228 |  xxg |  7.3 |  etcd1.test.cn
xxg | 10.xx.xx.229	 | xxg |  7.3  |  etcd2.test.cn

## 生成证书

## 启动集群

```
export ETCDNODEIP0=10.xx.xx.227 
export ETCDNODEIP1=10.xx.xx.228
export ETCDNODEIP2=10.xx.xx.229

“引火”etcd
[root@227 ~]# docker run -d  -p 12379:2379 --name etcd-one quay.io/coreos/etcd:v3.2.12  etcd  --name etcd-one  --listen-client-urls="http://0.0.0.0:2379" --advertise-client-urls http://0.0.0.0:2379
8be872d59011bd69df1fe5cce8b422792c9761bb50bb297d06b049616a9930ac
[root@227 ~]# curl  -X PUT   http://10.xx.xx.227:12379/v2/keys/discovery/for-discovery-keyabc/_config/size -d value=3
{"action":"set","node":{"key":"/discovery/for-discovery-keyabc/_config/size","value":"3","modifiedIndex":4,"createdIndex":4}}

etcd0
[root@227 ~]# docker run -d --net=host  --restart always  -v /data2/etcd-data:/var/etcd --volume=/usr/share/ca-certificates/:/etc/etcd/ssl -p 4001:4001 -p 2380:2380 -p 2379:2379  --name etcd quay.io/coreos/etcd:v3.2.12  etcd  --name etcd0  --data-dir=/var/etcd  -advertise-client-urls https://${ETCDNODEIP0}:2379,https://${ETCDNODEIP0}:4001  -listen-client-urls https://0.0.0.0:2379,http://0.0.0.0:4001  -initial-advertise-peer-urls https://${ETCDNODEIP0}:2380  -listen-peer-urls https://0.0.0.0:2380  --cert-file /etc/etcd/ssl/etcd-server.pem  --key-file /etc/etcd/ssl/etcd-server-key.pem  --trusted-ca-file /etc/etcd/ssl/ca.pem   --client-cert-auth=true  --peer-cert-file /etc/etcd/ssl/etcd-server.pem   --peer-key-file /etc/etcd/ssl/etcd-server-key.pem  --peer-trusted-ca-file /etc/etcd/ssl/ca.pem   --peer-client-cert-auth=true --discovery http://10.xx.xx.227:12379/v2/keys/discovery/for-discovery-keyabc
6a1fe7ecd6b3d75aabb78d50e136b80b77dcb0a3054e6562f31adf6d2f3ea46b


etcd1
[root@228 ~]# docker run -d --net=host  --restart always  -v /data2/etcd-data:/var/etcd --volume=/usr/share/ca-certificates/:/etc/etcd/ssl -p 4001:4001 -p 2380:2380 -p 2379:2379  --name etcd quay.io/coreos/etcd:v3.2.12  etcd  --name etcd1  --data-dir=/var/etcd  -advertise-client-urls https://${ETCDNODEIP0}:2379,https://${ETCDNODEIP0}:4001  -listen-client-urls https://0.0.0.0:2379,http://0.0.0.0:4001  -initial-advertise-peer-urls https://${ETCDNODEIP0}:2380  -listen-peer-urls https://0.0.0.0:2380  --cert-file /etc/etcd/ssl/etcd-server.pem  --key-file /etc/etcd/ssl/etcd-server-key.pem  --trusted-ca-file /etc/etcd/ssl/ca.pem   --client-cert-auth=true  --peer-cert-file /etc/etcd/ssl/etcd-server.pem   --peer-key-file /etc/etcd/ssl/etcd-server-key.pem  --peer-trusted-ca-file /etc/etcd/ssl/ca.pem   --peer-client-cert-auth=true  --discovery  http://10.xx.xx.227:12379/v2/keys/discovery/for-discovery-keyabc
ff0762c2a3da243af30e7c05fa583f137ab9216e68178b3c730e135f3c67f77f

etcd2
[root@229 ~]# docker run -d --net=host  --restart always  -v /data2/etcd-data:/var/etcd --volume=/usr/share/ca-certificates/:/etc/etcd/ssl -p 4001:4001 -p 2380:2380 -p 2379:2379  --name etcd quay.io/coreos/etcd:v3.2.12  etcd  --name etcd2  --data-dir=/var/etcd  -advertise-client-urls https://${ETCDNODEIP2}:2379,https://${ETCDNODEIP2}:4001  -listen-client-urls https://0.0.0.0:2379,http://0.0.0.0:4001  -initial-advertise-peer-urls https://${ETCDNODEIP2}:2380  -listen-peer-urls https://0.0.0.0:2380  --cert-file /etc/etcd/ssl/etcd-server.pem  --key-file /etc/etcd/ssl/etcd-server-key.pem  --trusted-ca-file /etc/etcd/ssl/ca.pem   --client-cert-auth=true  --peer-cert-file /etc/etcd/ssl/etcd-server.pem   --peer-key-file /etc/etcd/ssl/etcd-server-key.pem  --peer-trusted-ca-file /etc/etcd/ssl/ca.pem   --peer-client-cert-auth=true   --discovery   http://10.xx.xx.227:12379/v2/keys/discovery/for-discovery-keyabc
20231dec63fabf3676bc086d9551975d22f7653c8410921875d549dbca0ba917
```

##配置解析
listen-peer-urls
用于节点与节点之间数据交换, 因此需要监听在其他节点可以访问的IP地址上
默认端口为: 2380 & 7001 (7001不推荐使用, 已基本废弃, 主要用于兼容老服务)
listen-client-urls
用户客户机访问etcd数据, 一般监听在本地, 如果需要集中管理, 可以监听在管理服务器可以访问的IP地址上
默认端口为: 2379 & 4001 (4001不推荐使用, 已基本废弃, 主要用于兼容老服务)
initial-advertise-peer-urls
该参数表示节点监听其他节点同步信号的地址
默认端口为: 2380 & 7001 (7001不推荐使用, 已基本废弃, 主要用于兼容老服务)
advertise-client-urls
在加入proxy节点后, 会使用该广播地址, 因此需要监听在一个proxy节点可以访问的IP地址上
默认端口为: 2379 & 4001 (7001不推荐使用, 已基本废弃, 主要用于兼容老服务)

