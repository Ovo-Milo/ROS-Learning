# 自主导航

[← 返回目录](../README.md)

---

## Navigation Stack 概述

### 架构

- **move_base**：核心路径规划节点
- 包含多个插件：全局规划、局部规划、代价地图、恢复行为
- 源码：https://github.com/ros-planning/navigation

### 核心组件

| 组件 | 功能 |
|------|------|
| `move_base` | 路径规划器中心节点 |
| `global_planner` | 全局路径规划 |
| `local_planner` | 局部路径规划（避障） |
| `global_costmap` | 全局代价地图 |
| `local_costmap` | 局部代价地图 |
| `recovery_behaviors` | 异常行为恢复 |

### 安装依赖

```bash
sudo apt-get install ros-melodic-dwa-local-planner
```

---

## 导航配置

1. 将 SLAM 构建的地图文件（`.pgm` 和 `.yaml`）拷贝到导航包的 `maps/` 目录
2. 修改 `map_server.launch` 中的地图文件名

### 启动导航

```bash
# 服务组
roslaunch bobac3_navigation demo_nav_2d.launch

# 工业组
roslaunch oryxbot_navigation demo_nav_2d.launch
```

### 手动导航

在 Rviz 中使用 **2D Nav Goal** 工具，点击地图位置设置目标点。

---

## 获取地图坐标

```bash
# 1. 启动导航
roslaunch bobac3_navigation demo_nav_2d.launch

# 2. 键盘控制到目标位置
rosrun teleop_twist_keyboard teleop_twist_keyboard.py

# 3. 查看当前坐标
rosrun tf tf_echo /map base_footprint
```

输出中 `(x, y)` 为坐标，`(z, w)` 为朝向四元数。

---

## 任务一：单点导航

### 创建功能包

```bash
cd ~/ros_workspace/src
catkin_create_pkg nav_goal roscpp rospy actionlib geometry_msgs move_base_msgs
```

### 实现代码

```cpp
#include "ros/ros.h"
#include <move_base_msgs/MoveBaseAction.h>
#include <actionlib/client/simple_action_client.h>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "nav_goal");
    ros::NodeHandle nh;

    // 创建 Action 客户端
    actionlib::SimpleActionClient<move_base_msgs::MoveBaseAction> ac("move_base", true);
    ac.waitForServer();

    // 定义目标
    move_base_msgs::MoveBaseGoal goal;
    goal.target_pose.header.frame_id = "map";
    goal.target_pose.header.stamp = ros::Time::now();
    goal.target_pose.pose.position.x = 2.477;
    goal.target_pose.pose.position.y = 0.151;
    goal.target_pose.pose.orientation.z = 0;
    goal.target_pose.pose.orientation.w = 1;

    ac.sendGoal(goal);

    // 等待到达
    while (!(ac.getState() == actionlib::SimpleClientGoalState::SUCCEEDED))
    {
        usleep(1000 * 20);
    }
    ROS_INFO("机器人到达目标点");
    return 0;
}
```

### 编译运行

```bash
cd ~/ros_workspace && catkin_make
roslaunch bobac3_navigation demo_nav_2d.launch
rosrun nav_goal nav_goal_node
```

---

## 任务二：多点巡航

### 实现思路

```cpp
// 定义多个目标点
move_base_msgs::MoveBaseGoal goals[5];
// ... 设置各目标点坐标 ...

// 循环巡航 N 圈
int Num = 2;
for (int i = 0; i < Num; i++) {
    for (int j = 0; j < 5; j++) {
        ac.sendGoal(goals[j]);
        // 等待到达...
        ROS_INFO("机器人到达目标点%d", j + 1);
    }
}
```

---

## 代码改进：增加前置目标点

为每个目标点增加一个 `_pre` 前置点，让机器人先到达目标附近，再精确到达目标位置，避免路径规划冲突。

---

[← 返回目录](../README.md)
