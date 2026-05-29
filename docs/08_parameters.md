# 8章: パラメータ

**パラメータ** は，ソースコードを変更・再ビルドせずに実行時に値を変えられる仕組みです．ゲインの調整・閾値の変更・ロボット名の設定など，変更頻度が高い値に使います．

---

## パラメータの基本

パラメータは ROS マスター内の **パラメータサーバー** に保存されます．ノードはここから値を読み書きできます．

---

## C++ でパラメータを読み取る

### `nh.param<T>()` ── デフォルト値あり（推奨）

```cpp
int value;
nh.param<int>("my_param", value, 10);
// → "my_param" が設定されていれば value にその値，なければ 10
```

### `nh.getParam()` ── デフォルト値なし

```cpp
int value;
if (nh.getParam("my_param", value))
{
    // 取得成功
}
else
{
    ROS_WARN("パラメータ my_param が設定されていません");
}
```

パラメータが **必須** のときは `getParam()` を使い，失敗したら終了する処理を書くとよいでしょう．

---

## パラメータを使うノード（param_example.cpp）

`~/catkin_ws/src/ros_tutorial/src/param_example.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <std_msgs/String.h>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "param_example");
    ros::NodeHandle nh;

    // パラメータの取得
    std::string robot_name;
    int         publish_rate;
    double      threshold;

    nh.param<std::string>("robot_name",    robot_name,    "robot_A");
    nh.param<int>        ("publish_rate",  publish_rate,  10);
    nh.param<double>     ("threshold",     threshold,     0.5);

    ROS_INFO("ロボット名    : %s",  robot_name.c_str());
    ROS_INFO("パブリッシュレート: %d Hz", publish_rate);
    ROS_INFO("閾値          : %.2f", threshold);

    ros::Publisher pub = nh.advertise<std_msgs::String>("status", 10);
    ros::Rate rate(publish_rate);

    while (ros::ok())
    {
        std_msgs::String msg;
        msg.data = robot_name + " is running";
        pub.publish(msg);

        ros::spinOnce();
        rate.sleep();
    }

    return 0;
}
```

### CMakeLists.txt に追記

```cmake
add_executable(param_example src/param_example.cpp)
target_link_libraries(param_example ${catkin_LIBRARIES})
```

---

## launch ファイルでパラメータを設定する

`launch/` ディレクトリがなければ作成します：

```bash
mkdir -p ~/catkin_ws/src/ros_tutorial/launch
```

`~/catkin_ws/src/ros_tutorial/launch/param_example.launch` を作成：

```xml
<launch>
  <node pkg="ros_tutorial" type="param_example" name="param_example" output="screen">
    <param name="robot_name"   value="my_robot"/>
    <param name="publish_rate" value="2"/>
    <param name="threshold"    value="0.8"/>
  </node>
</launch>
```

### 実行

```bash
cd ~/catkin_ws
catkin build
roslaunch ros_tutorial param_example.launch
```

出力：
```
[ INFO]: ロボット名    : my_robot
[ INFO]: パブリッシュレート: 2 Hz
[ INFO]: 閾値          : 0.80
```

---

## プライベートパラメータとグローバルパラメータ

### NodeHandle の種類

ROS のパラメータには **名前空間** があります．`NodeHandle` の作り方で変わります．

```cpp
// グローバル名前空間（"/" 以下にあるパラメータ）
ros::NodeHandle nh;

// プライベート名前空間（"/ノード名/" 以下にあるパラメータ）
ros::NodeHandle private_nh("~");
```

**launch ファイルの `<param>` はプライベート名前空間に設定されます．**

```cpp
// launch ファイルの <param name="robot_name" ...> を読む場合は
// プライベート NodeHandle を使う
std::string robot_name;
private_nh.param<std::string>("robot_name", robot_name, "default");
```

> 慣習として，ノード固有の設定パラメータにはプライベート名前空間 (`"~"`) を使います．

実際にどちらを使うべきかは，以下で判断します：
- ノード専用の設定 → `ros::NodeHandle private_nh("~")`
- 複数ノードで共有する値 → `ros::NodeHandle nh`

---

## rosparam コマンド

実行中にパラメータを確認・変更するコマンドです．

```bash
# パラメータ一覧
rosparam list

# パラメータの値を確認
rosparam get /param_example/robot_name

# パラメータの値をセット
rosparam set /param_example/publish_rate 5

# パラメータを YAML ファイルに保存
rosparam dump params.yaml

# YAML ファイルからパラメータを読み込む
rosparam load params.yaml
```

> `rosparam set` でパラメータを変更しても，**すでに起動しているノードには即座に反映されません**．ノードが起動時に一度だけ読み取るような実装の場合，再起動が必要です．動的に変更したい場合は `dynamic_reconfigure` パッケージを使います（このチュートリアルの範囲外）．

---

## YAML ファイルでパラメータをまとめる

複数のパラメータは YAML ファイルにまとめると管理しやすいです．

`~/catkin_ws/src/ros_tutorial/config/robot_params.yaml` を作成：

```bash
mkdir -p ~/catkin_ws/src/ros_tutorial/config
```

```yaml
robot_name: "cool_robot"
publish_rate: 5
threshold: 0.75
```

launch ファイルから読み込む：

```xml
<launch>
  <node pkg="ros_tutorial" type="param_example" name="param_example" output="screen">
    <!-- YAML ファイルを読み込む -->
    <rosparam file="$(find ros_tutorial)/config/robot_params.yaml" command="load"/>
  </node>
</launch>
```

`$(find パッケージ名)` は launch ファイル内でパッケージのインストールパスを取得する記法です（例：`$(find ros_tutorial)` → `~/catkin_ws/src/ros_tutorial`）．パスをハードコードせずに済むため，環境が変わっても動作します（詳しくは 9章）．

---

[→ 9章: launch ファイル](09_launch_files.md)
