# 语音导航

[← 返回目录](../README.md)

---

## 实验场景

```
人："元宝 元宝"
机器人：哎什么事儿？
人："导航到上海馆"
机器人：（自主导航移动到上海馆）
机器人到达后播报："上海，中国的经济中心"
```

---

## 实现流程

```
1. 构建地图（参考地图构建实验）
2. 标记坐标点（参考多点导航实验）
3. 语音采集 → 语音听写 → 关键词匹配 → 导航 → 播报介绍
```

---

## 数据结构

```cpp
struct Point {
    float x;           // x 坐标
    float y;           // y 坐标
    float z;           // 姿态 z
    float w;           // 姿态 w
    string name;       // 地点名字
    string present;    // 介绍语
};

// 预设导航点
struct Point m_point[5] = {
    {1.026, 1.109, -0.012, 1.000, "深圳", "深圳，中国的科创中心"},
    {1.073, 2.120, 0.013, 1.000, "上海", "上海，中国的经济中心"},
    {2.48, 0.13, 0.03, 0.99, "北京", "北京，中国的首都"},
    {2.48, 1.077, -0.026, 1.000, "广州", "广州，自古以来都是中国的商都"},
    {2.48, 2.120, -0.032, 0.999, "吉林", "吉林，位于中国的东北是人参之都"}
};
```

---

## 核心代码

```cpp
class interaction {
public:
    string voice_collect();                                    // 语音采集
    string voice_dictation(const char* filename);              // 语音听写
    string voice_tts(const char* text);                        // 语音合成
    void goto_nav(struct Point* point);                        // 导航到目标
private:
    ros::ServiceClient collect_client, dictation_client, tts_client;
    actionlib::SimpleActionClient<move_base_msgs::MoveBaseAction>* ac;
};

// 导航到目标位置
void interaction::goto_nav(struct Point* point) {
    ac = new AC("move_base", true);
    ac->waitForServer();

    move_base_msgs::MoveBaseGoal goal;
    goal.target_pose.header.frame_id = "map";
    goal.target_pose.pose.position.x = point->x;
    goal.target_pose.pose.position.y = point->y;
    goal.target_pose.pose.orientation.z = point->z;
    goal.target_pose.pose.orientation.w = point->w;

    ac->sendGoal(goal);
    ac->waitForResult();

    if (ac->getState() == actionlib::SimpleClientGoalState::SUCCEEDED)
        ROS_INFO("Goal succeeded!");
    delete ac;
}

// 主循环
while (ros::ok()) {
    dir = audio.voice_collect();                            // 1. 采集语音
    text = audio.voice_dictation(dir.c_str());              // 2. 听写

    if (text.find("元宝元宝") != string::npos) {           // 3. 唤醒词
        audio.voice_tts("哎，什么事呀");                    // 4. 应答

        dir = audio.voice_collect();                        // 5. 再次采集
        text = audio.voice_dictation(dir.c_str());          // 6. 听写指令

        if (text.find("导航") != string::npos) {            // 7. 识别导航意图
            for (int i = 0; i < 5; i++) {
                if (text.find(m_point[i].name) != string::npos) {
                    audio.goto_nav(&m_point[i]);            // 8. 导航
                    audio.voice_tts(m_point[i].present);    // 9. 到达播报
                    break;
                }
            }
        }
    }
}
```

---

## Launch 文件

```xml
<launch>
    <node pkg="bobac3_audio" type="nav_node" name="nav" output="screen"/>
    <node name="voice_collect" pkg="robot_audio" type="voice_collect_node" output="screen">
        <param name="audio_file" type="string" value="./AIUI/audio/audio.wav"/>
    </node>
    <node pkg="robot_audio" type="voice_aiui_node" name="voice_aiui_node"/>
</launch>
```

---

## 运行

```bash
# 终端1：启动导航
roslaunch bobac3_navigation demo_nav_2d.launch

# 终端2：启动语音导航
roslaunch bobac3_audio nav.launch
```

---

[← 返回目录](../README.md)
