# 🤖 ROS 机器人操作系统学习笔记

> 基于 ROS Melodic + Ubuntu 18.04 的机器人开发课程完整笔记

---

## 📋 课程目录

| 课次 | 主题 | 核心内容 |
|------|------|----------|
| 第1次 | 环境配置 | VMware + Ubuntu 18.04 虚拟机安装 |
| 第2次 | ROS 工程认知 | 工作空间、功能包、catkin 编译系统 |
| 第3次 | ROS 节点编写 | Node、Master、roscpp、launch 文件 |
| 第4次 | ROS 话题通讯 | topic 发布/订阅、自定义消息类型 |
| 第5次 | 仿真机器人及环境 | Gazebo 仿真、Rviz 可视化 |
| 第6次 | ROS 服务 | service 通信机制、请求-应答模型 |
| 第6次(实验2) | 机器人移动控制 | 话题控制仿真机器人运动 |
| 第8次(1) | 语音采集 | 语音录制服务调用 |
| 第8次(2) | 语音听写 | 语音转文字服务 |
| 第8次(3) | 语义理解 | AIUI 语义分析与意图识别 |
| 第9次 | 地图构建 | SLAM + gmapping 算法 |
| 第10次 | 自主导航 | Navigation Stack、move_base、多点巡航 |
| 第11次 | 语音导航 | 语音 + 导航融合 |
| 第12次 | 人脸识别 | face_recognition + ROS 服务封装 |
| 补充 | 机器人二次定位 | AR 码跟踪 + 自主充电 |

---

## 第1次课：环境配置

### 安装步骤

1. **安装 VMware 17.0**：一路下一步，使用激活密钥完成注册
2. **导入虚拟机**：解压 `ubuntu元宝` 文件夹，打开 `Ubuntu 64 位.vmx`
3. **登录系统**：用户名 `bobac3`，密码 `root`

> 💡 遇到 "我已复制该虚拟机" 选择"我已复制该虚拟机"即可

---

## 第2次课：ROS 工程认知

### 核心概念

#### 工作空间（Workspace）

```
ros_workspace/
├── build/    # 编译中间文件
├── devel/    # 编译生成的可执行代码、库文件
└── src/      # 功能包目录
```

**创建与编译：**

```bash
mkdir -p ~/ros_workspace/src
cd ~/ros_workspace/
catkin_make
source ~/ros_workspace/devel/setup.bash
# 永久生效
echo "source ~/ros_workspace/devel/setup.bash" >> ~/.bashrc
```

#### 功能包（Package）

```
package/
├── CMakeLists.txt    # 编译规则（必须）
├── package.xml       # 描述信息（必须）
├── src/              # 源代码
├── include/          # C++ 头文件
├── scripts/          # 可执行脚本
├── msg/              # 自定义消息
├── srv/              # 自定义服务
├── launch/           # launch 文件
├── models/           # 3D 模型
└── urdf/             # 机器人模型描述
```

**创建功能包：**

```bash
cd ~/ros_workspace/src
catkin_create_pkg <包名> <依赖1> <依赖2> ...
# 示例
catkin_create_pkg test_pkg roscpp rospy std_msgs
```

### 常用命令

| 命令 | 作用 |
|------|------|
| `rospack find [pkg]` | 定位功能包 |
| `rospack depends [pkg]` | 显示依赖 |
| `roscd [pkg]` | cd 到功能包路径 |
| `rosls [pkg]` | 列出功能包文件 |
| `rosdep install --from-paths src --ignore-src -y` | 安装所有依赖 |

---

## 第3次课：ROS 节点编写

### Node 与 Master

- **Node（节点）**：最小进程单元，一个可执行文件运行后就是一个节点
- **Master（节点管理器）**：管理中心，负责节点注册和通信"牵线"

> 💡 设计理念：分布式架构，每个节点负责单一功能（底盘控制、摄像头、路径规划等），降低崩溃风险

### 编写第一个 Node

```cpp
#include "ros/ros.h"
#include "std_msgs/String.h"
#include <sstream>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "talker");       // 初始化节点
    ros::NodeHandle n;                      // 创建句柄
    ros::Publisher chatter_pub = n.advertise<std_msgs::String>("chatter", 1000);
    ros::Rate loop_rate(10);                // 10Hz

    while (ros::ok())
    {
        std_msgs::String msg;
        std::stringstream ss;
        ss << "hello world " << count;
        msg.data = ss.str();
        ROS_INFO("%s", msg.data.c_str());
        chatter_pub.publish(msg);
        ros::spinOnce();
        loop_rate.sleep();
        ++count;
    }
    return 0;
}
```

### CMakeLists.txt 编译配置

