# 语义理解

[← 返回目录](../README.md)

---

## 实验目的

- 熟悉 robot_audio 功能包中的语音服务
- 学习如何启动 AIUI 语义理解服务
- 编写客户端程序让机器人对语音做出回答

## 服务接口

**服务名**：`voice_aiui`

**类型**：`robot_audio/robot_semanteme`

```
# Request
int32 mode          # 1=文字输入, 2=音频输入
string textorpath   # mode=1 时为文字, mode=2 时为音频路径
---
# Response
string tech         # 技能: "system" 系统技能 / "user" 用户自定义技能
string iat          # 听写内容
string anwser       # 回答
string intent       # 意图: "robot_nav" 导航 / "robot_control" 控制
string[] slots_name # 语义实体名称
string[] slots_value # 语义实体值
```

---

## 技能分类

| 类型 | 说明 | 示例 |
|------|------|------|
| `system` | 系统技能 | "今天天气怎么样" |
| `user` (robot_nav) | 导航意图 | "带我去卧室" |
| `user` (robot_control) | 控制意图 | "前进0.2米" |

---

## 实现代码

```cpp
#include <ros/ros.h>
#include <robot_audio/robot_semanteme.h>
#include <string>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "semanteme");
    ros::NodeHandle n;

    ros::ServiceClient client = n.serviceClient<robot_audio::robot_semanteme>("voice_aiui");
    ros::service::waitForService("voice_aiui");

    robot_audio::robot_semanteme srv;
    srv.request.mode = 1;
    srv.request.textorpath = "带我去卧室";  // 可改为其他测试文本

    std::cout << "问题：" << srv.request.textorpath << std::endl;
    client.call(srv);

    // 系统技能
    if (srv.response.tech == "system") {
        std::cout << "回答：" << srv.response.anwser << std::endl;
    }
    // 用户自定义技能
    else if (srv.response.tech == "user") {
        // 导航意图
        if (srv.response.intent == "robot_nav") {
            std::cout << srv.response.slots_name[0] << ": "
                      << srv.response.slots_value[0] << std::endl;
        }
        // 控制意图
        else if (srv.response.intent == "robot_control") {
            for (int i = 0; i < srv.response.slots_name.size(); i++) {
                std::cout << srv.response.slots_name[i] << ": "
                          << srv.response.slots_value[i] << std::endl;
            }
        }
        std::cout << "回答：" << srv.response.anwser << std::endl;
    }
    return 0;
}
```

---

## Launch 文件

`launch/voice_semanteme.launch`：

```xml
<launch>
    <node pkg="bobac3_audio" type="semanteme_node" name="semanteme" output="screen"/>
    <node pkg="robot_audio" type="voice_aiui_node" name="voice_aiui_node"/>
    <node name="voice_collect" pkg="robot_audio" type="voice_collect_node" output="screen">
        <param name="audio_file" type="string" value="./AIUI/audio/audio.wav"/>
    </node>
</launch>
```

---

## 运行测试

```bash
roslaunch bobac3_audio voice_semanteme.launch
```

修改 `srv.request.textorpath` 测试不同场景：

| 输入 | 技能 | 效果 |
|------|------|------|
| "今天天气怎么样" | system | 返回天气信息 |
| "带我去卧室" | user / robot_nav | 提取导航地点 |
| "前进0.2米" | user / robot_control | 提取运动方向和距离 |

---

[← 返回目录](../README.md)
