---
{
    "title": "IPV4_TO_IPV6",
    "language": "zh-CN"
}
---

## 描述
接受一个类型为 IPv4 的地址，返回相应 IPv6 的形式。

## 语法
```sql
IPV4_TO_IPV6(<ipv4>)
```

## 参数
| Parameter | Description                                      |
|-----------|--------------------------------------------------|
| `<ipv4>`      | ipv4 类型的地址 |

## 返回值
返回转化后的 ipv6 地址

## 举例
```sql
select ipv6_num_to_string(ipv4_to_ipv6(to_ipv4('192.168.0.1')));
```
```text
+----------------------------------------------------------+
| ipv6_num_to_string(ipv4_to_ipv6(to_ipv4('192.168.0.1'))) |
+----------------------------------------------------------+
| ::ffff:192.168.0.1                                       |
+----------------------------------------------------------+
```