```cmake
add_executable(${PROJECT_NAME}_node src/test.cpp)
target_link_libraries(${PROJECT_NAME}_node ${catkin_LIBRARIES})
```

### 运行节点

```bash
roscore                                    # 先启动 Master
rosrun test_pkg test_pkg_node              # 运行节点
```

### 头文件引用

**引用当前包头文件：**
- 头文件放在 `include/<包名>/` 下
- CMakeLists.txt 添加：`include_directories(include)`

**引用其他包头文件：**
- 创建包时依赖目标包
- 目标包的 CMakeLists.txt 中用 `catkin_package()` 导出 include 路径

### Launch 文件

```xml
<launch>
    <!-- 启动节点 -->
    <node pkg="test_pkg" type="test_pkg_node" name="test_pkg_node" output="screen"/>
    <!-- 包含其他 launch -->
    <include file="$(find third_pkg)/launch/third_pkg.launch"/>
</launch>
```

**运行：** `roslaunch robot_bringup startup.launch`

---

## 第4次课：ROS 话题通讯

### Topic 通信原理

- **异步单向**通信：Publisher → Topic → Subscriber
- Publisher 和 Subscriber 都需向 Master 注册
- Master 协助建立点对点连接
- 一个 Topic 可有多个 Publisher 和 Subscriber

```
Publisher ──publish──> [Topic] ──callback──> Subscriber
```

### 编写 Publisher

```cpp
#include "ros/ros.h"
#include "std_msgs/Int32.h"

int main(int argc, char **argv)
{
    ros::init(argc, argv, "third_pkg");
    ros::NodeHandle n;
    ros::Publisher chatter_pub = n.advertise<std_msgs::Int32>("third_pkg_topic", 1000);
    ros::Rate loop_rate(2);  // 2Hz

    while (ros::ok())
    {
        std_msgs::Int32 msg;
        msg.data = count;
        chatter_pub.publish(msg);
        ros::spinOnce();
        loop_rate.sleep();
        count = (++count) % 5;
    }
    return 0;
}
```

### 编写 Subscriber

```cpp
#include "ros/ros.h"
#include "std_msgs/Int32.h"

void chatterCallback(const std_msgs::Int32::ConstPtr& msg)
{
    ROS_INFO("I heard: [%d]", msg->data);
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "subscribe_node");
    ros::NodeHandle n;
    ros::Subscriber sub = n.subscribe("third_pkg_topic", 1000, chatterCallback);
    ros::spin();
    return 0;
}
```

### 自定义消息类型

**1. 创建 msg 文件** `msg/myTestMsg.msg`：
```
string name
int32 age
bool handsome
float64 salary
```

**2. 修改 CMakeLists.txt：**
```cmake
find_package(catkin REQUIRED COMPONENTS message_generation ...)
add_message_files(FILES myTestMsg.msg)
generate_messages(DEPENDENCIES std_msgs)
```

**3. 修改 package.xml：** 添加 `message_generation` 依赖

**4. 使用自定义消息：**
```cpp
#include "third_pkg/myTestMsg.h"

third_pkg::myTestMsg msg;
msg.name = "corvin";
msg.age = 20;
```

### 常用调试命令

| 命令 | 作用 |
|------|------|
| `rqt_graph` | 可视化节点通信图 |
| `rostopic list` | 列出所有话题 |
| `rostopic info /topic` | 查看话题信息 |
| `rostopic echo /topic` | 打印话题内容 |
| `rostopic pub -r 10 /topic type "data"` | 发布话题 |
| `rostopic type /topic` | 查看话题类型 |
| `rosmsg list` | 列出所有消息类型 |
| `rosmsg show type` | 展示消息结构 |

---

## 第5次课：仿真机器人及环境

### Gazebo 仿真平台

- 开源机器人仿真平台，与 ROS 同由 OSRF 维护
- 支持 ODE 物理引擎，可模拟传感器（激光雷达、摄像头、IMU 等）

```bash
# 启动仿真机器人
roslaunch bobac3_description gazebo.launch
```

> ⚠️ 虚拟机中 Gazebo 容易崩溃，如遇问题执行：
> `echo "export SVGA_VGPU10=0" >> ~/.bashrc`

### 仿真机器人话题

| 话题 | 内容 |
|------|------|
| `/scan` | 激光雷达数据 |
| `/bottom_camera/*` | 底部相机数据 |
| `/imu` | IMU 数据 |
| `/berxel_base/*` | 深度相机数据 |

### Rviz 数据可视化

1. 打开 Rviz → 点击 `Add` → 选择 `By topic`
2. 添加 `LaserScan` 可视化激光雷达数据

---

## 第6次课：ROS 服务

### Service 通信原理

- **同步双向**：Client 发送 request → Server 处理 → 返回 reply
- Client 等待时处于**阻塞**状态
- 适合非周期性的请求-应答场景

