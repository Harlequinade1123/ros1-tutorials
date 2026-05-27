# 14章: クラスを使った Kobuki プログラミング

13 章ではグローバル変数でコールバックと main 関数の間でデータを共有していました。ノードが複雑になるほどグローバル変数が増えてコードが読みにくくなります。11 章で学んだクラスを使うと整理できます。

---

## グローバル変数の問題点

13 章の演習 4 のコード（抜粋）：

```cpp
bool g_bumper_pressed = false;   // ← ファイルの先頭にグローバル変数

void bumperCallback(...)
{
    g_bumper_pressed = true;     // ← どこからでも変更できてしまう
}

int main(...)
{
    if (g_bumper_pressed) { ... }
}
```

ノードが複雑になると：
- どこで変数が変わるか追いにくい
- 複数のコールバックが同じ変数を変更してバグの原因になる

**クラスにまとめると**、状態をメンバ変数として管理でき、コールバックも自然にメンバ関数として書けます。

---

## クラス設計の方針

```
class BounceRobot
{
    メンバ変数:
        ros::NodeHandle, ros::Publisher, ros::Subscriber, ros::Timer
        State  state_          ← ロボットの現在状態
        ros::Time state_start_ ← その状態になった時刻
        bool bumper_pressed_   ← グローバル変数の代わり

    コンストラクタ:
        Publisher・Subscriber・Timer を初期化

    public:
        run() → ros::spin()

    private:
        bumperCallback()  ← メンバ関数としてのコールバック
        timerCallback()   ← 定期処理
        changeState()     ← 状態遷移の補助関数
}
```

---

## サンプル：クラスベースの往復ロボット

バンパーを検知して後退し、前進を再開する往復動作をクラスで実装します。

`~/catkin_ws/src/kobuki_tutorial/src/bounce_robot.cpp`

```cpp
#include <ros/ros.h>
#include <geometry_msgs/Twist.h>
#include <kobuki_msgs/BumperEvent.h>

class BounceRobot
{
public:
    BounceRobot() : state_(MOVING), bumper_pressed_(false)
    {
        cmd_pub_ = nh_.advertise<geometry_msgs::Twist>(
            "/mobile_base/commands/velocity", 10);

        bumper_sub_ = nh_.subscribe(
            "/mobile_base/events/bumper", 10,
            &BounceRobot::bumperCallback, this);

        // 100 ms ごとに timerCallback を呼ぶ
        timer_ = nh_.createTimer(
            ros::Duration(0.1), &BounceRobot::timerCallback, this);

        state_start_ = ros::Time::now();
        ROS_INFO("BounceRobot 起動");
    }

    void run()
    {
        ros::spin();
    }

private:
    enum State { MOVING, BACKING_UP };

    void bumperCallback(const kobuki_msgs::BumperEvent::ConstPtr& msg)
    {
        // 前進中にバンパーが押されたときだけ反応する
        if (msg->state == kobuki_msgs::BumperEvent::PRESSED
            && state_ == MOVING)
        {
            bumper_pressed_ = true;
            ROS_WARN("バンパー接触 → 後退");
        }
    }

    void timerCallback(const ros::TimerEvent&)
    {
        geometry_msgs::Twist cmd;
        double elapsed = (ros::Time::now() - state_start_).toSec();

        switch (state_)
        {
            case MOVING:
                cmd.linear.x = 0.1;
                if (bumper_pressed_)
                {
                    bumper_pressed_ = false;
                    changeState(BACKING_UP);
                }
                break;

            case BACKING_UP:
                cmd.linear.x = -0.1;
                if (elapsed > 1.0)
                {
                    ROS_INFO("後退完了 → 前進");
                    changeState(MOVING);
                }
                break;
        }

        cmd_pub_.publish(cmd);
    }

    void changeState(State next)
    {
        state_       = next;
        state_start_ = ros::Time::now();
    }

    ros::NodeHandle nh_;
    ros::Publisher  cmd_pub_;
    ros::Subscriber bumper_sub_;
    ros::Timer      timer_;

    State     state_;
    ros::Time state_start_;
    bool      bumper_pressed_;
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "bounce_robot");
    BounceRobot robot;
    robot.run();
    return 0;
}
```

