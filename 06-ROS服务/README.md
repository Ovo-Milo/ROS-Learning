# 第6次课：ROS 服务

[← 返回目录](../README.md)

---

## Service 通信原理

### 特点

- **同步双向**：Client 发送 request → Server 处理 → 返回 reply
- Client 等待时处于**阻塞**状态
- 适合非周期性的请求-应答场景

### 通信流程

```
Node A (Client)                          Node B (Server)
    |                                        |
    |--- request ---> /Service --->          |
    |                  (阻塞等待)             |
    |                    处理请求             |
    |<-- reply ----- /Service <---           |
    |                                        |
```

### Topic vs Service 对比

| 特性 | Topic | Service |
|------|-------|---------|
| 通信方式 | 异步单向 | 同步双向 |
| 模式 | 发布/订阅 | 请求/应答 |
| 适用场景 | 周期性数据传输 | 按需查询/控制 |
| 资源占用 | 持续传输 | 按需调用 |
| 类比 | 广播 | 打电话 |

---

## 自定义服务类型

### 创建 srv 文件

`srv/myTestSrv.srv`：

```
int32 index
---
int32 result
```

> `---` 上方为请求（Request）部分，下方为响应（Response）部分

### 修改 CMakeLists.txt

```cmake
add_service_files(FILES myTestSrv.srv)
generate_messages(DEPENDENCIES std_msgs)
```

### 编译生成头文件

```bash
cd ~/ros_workspace
catkin_make
# 生成：devel/include/third_pkg/myTestSrv.h
```

---

## 编写 Server

```cpp
#include "third_pkg/myTestSrv.h"

static int battery = 100;

// 服务回调函数
bool checkBattery(third_pkg::myTestSrv::Request &req,
                  third_pkg::myTestSrv::Response &res)
{
    if (1 == req.index)
    {
        ROS_WARN("service sending response:[%d]", battery);
        res.result = battery;
    }
    return true;
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "third_pkg");
    ros::NodeHandle n;

    // 注册服务，名称为 "batteryStatus"
    ros::ServiceServer service = n.advertiseService("batteryStatus", checkBattery);

    ros::spin();
    return 0;
}
```

---

## 编写 Client

```cpp
#include "third_pkg/myTestSrv.h"

int main(int argc, char **argv)
{
    ros::init(argc, argv, "subscribe_node");
    ros::NodeHandle n;

    // 创建客户端
    ros::ServiceClient client = n.serviceClient<third_pkg::myTestSrv>("batteryStatus");

    third_pkg::myTestSrv mySrv;
    mySrv.request.index = 1;  // 查询电池编号 1

    ros::Rate loop_rate(1);
    while (ros::ok())
    {
        if (client.call(mySrv))  // 调用服务
        {
            ROS_WARN("Client Get Battery:%d", mySrv.response.result);
        }
        else
        {
            ROS_ERROR("Failed to call batteryStatus service");
            return 1;
        }
        ros::spinOnce();
        loop_rate.sleep();
    }
    return 0;
}
```

---

## 要点总结

| 概念 | 说明 |
|------|------|
| `advertiseService()` | 注册服务端，指定回调函数 |
| `serviceClient()` | 创建客户端 |
| `client.call()` | 调用服务（阻塞等待） |
| 回调函数签名 | `bool func(Request &req, Response &res)` |
| Request 成员 | 对应 srv 文件中 `---` 之前的字段 |
| Response 成员 | 对应 srv 文件中 `---` 之后的字段 |

---

[← 返回目录](../README.md)
