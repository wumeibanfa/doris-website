---
{
"title": "集群环境要求",
"language": "zh-CN"
}
---

## 软件版本要求

| 软件             | 版本要求      |
|----------------|-----------|
| Docker         | \>= 1.20  |
| Kubernetes     | \>= 1.19  |
| Doris          | \>= 2.0.0 |
| Helm（可选）       | \>= 3.7   |

## 操作系统要求

### 防火墙配置

在 Kubernetes 上部署 Doris 集群，建议关闭防火墙配置：

```shell
systemctl stop firewalld
systemctl disable firewalld
```

如果无法关闭防火墙服务，可以根据规划，打开 FE 与 BE 端口：
:::tip 提示
如果无法关闭防火墙，需要根据 Kubernetes 映射规则打开 Doris 相应端口的防火墙。具体端口可以参考 [Doris 集群端口规划](../standard-deployment.md#端口规划)。
:::


### 禁用和关闭 swap

在部署 Doris 时，建议关闭 swap 分区。

通过以下命令可以永久关闭 swap 分区。

```shell
echo "vm.swappiness = 0">> /etc/sysctl.conf
swapoff -a && swapon -a
sysctl -p
```

### 设置系统最大打开文件句柄数

```shell
vi /etc/security/limits.conf 
* soft nofile 65536
* hard nofile 65536
```

### 修改虚拟内存区域数量

修改虚拟内存区域至少 2000000

```shell
sysctl -w vm.max_map_count=2000000
```

### 关闭透明大页

在部署 Doris 时，建议关闭透明大页。

```shell
echo madvise > /sys/kernel/mm/transparent_hugepage/enabled
echo madvise > /sys/kernel/mm/transparent_hugepage/defrag
```
