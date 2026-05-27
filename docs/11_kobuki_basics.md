# 11章: Kobuki 基礎 ── 速度コマンドを送る

## この章を始める前に

> **セットアップについて**
>
> Kobuki ドライバ（`kobuki_ros` など）のインストールや USB 接続設定はこの教材では扱いません。  
> 環境構築済みの PC を使用してください。

> **動作環境の制約（必ず読んでください）**
>
> - **動作スペースが非常に狭い（0.1〜0.2 m 程度）** ため、移動距離は小さく設定してください
> - **有線接続のため、旋回しすぎるとケーブルが絡まります**。旋回は最小限にとどめ、必ずケーブルの取り回しを確認しながら実行してください
> - コードを実行する前に **手でロボットを止められる準備** をしておいてください
> - 速度は **0.1 m/s 以下** を基本とします

---

## パッケージの作成

Kobuki 用のコードは専用パッケージ `kobuki_tutorial` にまとめます。

```bash
cd ~/catkin_ws/src
catkin_create_pkg kobuki_tutorial roscpp geometry_msgs sensor_msgs nav_msgs
```

作成後、一度ビルドしておきます：

```bash
cd ~/catkin_ws && catkin build
source devel/setup.bash
```

---

## Kobuki とは

**Kobuki** は韓国 Yujin Robot 社の差動二輪ロボットです。TurtleBot2 のベースとして広く使われています。

---

## Kobuki の速度制御：Twist メッセージ

`/mobile_base/commands/velocity` トピックに **`geometry_msgs/Twist`** を送ります。

```
geometry_msgs/Twist:
  linear:
    x: 前後方向の速度 [m/s]  ← 正が前進、負が後退
    y: 0.0（差動二輪は横移動不可）
    z: 0.0
  angular:
    x: 0.0
    y: 0.0
    z: 旋回角速度 [rad/s]   ← 正が左旋回、負が右旋回
```

**このチュートリアルでの速度設定：**

| 項目 | 推奨値 | 理由 |
|------|--------|------|
| 直進速度 | 0.1 m/s | 狭いスペースで制御しやすい |
| 旋回速度 | 使用しない | ケーブル絡まり防止 |

### コマンドラインから動作確認（1 回だけ送信）

```bash
# ゆっくり前進（0.5 秒で止まるので手を添えておくこと）
rostopic pub -1 /mobile_base/commands/velocity geometry_msgs/Twist \
  "linear: {x: 0.1, y: 0.0, z: 0.0} angular: {x: 0.0, y: 0.0, z: 0.0}"

# 停止
rostopic pub -1 /mobile_base/commands/velocity geometry_msgs/Twist \
  "linear: {x: 0.0, y: 0.0, z: 0.0} angular: {x: 0.0, y: 0.0, z: 0.0}"
```

> `-1` オプションは「1 回だけ送信して終了」です。繰り返し送り続けるよりも安全です。

---

## サンプルコード：短時間前進して停止

`~/catkin_ws/src/kobuki_tutorial/src/move_forward.cpp`

```cpp
#include <ros/ros.h>
#include <geometry_msgs/Twist.h>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "move_forward");
    ros::NodeHandle nh;

    ros::Publisher cmd_pub =
        nh.advertise<geometry_msgs::Twist>("/mobile_base/commands/velocity", 10);

    // Publisher がサブスクライバーと接続されるまで待つ
    ros::Duration(1.0).sleep();

    geometry_msgs::Twist cmd;
    ros::Rate rate(10);
    ros::Time start = ros::Time::now();

    ROS_INFO("前進開始（1 秒後に停止）");

    while (ros::ok())
    {
        double elapsed = (ros::Time::now() - start).toSec();

        if (elapsed < 1.0)
        {
            cmd.linear.x = 0.1;   // 0.1 m/s × 1 s = 約 0.1 m
        }
        else
        {
            cmd.linear.x = 0.0;
            cmd_pub.publish(cmd);
            ROS_INFO("停止");
            break;
        }

        cmd_pub.publish(cmd);
        ros::spinOnce();
        rate.sleep();
    }

    return 0;
}
```

### CMakeLists.txt に追加

```cmake
add_executable(move_forward src/move_forward.cpp)
target_link_libraries(move_forward ${catkin_LIBRARIES})
```

### ビルドと実行

```bash
cd ~/catkin_ws && catkin build
rosrun kobuki_tutorial move_forward
```

---

## 演習 1：指定時間だけ前進して停止

### 課題

コマンドライン引数で前進する **秒数** を指定できるプログラムを書いてください。

