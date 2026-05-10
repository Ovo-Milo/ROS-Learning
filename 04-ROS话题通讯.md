# 第4次课：ROS 话题通讯

[← 返回目录](./README.md)

---

## Topic 通信原理

### 特点

- **异步单向**通信：Publisher → Topic → Subscriber
- Publisher 和 Subscriber 都需向 Master 注册
- Master 协助建立点对点连接
- 一个 Topic 可有多个 Publisher 和多个 Subscriber

### 通信流程

```
1. Publisher 和 Subscriber 向 Master 注册
2. Publisher 发布 Topic
3. Subscriber 在 Master 指挥下订阅 Topic
4. 建立 Pub-Sub 通信连接
```

> 💡 **异步**：Publisher 发完消息立即返回，不关心谁接收；Subscriber 只管处理消息，不关心谁发送

### 回调机制

Subscriber 通过 **回调函数（Callback）** 处理消息：预先定义好处理函数，有消息到达时自动触发。

---

## 编写 Publisher

```cpp
#include "ros/ros.h"
#include "std_msgs/Int32.h"

int main(int argc, char **argv)
{
    ros::init(argc, argv, "third_pkg");
    ros::NodeHandle n;

    // 创建发布者，话题名 "third_pkg_topic"，队列大小 1000
    ros::Publisher chatter_pub = n.advertise<std_msgs::Int32>("third_pkg_topic", 1000);
    ros::Rate loop_rate(2);  // 2Hz，500ms 发一次

    int count = 0;
    while (ros::ok())
    {
        std_msgs::Int32 msg;
        msg.data = count;
        ROS_INFO("%d", msg.data);
        chatter_pub.publish(msg);
        ros::spinOnce();
        loop_rate.sleep();
        count = (++count) % 5;  // 0~4 循环
    }
    return 0;
}
```

---

## 编写 Subscriber

```cpp
#include "ros/ros.h"
#include "std_msgs/Int32.h"

// 回调函数：收到消息时自动调用
void chatterCallback(const std_msgs::Int32::ConstPtr& msg)
{
    ROS_INFO("I heard: [%d]", msg->data);
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "subscribe_node");
    ros::NodeHandle n;

    // 创建订阅者，订阅 "third_pkg_topic"
    ros::Subscriber sub = n.subscribe("third_pkg_topic", 1000, chatterCallback);
    ros::spin();  // 进入循环，等待消息
    return 0;
}
```

---

## 自定义消息类型

### 1. 创建 msg 文件

`msg/myTestMsg.msg`：

```
string name
int32 age
bool handsome
float64 salary
```

### 2. 修改 CMakeLists.txt

```cmake
find_package(catkin REQUIRED COMPONENTS message_generation ...)

add_message_files(FILES myTestMsg.msg)
generate_messages(DEPENDENCIES std_msgs)
```

### 3. 修改 package.xml

```xml
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
```

### 4. 编译

```bash
cd ~/ros_workspace
catkin_make
# 自动生成头文件：devel/include/third_pkg/myTestMsg.h
```

### 5. 使用自定义消息

```cpp
#include "third_pkg/myTestMsg.h"

// 发布端
third_pkg::myTestMsg msg;
msg.name = "corvin";
msg.age = 20;
msg.handsome = true;
msg.salary = 123.45;
chatter_pub.publish(msg);

// 接收端
void callback(const third_pkg::myTestMsg::ConstPtr& msg)
{
    ROS_INFO("name:%s, age:%d", msg->name.c_str(), msg->age);
}
```

### ROS 基本数据类型

```
int8, int16, int32, int64 (plus uint*)
float32, float64
string
time, duration
other msg files
variable-length array[] and fixed-length array[C]
```

---

## 调试命令

### rqt_graph — 节点通信可视化

```bash
rqt_graph
```

- 椭圆 = 节点，长方形 = 话题，箭头 = 通信方向

### rostopic 命令

| 命令 | 作用 |
|------|------|
| `rostopic list` | 列出所有话题 |
| `rostopic info /topic` | 查看话题的发布者、订阅者、类型 |
| `rostopic echo /topic` | 打印话题内容 |
| `rostopic type /topic` | 查看话题类型 |
| `rostopic pub -r 10 /topic type "data: 100"` | 向话题发布消息 |

### rosmsg 命令

| 命令 | 作用 |
|------|------|
| `rosmsg list` | 列出所有消息类型 |
| `rosmsg show <type>` | 展示消息结构 |
| `rosmsg list \| wc -l` | 统计消息数量 |

---

[← 返回目录](./README.md)
