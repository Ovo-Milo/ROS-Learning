# 语音听写

[← 返回目录](../README.md)

---

## 实验目的

- 熟悉 robot_audio 功能包中的语音听写服务
- 学习如何启动语音听写服务
- 编写客户端程序听写一段音频的内容

## 服务接口

**服务名**：`/voice_iat`

**类型**：`robot_audio/robot_iat`

```
# Request
string audiopath    # 需要听写的音频路径
---
# Response
string text         # 听写结果
```

---

## 实现代码

```cpp
#include <ros/ros.h>
#include <robot_audio/robot_iat.h>
#include <robot_audio/Collect.h>
#include <string>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "dictation");
    ros::NodeHandle n;

    ros::ServiceClient iat_client = n.serviceClient<robot_audio::robot_iat>("voice_iat");
    ros::ServiceClient collect_client = n.serviceClient<robot_audio::Collect>("voice_collect");

    // 1. 先采集语音
    robot_audio::Collect coll_srv;
    coll_srv.request.collect_flag = 1;
    ros::service::waitForService("voice_collect");
    collect_client.call(coll_srv);
    std::cout << "语音采集结束：" << coll_srv.response.voice_filename << std::endl;

    // 2. 再听写
    robot_audio::robot_iat iat_srv;
    iat_srv.request.audiopath = coll_srv.response.voice_filename;
    ros::service::waitForService("voice_iat");
    iat_client.call(iat_srv);
    std::cout << "听到的内容：" << iat_srv.response.text << std::endl;

    return 0;
}
```

---

## Launch 文件

`launch/voice_dictation.launch`：

```xml
<launch>
    <!-- 实验节点 -->
    <node pkg="bobac3_audio" type="dictation_node" name="dictation" output="screen"/>
    <!-- 语音采集节点 -->
    <node name="voice_collect" pkg="robot_audio" type="voice_collect_node" output="screen">
        <param name="audio_file" type="string" value="./AIUI/audio/audio.wav"/>
    </node>
    <!-- 语音服务 -->
    <node pkg="robot_audio" type="voice_aiui_node" name="voice_aiui_node"/>
</launch>
```

---

## 运行

```bash
roslaunch bobac3_audio voice_dictation.launch
# 终端会显示听写结果
```

---

[← 返回目录](../README.md)