**Topic vs Service：**

| 特性 | Topic | Service |
|------|-------|---------|
| 通信方式 | 异步单向 | 同步双向 |
| 模式 | 发布/订阅 | 请求/应答 |
| 适用场景 | 周期性数据 | 按需查询 |
| 资源占用 | 持续传输 | 按需调用 |

### 自定义服务类型

**创建 srv 文件** `srv/myTestSrv.srv`：
```
int32 index
---
int32 result
```
> `---` 上方为请求部分，下方为响应部分

### 编写 Server

```cpp
#include "third_pkg/myTestSrv.h"

bool checkBattery(third_pkg::myTestSrv::Request &req,
                  third_pkg::myTestSrv::Response &res)
{
    if (1 == req.index)
    {
        res.result = battery;
    }
    return true;
}

// 注册服务
ros::ServiceServer service = n.advertiseService("batteryStatus", checkBattery);
```

### 编写 Client

```cpp
ros::ServiceClient client = n.serviceClient<third_pkg::myTestSrv>("batteryStatus");
third_pkg::myTestSrv mySrv;
mySrv.request.index = 1;

if (client.call(mySrv))
{
    ROS_WARN("Battery: %d", mySrv.response.result);
}
```

---

## 实验2：机器人移动控制

### 通过话题控制机器人运动

发布 `cmd_vel` 话题控制机器人移动：

```cpp
#include "geometry_msgs/Twist.h"

ros::Publisher Vel_c = n.advertise<geometry_msgs::Twist>("cmd_vel", 10);

geometry_msgs::Twist vel_msg;
vel_msg.linear.x = 0.05;    // 前进速度 m/s
vel_msg.angular.z = 0.78;   // 转弯速度 rad/s
Vel_c.publish(vel_msg);
```

---

## 第8次课：语音交互

### 1. 语音采集

```cpp
#include <robot_audio/Collect.h>

ros::ServiceClient collect_client = n.serviceClient<robot_audio::Collect>("voice_collect");
robot_audio::Collect srv;
srv.request.collect_flag = 1;  // 1=采集, 0=不采集
collect_client.call(srv);
// srv.response.voice_filename 为音频保存路径
```

### 2. 语音听写

```cpp
#include <robot_audio/robot_iat.h>

ros::ServiceClient iat_client = n.serviceClient<robot_audio::robot_iat>("voice_iat");
robot_audio::robot_iat srv;
srv.request.audiopath = audio_file_path;
iat_client.call(srv);
// srv.response.text 为听写结果
```

### 3. 语义理解

```cpp
#include <robot_audio/robot_semanteme.h>

robot_audio::robot_semanteme srv;
srv.request.mode = 1;           // 1=文字输入, 2=音频输入
srv.request.textorpath = "带我去卧室";
client.call(srv);

// srv.response.tech: "system" 系统技能 / "user" 用户自定义技能
// srv.response.intent: "robot_nav" 导航意图 / "robot_control" 控制意图
// srv.response.slots_name / slots_value: 语义实体
```

---

## 第9次课：地图构建（SLAM）

### Gmapping 算法

- 基于激光雷达 + 里程计方案
- 采用 RBPF（粒子滤波）方法
- 成熟稳定，广泛应用于 ROS 机器人

### 操作步骤

```bash
# 服务组
roslaunch bobac3_slam bobac3_slam_sim.launch

# 工业组
roslaunch oryxbot_slam oryxbot_slam_sim.launch

# 键盘控制
rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

**键盘控制：** `i`前进, `,`后退, `j`左转, `l`右转, `k`停止

```bash
# 保存地图
cd ~/ros_workspace/
rosrun map_server map_saver -f demo
```

---

## 第10次课：自主导航

### Navigation Stack

- **move_base**：核心路径规划节点
- 包含全局/局部路径规划、代价地图、恢复行为等插件
- 需要安装：`sudo apt-get install ros-melodic-dwa-local-planner`

### 单点导航

```cpp
#include <move_base_msgs/MoveBaseAction.h>
#include <actionlib/client/simple_action_client.h>

actionlib::SimpleActionClient<move_base_msgs::MoveBaseAction> ac("move_base", true);
ac.waitForServer();

move_base_msgs::MoveBaseGoal goal;
goal.target_pose.header.frame_id = "map";
goal.target_pose.pose.position.x = 2.477;
goal.target_pose.pose.position.y = 0.151;
goal.target_pose.pose.orientation.w = 1;

ac.sendGoal(goal);
while (!(ac.getState() == actionlib::SimpleClientGoalState::SUCCEEDED))
{
    usleep(1000 * 20);
}
```

### 多点巡航

```cpp
// 定义多个目标点
move_base_msgs::MoveBaseGoal goal1, goal2, goal3;
// ... 设置各目标点坐标 ...