### CMakeLists.txt に追加

```cmake
add_executable(bounce_robot src/bounce_robot.cpp)
target_link_libraries(bounce_robot ${catkin_LIBRARIES})
```

---

## LED とサウンドを使う

Kobuki は LED（2 個）とブザーを搭載しています。**移動不要**で Kobuki のトピック通信を練習できます。

### LED の制御

```bash
rosmsg show kobuki_msgs/Led
```
```
uint8 BLACK=0
uint8 GREEN=1
uint8 ORANGE=2
uint8 RED=3
uint8 value
```

```bash
# LED1 を赤に光らせる（コマンドラインから）
rostopic pub -1 /mobile_base/commands/led1 kobuki_msgs/Led "value: 3"

# LED1 を消す
rostopic pub -1 /mobile_base/commands/led1 kobuki_msgs/Led "value: 0"
```

### サウンドの制御

```bash
rosmsg show kobuki_msgs/Sound
```
```
uint8 ON=0
uint8 OFF=1
uint8 RECHARGE=2
uint8 BUTTON=3
uint8 ERROR=4
uint8 CLEANINGSTART=5
uint8 CLEANINGEND=6
uint8 value
```

```bash
# ボタン音を再生
rostopic pub -1 /mobile_base/commands/sound kobuki_msgs/Sound "value: 3"
```

---

## 演習 6：バンパーに応じて LED を光らせる（移動なし）

### 課題

**ロボットは動かさずに**、バンパーの状態を LED で表示するプログラムをクラスで書いてください。

**仕様：**

| 状態 | LED1 | LED2 |
|------|------|------|
| 何も当たっていない（初期） | 緑 | 緑 |
| 左バンパー接触 | 赤 | 緑 |
| 中央バンパー接触 | 赤 | 赤 |
| 右バンパー接触 | 緑 | 赤 |
| バンパーが離れた | 緑 | 緑 |

接触時はサウンドも鳴らしてください（`kobuki_msgs::Sound::BUTTON`）。

### ヒント

- `kobuki_msgs::Led` と `kobuki_msgs::Sound` を Publisher として持つ
- `bumperCallback` の中で条件分岐し、対応する LED の値を publish する
- 初期化時（コンストラクタ）に LED を緑にしておく

<details>
<summary>サンプルコード（考えてから開くこと！）</summary>