```bash
# 0.5 秒前進（約 0.05 m）
rosrun kobuki_tutorial ex1_move_time 0.5
```

> **スペースの目安**: 0.1 m/s × 0.5 s = 0.05 m。壁まで余裕があることを確認してから実行してください。

### ヒント

- `argc < 2` で引数がないときは `ROS_ERROR` を出して `return 1`
- `std::stod(argv[1])` で文字列を `double` に変換できます
- 秒数が大きすぎる場合は上限チェックを入れると安全です（例: `duration > 2.0` なら警告）

<details>
<summary>サンプルコード（考えてから開くこと！）</summary>

```cpp
#include <ros/ros.h>
#include <geometry_msgs/Twist.h>
#include <string>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "ex1_move_time");

    if (argc < 2)
    {
        ROS_ERROR("使い方: ex1_move_time <秒数>");
        return 1;
    }

    double duration = std::stod(argv[1]);

    if (duration > 2.0)
    {
        ROS_WARN("%.1f 秒は長すぎます。スペースを確認してください", duration);
    }

    ROS_INFO("%.1f 秒間前進します（推定 %.3f m）", duration, 0.1 * duration);

    ros::NodeHandle nh;
    ros::Publisher cmd_pub =
        nh.advertise<geometry_msgs::Twist>("/mobile_base/commands/velocity", 10);
    ros::Duration(1.0).sleep();

    geometry_msgs::Twist cmd;
    ros::Rate rate(10);
    ros::Time start = ros::Time::now();

    while (ros::ok())
    {
        double elapsed = (ros::Time::now() - start).toSec();

        if (elapsed >= duration)
        {
            cmd.linear.x = 0.0;
            cmd_pub.publish(cmd);
            ROS_INFO("停止（経過: %.2f 秒）", elapsed);
            break;
        }

        cmd.linear.x = 0.1;
        cmd_pub.publish(cmd);
        ros::spinOnce();
        rate.sleep();
    }

    return 0;
}
```

</details>

---

## 演習 2：往復動作を繰り返す

### 課題

**「前進 → 後退」を N 回繰り返す** プログラムを書いてください。

```bash
# 3 往復する
rosrun kobuki_tutorial ex2_back_and_forth 3
```

**動作イメージ：**
```
スタート位置
  │ ← 約 0.05 m
  ↓
前進（0.5 秒）
  ↓
後退（0.5 秒）← スタート位置に戻る
  ↓
（N 回繰り返す）
```

### ヒント

- 前進・後退の動作を `move(pub, 速度, 秒数)` という補助関数にまとめると `main` がすっきりします
- 後退は `linear.x` を **負** にします（`-0.1`）
- 動作と動作の間に `ros::Duration(0.3).sleep()` で少し止まると安全です

<details>
<summary>サンプルコード（考えてから開くこと！）</summary>

```cpp
#include <ros/ros.h>
#include <geometry_msgs/Twist.h>
#include <string>

void move(ros::Publisher& pub, double linear_x, double duration_sec)
{
    geometry_msgs::Twist cmd;
    cmd.linear.x = linear_x;

    ros::Time start = ros::Time::now();
    ros::Rate rate(10);

    while (ros::ok() && (ros::Time::now() - start).toSec() < duration_sec)
    {
        pub.publish(cmd);
        ros::spinOnce();
        rate.sleep();
    }

    // 停止
    cmd.linear.x = 0.0;
    pub.publish(cmd);
    ros::Duration(0.3).sleep();
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "ex2_back_and_forth");

    if (argc < 2)
    {
        ROS_ERROR("使い方: ex2_back_and_forth <往復回数>");
        return 1;
    }

    int n = std::stoi(argv[1]);
    ROS_INFO("%d 往復します（1 往復 ≒ 0.05 m）", n);

    ros::NodeHandle nh;
    ros::Publisher cmd_pub =
        nh.advertise<geometry_msgs::Twist>("/mobile_base/commands/velocity", 10);
    ros::Duration(1.0).sleep();

    for (int i = 0; i < n; i++)
    {
        ROS_INFO("往復 %d/%d: 前進", i + 1, n);
        move(cmd_pub,  0.1, 0.5);   // 0.1 m/s × 0.5 s ≒ 0.05 m 前進

        ROS_INFO("往復 %d/%d: 後退", i + 1, n);
        move(cmd_pub, -0.1, 0.5);   // 0.05 m 後退
    }

    ROS_INFO("完了！");
    return 0;
}
```

</details>

---

[→ 12章: Kobuki センサー](12_kobuki_sensors.md)
