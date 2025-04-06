---
title: 'Docker Network'
disqus: hackmd
---

Docker Network
===
![downloads](https://img.shields.io/github/downloads/atom/atom/total.svg)
![build](https://img.shields.io/appveyor/ci/:user/:repo.svg)
![chat](https://img.shields.io/discord/:serverId.svg)

## Table of Contents

[TOC]


## Network Labs
> Docker有四種網路模式: none、bridge、host
### 準備工作
> Dockerfile(Dockerfile-ubuntu-network)
```gherkin=
FROM ubuntu:22.04

RUN apt-get update && \
    apt-get -y net-tools
```
> 以 Dockerfile 建立鏡像
```gherkin=
docker build -f Dockerfile-ubuntu-network -t ubuntu-network .
```
> 在 Host 上查看網路狀態
```gherkin=
sudo lsns -t net
```

### Lab1 - none(--network=none)
> 啟動一個名為ubuntu-network的容器後，並以 -it 登入容器的shell，--rm 代表登出後自動刪除
> --network=none 表示不設定網路
```gherkin=
docker container run -it --rm --network=none ubuntu-network
```
> 查看啟動中的 container
![截圖 2025-03-24 凌晨12.22.02](https://hackmd.io/_uploads/SJ4irha2ye.png)
> 登入 container 的 shell 
![截圖 2025-03-24 凌晨12.22.17](https://hackmd.io/_uploads/B10jS2a3ye.png)

> 皆顯示一樣的 container id

> 在Host 上 查看網路設定
 
```gherkin=
docker container inspect --format='{{ json .NetworkSettings}}'  35457f6c29b2 | jq
```
![截圖 2025-03-24 凌晨12.28.54](https://hackmd.io/_uploads/ryqBw2pnkg.png)
> 可觀察三個重點
> 1. 35457f6c29b2 為 Container ID，每次啟動可能會有所不同，請更換實際請動後的 Container ID
> 2. none 表示沒有設定網路模式
> 3. 沒有 IPAddress 

### Lab2 - Host(--network=host)
> --network=host 表示網路設定成 HOST 模式
```gherkin=
docker container run -it --rm --network=host ubuntu-network
```
> 查看Container的 ip 為 192.168.102.128(依照每台HOST的實際使用 ip，可能會有所不同 )
![image](https://hackmd.io/_uploads/B1Nu1dA2kx.png)
> 查看 Container 以 HOST 的網路模式啟動
![image](https://hackmd.io/_uploads/Hypre_R2Je.png)
> Container的 ip 與 HOST 主機的 ip 一樣
![image](https://hackmd.io/_uploads/BJDbZdRhJl.png)

### Lab3 - Bridge(--network=bridge)

> --network=bridge 表示網路設定成 BRIDGE 模式
```gherkin=
docker container run -it --rm --network=bridge ubuntu-network
```
![image](https://hackmd.io/_uploads/SkQoJ80h1e.png)


### Lab4 Container (--network=container:[container_id])
> 以 Lab3(Bridge)的網路模式啟動一個容器
![截圖 2025-04-06 下午3.29.33](https://hackmd.io/_uploads/Bycv6oyCJe.png)

> 查詢 container id 
> 再啟動一個 Container的網路模式
```gherkin=
docker container run -it --rm --network=container:c3bbf1df85f1 ubuntu-network
# c3bbf1df85f1 為Lab3 的 container id，請改成你電腦裡的 container id
```
![截圖 2025-04-06 下午3.28.56](https://hackmd.io/_uploads/ByUSps1R1l.png)

> 在Container裡執行下列指令
```gherkin=
ip addr
```
![截圖 2025-04-06 下午3.31.24](https://hackmd.io/_uploads/r1sRaskAJe.png)
> 在Host 裡執行下列指令
```gherkin=
docker ps -a
# or
docker container ls
# c3bbf1df85f1 是之前Lab啟動的容器
# 912511cdad8d 這個Lab啟動的容器
```
![截圖 2025-04-06 下午3.40.04](https://hackmd.io/_uploads/Bkz1xnJRkl.png)

> 前一個Lab啟動的容器網路設定內容 
![截圖 2025-04-06 下午3.42.33](https://hackmd.io/_uploads/BkI_ehyA1x.png)
> 這個Lab啟動的容器網路設定內容
![截圖 2025-04-06 下午3.42.54](https://hackmd.io/_uploads/ryTKl3yRkg.png)

> 在前一個Lab啟動的容器執行
```gherkin=
lsns
```
![截圖 2025-04-06 下午3.48.56](https://hackmd.io/_uploads/S1cgznJ0kl.png)
> 在現在啟動的容器執行
```gherkin=
lsns
```
![截圖 2025-04-06 下午3.48.03](https://hackmd.io/_uploads/HJc0bhJRJl.png)
> 兩個容器內的 net namespace 有一樣的 id，其他 namespace 的 id 不一樣
 
> 兩個容器皆可 ping 8.8.8.8，表示對外連線皆正常
![截圖 2025-04-06 下午3.53.52](https://hackmd.io/_uploads/SkaGXhJCyl.png)
![截圖 2025-04-06 下午3.53.21](https://hackmd.io/_uploads/B1lZX3JCJg.png)

> 前一個容器退出，意即刪除，第二個容器將無法連到外面網路
![截圖 2025-04-06 下午3.56.03](https://hackmd.io/_uploads/HJlimhJ0yg.png)

### Lab5 veth
```gherkin=
ip link list

sudo ip netns add ns0

sudo ip netns add ns1

ip netns list
```

```gherkin=
sudo ip link add veth0 type veth peer name veth1
```

```gherkin=
sudo ip link set veth0 netns ns0

sudo ip link set veth1 netns ns1
```

```gherkin=
sudo ip netns list
```

```gherkin=
sudo ip netns exec ns0 ip addr add 172.18.0.2/24 dev veth0

sudo ip netns exec ns0 ip link set veth0 up
```

```gherkin=
sudo ip netns exec ns0 ip addr
```
## Appendix and FAQ

:::info
**Find this document incomplete?** Leave a comment!
:::

###### tags: `Templates` `Documentation`