int Num = 2;  // 巡航圈数
for (int i = 0; i < Num; i++)
{
    ac.sendGoal(goal1);
    // 等待到达...
    ac.sendGoal(goal2);
    // 等待到达...
    ac.sendGoal(goal3);
    // 等待到达...
}
```

### 获取地图坐标

```bash
# 启动导航
roslaunch bobac3_navigation demo_nav_2d.launch

# 键盘控制到目标位置
rosrun teleop_twist_keyboard teleop_twist_keyboard.py

# 查看坐标
rosrun tf tf_echo /map base_footprint
```

---

## 第11次课：语音导航

### 融合语音 + 导航

```cpp
struct Point {
    float x, y, z, w;
    string name;
    string present;  // 到达后的介绍语
};

Point m_point[5] = {
    {1.026, 1.109, -0.012, 1.000, "深圳", "深圳，中国的科创中心"},
    {1.073, 2.120, 0.013, 1.000, "上海", "上海，中国的经济中心"},
    // ...
};

// 流程：语音采集 → 语音听写 → 关键词匹配 → 导航 → 播报介绍
while (ros::ok()) {
    dir = audio.voice_collect();
    text = audio.voice_dictation(dir.c_str());
    if (text.find("元宝元宝") != string::npos) {
        audio.voice_tts("哎，什么事呀");
        // 再次采集，判断导航意图
        for (int i = 0; i < 5; i++) {
            if (text.find(m_point[i].name) != string::npos) {
                audio.goto_nav(&m_point[i]);
                audio.voice_tts(m_point[i].present.c_str());
            }
        }
    }
}
```

---

## 第12次课：人脸识别

### face_rec 功能包

- 基于 dlib 深度学习模型，准确率 99.38%
- 提供 Service 和 Topic 两种调用方式

### 服务调用方式

```bash
# 启动服务
roslaunch face_rec face_rec_service.launch

# 调用识别
rosservice call /face_recognition_results "mode: 1
image_path: ''"
```

### 编程调用

```cpp
#include <face_rec/recognition_results.h>

ros::ServiceClient client = n.serviceClient<face_rec::recognition_results>("face_recognition_results");
face_rec::recognition_results srv;
srv.request.mode = 1;  // 1=相机, 2=图片

if (client.call(srv) && srv.response.success) {
    int face_num = srv.response.result.num;
    for (int i = 0; i < face_num; i++) {
        string name = srv.response.result.face_data[i].name;
    }
}
```

### 添加人脸库

在 `face_data` 目录下创建以人名命名的文件夹，放入该人的照片即可。

---

## 补充：机器人二次定位（自主充电）

### 原理

1. 识别充电桩上的 AR 码
2. 调用 `/track` 服务跟踪 AR 码，调整姿态
3. 调用 `/relative_move` 服务，控制机器人走上充电桩

```cpp
// 跟踪 AR 码
ar_pose::Track Track_data;
Track_data.request.ar_id = 0;       // 跟踪 0 号 AR 码
Track_data.request.goal_dist = 0.3; // 距离 0.3m
ar_track_client.call(Track_data);

// 相对运动，走上充电桩
relative_move::SetRelativeMove RelativeMove_data;
RelativeMove_data.request.goal.x = -0.1;  // 后退 0.1m
RelativeMove_data.request.global_frame = "odom";
relative_move_client.call(RelativeMove_data);
```

---

## 📌 附录：ROS 开发通用流程

```
1. 创建功能包    catkin_create_pkg <name> <deps>
2. 编写源码      src/xxx.cpp
3. 修改 CMake    add_executable + target_link_libraries
4. 编译          cd ~/ros_workspace && catkin_make
5. 刷新环境      source devel/setup.bash
6. 启动          roscore → rosrun / roslaunch
```

### 常用 ROS 命令速查

| 命令 | 用途 |
|------|------|
| `roscore` | 启动 Master |
| `rosrun <pkg> <node>` | 运行节点 |
| `roslaunch <pkg> <file.launch>` | 批量启动 |
| `rostopic list` | 列出话题 |
| `rostopic echo /topic` | 打印话题数据 |
| `rosservice list` | 列出服务 |
| `rosservice call /srv args` | 调用服务 |
| `rosnode list` | 列出节点 |
| `rosnode info /node` | 节点信息 |
| `rqt_graph` | 节点通信可视化 |
| `rosmsg show <type>` | 消息结构 |
| `rosrun tf tf_echo /frame1 /frame2` | 坐标变换 |

---

> 📝 笔记整理自 ROS 机器人课程，基于 ROS Melodic + Ubuntu 18.04