```cpp
#include <ros/ros.h>
#include <kobuki_msgs/BumperEvent.h>
#include <kobuki_msgs/Led.h>
#include <kobuki_msgs/Sound.h>

class LedDisplay
{
public:
    LedDisplay()
    {
        led1_pub_  = nh_.advertise<kobuki_msgs::Led>(
            "/mobile_base/commands/led1",  10);
        led2_pub_  = nh_.advertise<kobuki_msgs::Led>(
            "/mobile_base/commands/led2",  10);
        sound_pub_ = nh_.advertise<kobuki_msgs::Sound>(
            "/mobile_base/commands/sound", 10);
        bumper_sub_ = nh_.subscribe(
            "/mobile_base/events/bumper", 10,
            &LedDisplay::bumperCallback, this);

        // 起動時は両方緑
        ros::Duration(1.0).sleep();
        setLeds(kobuki_msgs::Led::GREEN, kobuki_msgs::Led::GREEN);
        ROS_INFO("バンパーを手で押してみてください");
    }

    void run() { ros::spin(); }

private:
    void bumperCallback(const kobuki_msgs::BumperEvent::ConstPtr& msg)
    {
        if (msg->state == kobuki_msgs::BumperEvent::PRESSED)
        {
            switch (msg->bumper)
            {
                case kobuki_msgs::BumperEvent::LEFT:
                    setLeds(kobuki_msgs::Led::RED, kobuki_msgs::Led::GREEN);
                    ROS_INFO("左バンパー接触");
                    break;
                case kobuki_msgs::BumperEvent::CENTER:
                    setLeds(kobuki_msgs::Led::RED, kobuki_msgs::Led::RED);
                    ROS_INFO("中央バンパー接触");
                    break;
                case kobuki_msgs::BumperEvent::RIGHT:
                    setLeds(kobuki_msgs::Led::GREEN, kobuki_msgs::Led::RED);
                    ROS_INFO("右バンパー接触");
                    break;
            }
            playSound(kobuki_msgs::Sound::BUTTON);
        }
        else  // RELEASED
        {
            setLeds(kobuki_msgs::Led::GREEN, kobuki_msgs::Led::GREEN);
            ROS_INFO("バンパー解放");
        }
    }

    void setLeds(uint8_t led1_val, uint8_t led2_val)
    {
        kobuki_msgs::Led led1, led2;
        led1.value = led1_val;
        led2.value = led2_val;
        led1_pub_.publish(led1);
        led2_pub_.publish(led2);
    }

    void playSound(uint8_t sound_val)
    {
        kobuki_msgs::Sound sound;
        sound.value = sound_val;
        sound_pub_.publish(sound);
    }

    ros::NodeHandle nh_;
    ros::Publisher  led1_pub_, led2_pub_, sound_pub_;
    ros::Subscriber bumper_sub_;
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "ex6_led_display");
    LedDisplay node;
    node.run();
    return 0;
}
```

</details>

---

## 演習 7：バンパーカウンターロボット

### 課題

前後に往復しながら、バンパーに当たった **累計回数** をカウントし、一定回数に達したら自動停止するクラスを設計・実装してください。

**仕様：**
- 動作：前進 → バンパー接触 → 後退（1 秒）→ 前進…を繰り返す
- バンパーに当たるたびに LED と ROS_INFO で回数を表示する
  - 1〜2 回目：LED 緑
  - 3〜4 回目：LED オレンジ
  - 5 回目：LED 赤＋サウンド「ERROR」→ 停止
- 停止後は `ros::shutdown()` を呼ぶ

**設計メモ：**

```
メンバ変数:
  int bounce_count_ = 0;
  // State, state_start_, Publisher, Subscriber, Timer ...

timerCallback:
  演習 4 の状態機械をベースに、状態が MOVING → BACKING_UP に遷移する
  たびに bounce_count_++ して LED を更新する

updateLed():
  bounce_count_ に応じて LED の色を変える補助関数
```

> この演習には答えを示しません。これまでのサンプルコードを参考にして取り組んでください。

---

## CMakeLists.txt（この章全体分）

```cmake
add_executable(bounce_robot    src/bounce_robot.cpp)
target_link_libraries(bounce_robot    ${catkin_LIBRARIES})

add_executable(ex6_led_display src/ex6_led_display.cpp)
target_link_libraries(ex6_led_display ${catkin_LIBRARIES})

add_executable(ex7_bounce_counter src/ex7_bounce_counter.cpp)
target_link_libraries(ex7_bounce_counter ${catkin_LIBRARIES})
```

---

## Kobuki チュートリアル まとめ

| 章 | 内容 | 主な演習 |
|---|------|---------|
| 12 | 速度コマンド・往復動作 | 時間指定前進・往復繰り返し |
| 13 | バンパー・崖・オドメトリ | 接触停止・接触後退・距離移動 |
| 14 | クラス設計・LED/サウンド | 往復クラス・LED 表示・カウンター |

### 次に挑戦できること

- **パラメータで速度・距離を外から設定する**（7 章の内容を応用）
- **launch ファイルで Kobuki 起動から演習ノードまでまとめる**（8 章の応用）
- **カスタムメッセージでバンパー回数をトピック配信する**（6 章の応用）
- **ナビゲーション**：`move_base` パッケージで地図ベースの自律移動
