# 第3次课：ROS 节点编写

[← 返回目录](./README.md)

---

## Node 与 Master

### 概念

- **Node（节点）**：最小进程单元，一个可执行文件运行后就是一个节点
- **Master（节点管理器）**：管理中心，负责节点注册和通信"牵线"

> 💡 设计理念：分布式架构，每个节点负责单一功能（底盘控制、摄像头、路径规划等），降低崩溃风险

### 启动流程

```
1. 启动 roscore（Master）
2. 各 Node 到 Master 注册
3. Master 协助 Node 之间建立通信
```

### roscpp 编程接口

ROS 支持多种语言的 Client Library：

| 库 | 语言 | 状态 |
|----|------|------|
| roscpp | C++ | ✅ 主流 |
| rospy | Python | ✅ 主流 |
| roslisp | Lisp | 测试版 |
| rosjava | Java | 测试版 |

---

## 编写第一个 Node

### 源码 `test.cpp`

```cpp
#include "ros/ros.h"
#include "std_msgs/String.h"
#include <sstream>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "talker");                    // 初始化节点，名称为 "talker"
    ros::NodeHandle n;                                   // 创建节点句柄
    ros::Publisher chatter_pub = n.advertise<std_msgs::String>("chatter", 1000);
    ros::Rate loop_rate(10);                             // 设置频率 10Hz

    int count = 0;
    while (ros::ok())
    {
        std_msgs::String msg;
        std::stringstream ss;
        ss << "hello world " << count;
        msg.data = ss.str();
        ROS_INFO("%s", msg.data.c_str());               // 打印日志
        chatter_pub.publish(msg);                        // 发布消息
        ros::spinOnce();                                 // 处理回调
        loop_rate.sleep();                               // 按频率休眠
        ++count;
    }
    return 0;
}
```

### 修改 CMakeLists.txt

```cmake
# 将源码编译为可执行文件
add_executable(${PROJECT_NAME}_node src/test.cpp)

# 链接 catkin 库
target_link_libraries(${PROJECT_NAME}_node ${catkin_LIBRARIES})
```

### 编译与运行

```bash
cd ~/ros_workspace
catkin_make

# 终端1：启动 Master
roscore

# 终端2：运行节点
rosrun test_pkg test_pkg_node
```

---

## 头文件引用

### 引用当前包的头文件

1. 在 `include/<包名>/` 下创建头文件
2. CMakeLists.txt 添加包含路径：

```cmake
include_directories(include)
```

### 引用其他包的头文件

1. 创建包时依赖目标包
2. 目标包 CMakeLists.txt 中用 `catkin_package()` 导出 include 路径
3. 源码中 `#include "目标包名/头文件名.h"`

---

## Launch 文件

用于批量启动多个节点，格式为 XML。

### 基本结构

```xml
<launch>
    <!-- 启动节点 -->
    <node pkg="包名" type="可执行文件名" name="节点名" output="screen"/>

    <!-- 包含其他 launch 文件 -->
    <include file="$(find 包名)/launch/其他launch文件.launch"/>
</launch>
```

### 常用属性

| 属性 | 作用 |
|------|------|
| `pkg` | 功能包名称 |
| `type` | 可执行文件名称 |
| `name` | 节点名称（覆盖 ros::init 的名称） |
| `output` | `"screen"` 输出到终端 |
| `respawn` | `true` 节点退出后自动重启 |
| `required` | `true` 节点退出后关闭整个 launch |

### 运行

```bash
roslaunch robot_bringup startup.launch
```

> 💡 `roslaunch` 会自动启动 roscore，不需要单独运行

---

## rosnode 命令

| 命令 | 作用 |
|------|------|
| `rosnode list` | 列出运行中的节点 |
| `rosnode info /node` | 查看节点详细信息 |
| `rosnode ping /node` | 测试节点连通性 |
| `rosnode kill /node` | 终止节点 |

---

[← 返回目录](./README.md)
