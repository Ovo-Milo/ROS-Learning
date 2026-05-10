# 地图构建（SLAM）

[← 返回目录](../README.md)

---

## SLAM 算法原理

### Gmapping 算法

- 基于激光雷达 + 里程计方案
- 采用 RBPF（Rao-Blackwellized Particle Filter）粒子滤波方法
- 成熟稳定，广泛应用于 ROS 机器人
- 源码位于 `ros-perception/slam_gmapping` 仓库

### 核心概念

- **SLAM**（Simultaneous Localization and Mapping）：即时定位与地图构建
- 解决机器人在未知环境中同时定位自身位置和构建环境地图的问题
- 是自主导航的基础

---

## 功能包获取

| 组别 | 功能包 | 下载链接 |
|------|--------|----------|
| 服务组 | `bobac3_slam` | https://pan.baidu.com/s/1FKqG9vQVaLLZNoUt5IYMBg （提取码: kfj6） |
| 工业组 | `oryxbot_slam` | 同上 |

下载后放到 `~/ros_workspace/src/` 下编译。

---

## 实验步骤

### 1. 启动仿真 + SLAM

```bash
# 服务组
roslaunch bobac3_slam bobac3_slam_sim.launch

# 工业组
roslaunch oryxbot_slam oryxbot_slam_sim.launch
```

成功后会打开 Gazebo（仿真环境）和 Rviz（地图可视化）。

### 2. 键盘控制机器人

```bash
rosrun teleop_twist_keyboard teleop_twist_keyboard.py
```

**键盘控制：**

| 按键 | 动作 |
|------|------|
| `i` | 前进 |
| `,` | 后退 |
| `j` | 左转 |
| `l` | 右转 |
| `u` | 左前 |
| `o` | 右前 |
| `m` | 左后 |
| `.` | 右后 |
| `k` 或其他 | 停止 |

> ⚠️ 控制机器人时，键盘控制的终端必须是当前选中的终端

### 3. 游历环境建图

控制机器人在环境中游历，Rviz 中会实时绘制地图。

### 4. 保存地图

```bash
cd ~/ros_workspace/
rosrun map_server map_saver -f demo
```

生成两个文件：
- `demo.pgm` — 地图图片
- `demo.yaml` — 地图信息（分辨率、原点等）

---

[← 返回目录](../README.md)
