# 实验2：机器人移动控制

[← 返回目录](../README.md)

---

## 实验目的

1. 熟悉话题通讯机制的原理以及应用场景
2. 学会编写节点，定义发布器，发布指定的话题

## 实验内容

编写代码控制机器人：
1. 以 0.05m/s 速度前进 2 秒
2. 以 0.78rad/s 速度左转 2 秒
3. 停止运动，程序退出

---

## 实现代码

```cpp
#include "ros/ros.h"
#include "geometry_msgs/Twist.h"

int main(int argc, char **argv)
{
    ros::init(argc, argv, "robot_control");
    ros::NodeHandle n;

    // 创建速度发布器，话题名 "cmd_vel"
    ros::Publisher Vel_c = n.advertise<geometry_msgs::Twist>("cmd_vel", 10);
    geometry_msgs::Twist vel_msg;

    // 1. 前进 2 秒
    usleep(1000 * 1000 * 2);
    vel_msg.linear.x = 0.05;   // 0.05 m/s
    vel_msg.angular.z = 0.0;
    Vel_c.publish(vel_msg);
    usleep(1000 * 1000 * 2);

    // 2. 左转 2 秒
    vel_msg.linear.x = 0.0;
    vel_msg.angular.z = 0.78;  // 0.78 rad/s ≈ 45°/s
    Vel_c.publish(vel_msg);
    usleep(1000 * 1000 * 2);

    // 3. 停止
    vel_msg.linear.x = 0.0;
    vel_msg.angular.z = 0.0;
    Vel_c.publish(vel_msg);
    usleep(1000 * 1000 * 2);

    return 0;
}
```

---

## 关键点

- **`cmd_vel`**：ROS 中控制机器人速度的标准话题
- **`geometry_msgs/Twist`**：速度消息类型
  - `linear.x`：前进/后退速度（m/s）
  - `angular.z`：旋转速度（rad/s）

---

## 创建与编译

```bash
cd ~/ros_workspace/src/
catkin_create_pkg robot_control roscpp rospy std_msgs geometry_msgs
cd ~/ros_workspace
catkin_make
```

## 运行测试

```bash
# 服务组：打开仿真机器人
roslaunch bobac3_description gazebo.launch

# 工业组
roslaunch oryxbot_description gazebo.launch

# 运行控制程序
rosrun robot_control robot_control_node
```

---

[← 返回目录](../README.md)
