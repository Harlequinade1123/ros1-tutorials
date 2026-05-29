# 7章: カスタムメッセージ

これまでは `std_msgs::String`（文字列）を使いました．実際のロボット開発では「センサー名・計測値・ID を 1 つにまとめて送りたい」といったケースがよくあります．このような場合に **カスタムメッセージ** を定義します．

---

## メッセージ定義ファイル（.msg）の作成

パッケージ内に `msg/` フォルダを作ります．

```bash
mkdir -p ~/catkin_ws/src/ros_tutorial/msg
```

`~/catkin_ws/src/ros_tutorial/msg/SensorData.msg` を作成：

```
string name
float64 value
int32 id
```

**使える基本型：**

| 型 | 意味 |
|----|------|
| `bool` | 真偽値 |
| `int8`, `int16`, `int32`, `int64` | 整数（ビット数が異なる） |
| `float32`, `float64` | 浮動小数点 |
| `string` | 文字列 |
| `uint8[]` | バイト列（画像データなど） |
| `Header` | タイムスタンプ・フレームID（よく使う） |

他のメッセージ型を入れ子にすることもできます：

```
# 例：geometry_msgs/Point を含む場合
geometry_msgs/Point position
float64 speed
```

---

## ビルド設定の変更

カスタムメッセージを使うには，`CMakeLists.txt` と `package.xml` を変更する必要があります．

### CMakeLists.txt の変更

変更箇所が複数あります．元のファイルを開いて，以下のように変更・追記してください．

#### 1. `find_package` に `message_generation` を追加

変更前：
```cmake
find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
)
```

変更後：
```cmake
find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  message_generation
)
```

#### 2. `add_message_files` を追加（`catkin_package()` の前）

```cmake
add_message_files(
  FILES
  SensorData.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)
```

`add_message_files` は `.msg` ファイル用です（`.srv` の場合は `add_service_files`，`.action` の場合は `add_action_files` を使います）．各設定の意味は [5章](05_service.md) の「追加した設定の意味」の表を参照してください．

#### 3. `catkin_package` に `message_runtime` を追加

変更前：
```cmake
catkin_package()
```

変更後：
```cmake
catkin_package(
  CATKIN_DEPENDS roscpp std_msgs message_runtime
)
```

#### 4. 各 `add_executable` に `add_dependencies` を追加

カスタムメッセージのヘッダファイルが生成されてからコンパイルするために必要です．

```cmake
add_executable(talker src/talker.cpp)
target_link_libraries(talker ${catkin_LIBRARIES})
add_dependencies(talker ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(listener src/listener.cpp)
target_link_libraries(listener ${catkin_LIBRARIES})
add_dependencies(listener ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
```

### package.xml の変更

`<buildtool_depend>catkin</buildtool_depend>` の下に以下を追加：

```xml
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
```

---

## カスタムメッセージを使うノード

### Publisher（sensor_publisher.cpp）

`~/catkin_ws/src/ros_tutorial/src/sensor_publisher.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <ros_tutorial/SensorData.h>   // 自分で定義したメッセージ

int main(int argc, char **argv)
{
    ros::init(argc, argv, "sensor_publisher");
    ros::NodeHandle nh;

    ros::Publisher pub = nh.advertise<ros_tutorial::SensorData>("sensor_topic", 10);
    ros::Rate rate(1);  // 1Hz

    int id = 0;
    while (ros::ok())
    {
        ros_tutorial::SensorData msg;
        msg.name  = "temperature";
        msg.value = 25.0 + id * 0.5;   // 疑似センサー値
        msg.id    = id++;

        ROS_INFO("送信: name=%s, value=%.1f, id=%d",
                 msg.name.c_str(), msg.value, msg.id);
        pub.publish(msg);

        ros::spinOnce();
        rate.sleep();
    }

    return 0;
}
```

### Subscriber（sensor_subscriber.cpp）

`~/catkin_ws/src/ros_tutorial/src/sensor_subscriber.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <ros_tutorial/SensorData.h>

void sensorCallback(const ros_tutorial::SensorData::ConstPtr& msg)
{
    ROS_INFO("受信: name=%s, value=%.1f, id=%d",
             msg->name.c_str(), msg->value, msg->id);
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "sensor_subscriber");
    ros::NodeHandle nh;

    ros::Subscriber sub = nh.subscribe("sensor_topic", 10, sensorCallback);
    ros::spin();

    return 0;
}
```

### CMakeLists.txt にビルド設定を追加

```cmake
add_executable(sensor_publisher src/sensor_publisher.cpp)
target_link_libraries(sensor_publisher ${catkin_LIBRARIES})
add_dependencies(sensor_publisher ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(sensor_subscriber src/sensor_subscriber.cpp)
target_link_libraries(sensor_subscriber ${catkin_LIBRARIES})
add_dependencies(sensor_subscriber ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
```

---

## ビルドと実行

```bash
cd ~/catkin_ws
catkin build
```

**ターミナル 1：**
```bash
roscore
```

**ターミナル 2：**
```bash
rosrun ros_tutorial sensor_publisher
```

**ターミナル 3：**
```bash
rosrun ros_tutorial sensor_subscriber
```

### メッセージ型の確認

```bash
rosmsg show ros_tutorial/SensorData
```

出力：
```
string name
float64 value
int32 id
```

---

## 現時点での CMakeLists.txt 全体像

混乱しないよう，この時点での `CMakeLists.txt` の完全な内容を示します．

```cmake
cmake_minimum_required(VERSION 3.0.2)
project(ros_tutorial)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
  actionlib
  actionlib_msgs
  message_generation
)

add_service_files(
  FILES
  AddTwoInts.srv
)

add_action_files(
  FILES
  CountDown.action
)

add_message_files(
  FILES
  SensorData.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
  actionlib_msgs
)

catkin_package(
  CATKIN_DEPENDS roscpp std_msgs actionlib actionlib_msgs message_runtime
)

include_directories(
  ${catkin_INCLUDE_DIRS}
)

add_executable(hello_node src/hello.cpp)
target_link_libraries(hello_node ${catkin_LIBRARIES})

add_executable(talker src/talker.cpp)
target_link_libraries(talker ${catkin_LIBRARIES})
add_dependencies(talker ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(listener src/listener.cpp)
target_link_libraries(listener ${catkin_LIBRARIES})
add_dependencies(listener ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(relay src/relay.cpp)
target_link_libraries(relay ${catkin_LIBRARIES})
add_dependencies(relay ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(add_two_ints_server src/add_two_ints_server.cpp)
target_link_libraries(add_two_ints_server ${catkin_LIBRARIES})
add_dependencies(add_two_ints_server ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(add_two_ints_client src/add_two_ints_client.cpp)
target_link_libraries(add_two_ints_client ${catkin_LIBRARIES})
add_dependencies(add_two_ints_client ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(count_down_server src/count_down_server.cpp)
target_link_libraries(count_down_server ${catkin_LIBRARIES})
add_dependencies(count_down_server ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(count_down_client src/count_down_client.cpp)
target_link_libraries(count_down_client ${catkin_LIBRARIES})
add_dependencies(count_down_client ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(sensor_publisher src/sensor_publisher.cpp)
target_link_libraries(sensor_publisher ${catkin_LIBRARIES})
add_dependencies(sensor_publisher ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(sensor_subscriber src/sensor_subscriber.cpp)
target_link_libraries(sensor_subscriber ${catkin_LIBRARIES})
add_dependencies(sensor_subscriber ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
```

---

[→ 8章: パラメータ](08_parameters.md)
