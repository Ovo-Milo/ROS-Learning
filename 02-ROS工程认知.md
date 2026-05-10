# 第2次课：ROS 工程认知

[← 返回目录](./README.md)

---

## 核心概念

### 工作空间（Workspace）

```
ros_workspace/
├── build/    # 编译中间文件
├── devel/    # 编译生成的可执行代码、库文件、自动生成的头文件
└── src/      # 功能包目录
```

**创建与编译：**

```bash
mkdir -p ~/ros_workspace/src
cd ~/ros_workspace/
catkin_make
source ~/ros_workspace/devel/setup.bash

# 永久生效（所有终端有效）
echo "source ~/ros_workspace/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### catkin 编译系统

编译流程：
1. 在 `catkin_ws/src/` 下递归查找每个 ROS package
2. 依据 `CMakeLists.txt` 生成 makefiles（放在 `build/`）
3. `make` 编译链接，生成可执行文件（放在 `devel/`）

> ⚠️ `catkin_make` 必须在工作空间根目录下执行，否则编译不会成功

---

## 功能包（Package）

### Package 结构

```
package/
├── CMakeLists.txt    # 编译规则（必须）
├── package.xml       # 描述信息（必须）
├── src/              # 源代码
├── include/          # C++ 头文件
├── scripts/          # 可执行脚本 (.sh, .py)
├── msg/              # 自定义消息 (.msg)
├── srv/              # 自定义服务 (.srv)
├── models/           # 3D 模型 (.sda, .stl, .dae)
├── urdf/             # 机器人模型描述 (.urdf, .xacro)
└── launch/           # launch 文件 (.launch, .xml)
```

> 只有 `CMakeLists.txt` 和 `package.xml` 是必须的，其余根据需要创建

### 创建功能包

```bash
cd ~/ros_workspace/src
catkin_create_pkg <包名> <依赖1> <依赖2> ...

# 示例：创建 test_pkg，依赖 roscpp、rospy、std_msgs
catkin_create_pkg test_pkg roscpp rospy std_msgs
```

> ⚠️ 包名只能使用小写字母、数字和下划线，首字符必须是小写字母

---

## 常用命令

### rospack — 包管理

| 命令 | 作用 |
|------|------|
| `rospack help` | 显示用法 |
| `rospack list` | 列出本机所有 package |
| `rospack find [pkg]` | 定位某个 package |
| `rospack depends [pkg]` | 显示依赖包 |
| `rospack profile` | 刷新 package 位置记录 |

### roscd / rosls — 快速导航

| 命令 | 作用 |
|------|------|
| `roscd [pkg]` | cd 到 package 所在路径 |
| `rosls [pkg]` | 列出 package 下的文件 |

### rosdep — 依赖管理

```bash
# 安装工作空间中所有 package 的依赖项
rosdep install --from-paths src --ignore-src --rosdistro=kinetic -y
```

---

[← 返回目录](./README.md)
