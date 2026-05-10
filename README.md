# 🤖 ROS 机器人操作系统学习笔记

> 基于 ROS Melodic + Ubuntu 18.04 的机器人开发课程完整笔记

---

## 📋 课程目录

| 课次 | 主题 | 笔记链接 |
|------|------|----------|
| 第1次 | 环境配置 | [→ 查看笔记](./01-环境配置.md) |
| 第2次 | ROS 工程认知 | [→ 查看笔记](./02-ROS工程认知.md) |
| 第3次 | ROS 节点编写 | [→ 查看笔记](./03-ROS节点编写.md) |
| 第4次 | ROS 话题通讯 | [→ 查看笔记](./04-ROS话题通讯.md) |
| 第5次 | 仿真机器人及环境 | [→ 查看笔记](./05-仿真机器人及环境.md) |
| 第6次 | ROS 服务 | [→ 查看笔记](./06-ROS服务.md) |
| 第6次(实验2) | 机器人移动控制 | [→ 查看笔记](./07-机器人移动控制.md) |
| 第8次(1) | 语音采集 | [→ 查看笔记](./08-语音采集.md) |
| 第8次(2) | 语音听写 | [→ 查看笔记](./09-语音听写.md) |
| 第8次(3) | 语义理解 | [→ 查看笔记](./10-语义理解.md) |
| 第9次 | 地图构建（SLAM） | [→ 查看笔记](./11-地图构建.md) |
| 第10次 | 自主导航 | [→ 查看笔记](./12-自主导航.md) |
| 第11次 | 语音导航 | [→ 查看笔记](./13-语音导航.md) |
| 第12次 | 人脸识别 | [→ 查看笔记](./14-人脸识别.md) |
| 补充 | 机器人二次定位（自主充电） | [→ 查看笔记](./15-机器人二次定位.md) |

---

## 🚀 快速参考

### ROS 开发通用流程

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
