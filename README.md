<p align="center">
  <img width="460" src="https://img.shields.io/badge/🤖_ROS_机器人学习笔记-2b5797?style=for-the-badge&labelColor=3b7dd8&color=2b5797" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Ubuntu_18.04-E95420?style=flat-square&logo=ubuntu&logoColor=white" />
  <img src="https://img.shields.io/badge/ROS_Melodic-22314E?style=flat-square&logo=ros&logoColor=white" />
  <img src="https://img.shields.io/badge/C++-00599C?style=flat-square&logo=cplusplus&logoColor=white" />
  <img src="https://img.shields.io/badge/Gazebo-777777?style=flat-square" />
  <img src="https://img.shields.io/badge/15_Lessons-4CAF50?style=flat-square" />
</p>

<p align="center">
  从零开始的 ROS 机器人开发完整学习路径<br/>
  环境搭建 → 通信机制 → 语音交互 → 导航定位 → 视觉识别
</p>

---

## 📖 课程导航

### 🟢 基础篇 — 搭建你的第一个 ROS 工程

| # | 主题 | 核心内容 | 笔记 |
|---|------|----------|------|
| 1 | 环境配置 | VMware + Ubuntu 虚拟机安装 | [📖 01](./01-环境配置) |
| 2 | ROS 工程认知 | 工作空间、功能包、catkin 编译 | [📖 02](./02-ROS工程认知) |
| 3 | 节点编写 | Node、Master、roscpp、launch | [📖 03](./03-ROS节点编写) |

### 🟡 进阶篇 — 掌握 ROS 核心通信机制

| # | 主题 | 核心内容 | 笔记 |
|---|------|----------|------|
| 4 | 话题通讯 | Topic 发布/订阅、自定义消息 | [📖 04](./04-ROS话题通讯) |
| 5 | 仿真环境 | Gazebo 仿真 + Rviz 可视化 | [📖 05](./05-仿真机器人及环境) |
| 6 | 服务通讯 | Service 请求/应答模型 | [📖 06](./06-ROS服务) |
| 6+ | 移动控制 | cmd_vel 话题控制机器人运动 | [📖 07](./07-机器人移动控制) |

### 🔴 实战篇 — 语音、导航、视觉

| # | 主题 | 核心内容 | 笔记 |
|---|------|----------|------|
| 8.1 | 语音采集 | 麦克风录音服务调用 | [📖 08](./08-语音采集) |
| 8.2 | 语音听写 | 语音转文字（ASR） | [📖 09](./09-语音听写) |
| 8.3 | 语义理解 | AIUI 意图识别与槽位提取 | [📖 10](./10-语义理解) |
| 9 | 地图构建 | SLAM + gmapping 建图 | [📖 11](./11-地图构建) |
| 10 | 自主导航 | Navigation Stack + 多点巡航 | [📖 12](./12-自主导航) |
| 11 | 语音导航 | 语音指令 → 导航 → 语音播报 | [📖 13](./13-语音导航) |
| 12 | 人脸识别 | dlib + ROS 服务封装 | [📖 14](./14-人脸识别) |
| — | 自主充电 | AR 码跟踪 + 二次定位 | [📖 15](./15-机器人二次定位) |

---

## 🗺️ 学习路线

<p align="center">

<!-- 基础篇 -->
<a href="./01-环境配置"><img src="https://img.shields.io/badge/1._环境配置-4CAF50?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./02-ROS工程认知"><img src="https://img.shields.io/badge/2._ROS工程认知-4CAF50?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./03-ROS节点编写"><img src="https://img.shields.io/badge/3._节点编写-4CAF50?style=flat&logoColor=white" /></a>

<br/>

<!-- 进阶篇 -->
<a href="./04-ROS话题通讯"><img src="https://img.shields.io/badge/4._话题通讯-FF9800?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./05-仿真机器人及环境"><img src="https://img.shields.io/badge/5._仿真环境-FF9800?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./06-ROS服务"><img src="https://img.shields.io/badge/6._服务通讯-FF9800?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./07-机器人移动控制"><img src="https://img.shields.io/badge/6+._移动控制-FF9800?style=flat&logoColor=white" /></a>

<br/>

<!-- 实战篇 -->
<a href="./08-语音采集"><img src="https://img.shields.io/badge/8._语音交互-f44336?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./11-地图构建"><img src="https://img.shields.io/badge/9._地图构建-f44336?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./12-自主导航"><img src="https://img.shields.io/badge/10._自主导航-f44336?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./13-语音导航"><img src="https://img.shields.io/badge/11._语音导航-f44336?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./14-人脸识别"><img src="https://img.shields.io/badge/12._人脸识别-f44336?style=flat&logoColor=white" /></a>
&nbsp;→&nbsp;
<a href="./15-机器人二次定位"><img src="https://img.shields.io/badge/自主充电-f44336?style=flat&logoColor=white" /></a>

</p>

---

## ⚡ 快速参考

<details>
<summary>🔧 ROS 常用命令速查</summary>

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
| `rqt_graph` | 节点通信可视化 |
| `rosmsg show <type>` | 消息结构 |
| `rosrun tf tf_echo /f1 /f2` | 坐标变换 |

</details>

<details>
<summary>📦 标准开发流程</summary>

```
1. catkin_create_pkg <name> <deps>   # 创建功能包
2. 编写源码 src/xxx.cpp               # 写代码
3. 修改 CMakeLists.txt                # add_executable + target_link_libraries
4. catkin_make                        # 编译
5. source devel/setup.bash            # 刷新环境
6. roslaunch / rosrun                 # 启动
```

</details>

<details>
<summary>⚖️ Topic vs Service 对比</summary>

| 特性 | Topic | Service |
|------|-------|---------|
| 通信方式 | 异步单向 | 同步双向 |
| 模式 | 发布/订阅 | 请求/应答 |
| 适用场景 | 周期性数据 | 按需查询 |
| 资源占用 | 持续传输 | 按需调用 |
| 类比 | 广播 | 打电话 |

</details>

---

## 📚 参考资源

- [ROS Wiki](http://wiki.ros.org/) — 官方文档
- [Navigation Stack](https://github.com/ros-planning/navigation) — 导航功能包源码
- [Gmapping](http://wiki.ros.org/gmapping) — SLAM 算法文档
- [face_recognition](https://github.com/ageitgey/face_recognition) — 人脸识别库

---

<p align="center">
  📝 整理自 ROS 机器人课程 · ROS Melodic + Ubuntu 18.04
</p>
