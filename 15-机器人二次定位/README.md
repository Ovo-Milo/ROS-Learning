# 机器人二次定位（自主充电）

[← 返回目录](../README.md)

---

## 实验目的

- 熟悉 ROS 服务通讯机制
- 学习用 C++ 编写服务客户端
- 学习机器人 AR 码跟踪服务接口
- 进一步熟悉机器人相对运动控制服务接口

## 实验原理

1. 使用底部相机识别充电桩上的 **AR 码**
2. 调用 `/track` 服务跟踪 AR 码，调整机器人姿态正对 AR 码
3. 调用 `/relative_move` 服务，控制机器人走上充电桩

```bash
# 安装依赖（仿真环境需要，实物机器人自带）
sudo apt-get install ros-melodic-ar-track-alvar
```

---

## 服务接口

### /track 服务

**类型**：`ar_pose/Track`

```
# Request
int8 ar_id          # 跟踪的 AR 码 ID
float32 goal_dist   # 机器人中心与 AR 码的目标距离（机器人半径 0.19m）
---
# Response
string message
bool success
```

### /relative_move 服务

**类型**：`relative_move/SetRelativeMove`

```
# Request
geometry_msgs/Pose2D goal      # 移动方向及距离
string global_frame            # 全局坐标系，一般为 /odom，导航时可设为 /map
bool finishStopObstacle        # 避障停车时是否直接结束
---
# Response
bool success
string message
```

---

## 实现代码

```cpp
#include <ros/ros.h>
#include <relative_move/SetRelativeMove.h>
#include <ar_pose/Track.h>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "auto_charge");
    ros::NodeHandle n;

    // 创建客户端
    ros::ServiceClient relative_move_client =
        n.serviceClient<relative_move::SetRelativeMove>("relative_move");
    ros::ServiceClient ar_track_client =
        n.serviceClient<ar_pose::Track>("track");

    relative_move::SetRelativeMove RelativeMove_data;
    ar_pose::Track Track_data;

    ros::service::waitForService("relative_move");
    ros::service::waitForService("track");

    // 1. 跟踪 0 号 AR 码，保持 0.3m 距离
    Track_data.request.ar_id = 0;
    Track_data.request.goal_dist = 0.3;
    ar_track_client.call(Track_data);

    // 2. 相对运动，走上充电桩
    RelativeMove_data.request.goal.x = -0.1;  // 后退 0.1m
    RelativeMove_data.request.global_frame = "odom";
    relative_move_client.call(RelativeMove_data);

    std::cout << "定位完成" << std::endl;
    return 0;
}
```

---

## Launch 文件

```xml
<launch>
    <!-- 底部相机 AR 码检测 -->
    <include file="$(find ar_pose)/launch/ar_base.launch"/>
    <!-- 相对移动服务 -->
    <include file="$(find relative_move)/launch/relative_move.launch"/>
    <!-- 实验节点 -->
    <node pkg="secondary_localization" name="secondary_localization"
          type="track_node" output="screen"/>
</launch>
```

---

## 运行

```bash
# 启动导航
# 真实机器人
roslaunch bobac3_navigation bobac3_nav_2d.launch
# 仿真机器人
roslaunch bobac3_navigation demo_nav_2d.launch

# 将机器人导航至充电桩前（相机可看到 AR 码），然后运行
roslaunch secondary_localization track.launch
```

---

[← 返回目录](../README.md)
