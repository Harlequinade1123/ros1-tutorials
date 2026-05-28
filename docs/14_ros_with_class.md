# 14章: クラスを使った ROS プログラミング

4 章で書いた talker・listener をクラスで書き直します．最初は「なぜクラスで書くのか？」という動機から入ります．

> クラスの基本（コンストラクタ・メンバ変数・public/private など）は **[付録C: C++ クラス入門](appendix_cpp_class.md)** で詳しく説明しています．クラスに不安がある場合は先に付録Cを読んでください．

---

## なぜクラスで書くのか

4 章の talker をもう一度見てみましょう．

```cpp
// 4章の talker（抜粋）
int count = 0;   // ← グローバル変数になりがち

void someOtherFunction()
{
    count++;   // どこからでも変更できてしまう（危険）
}
```

ノードが複雑になると：
- Publisher・Subscriber・タイマーなど多くのオブジェクトが増える
- コールバック関数が増えてきたとき，必要なデータをどう渡すか困る
- グローバル変数が増えてコードが追いにくくなる

**クラスを使うと：**
- 関連するデータと処理をひとまとめにできる
- コールバックからメンバ変数に自然にアクセスできる
- ノードを複数インスタンス化することもできる

---

## クラス版 talker（class_talker.cpp）

`~/catkin_ws/src/ros_tutorial/src/class_talker.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <std_msgs/String.h>
#include <sstream>

class Talker
{
public:
    Talker() : count_(0)
    {
        // コンストラクタで Publisher を初期化
        pub_ = nh_.advertise<std_msgs::String>("chatter", 10);
    }

    void run()
    {
        ros::Rate rate(10);
        while (ros::ok())
        {
            publish();
            ros::spinOnce();
            rate.sleep();
        }
    }

private:
    void publish()
    {
        std_msgs::String msg;
        std::stringstream ss;
        ss << "hello world " << count_++;
        msg.data = ss.str();

        ROS_INFO("%s", msg.data.c_str());
        pub_.publish(msg);
    }

    ros::NodeHandle nh_;   // ROS との通信窓口
    ros::Publisher  pub_;  // Publisher オブジェクト
    int             count_;
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "talker");

    Talker talker;   // オブジェクトを生成（コンストラクタが呼ばれる）
    talker.run();    // ループ開始

    return 0;
}
```

### ポイント

- `ros::NodeHandle nh_` をメンバ変数として持つ
- `ros::Publisher pub_` もメンバ変数に
- `count_` もメンバ変数なのでグローバル変数不要
- `main()` がとてもシンプルになる

> **重要**: `ros::NodeHandle` はメンバ変数として宣言した場合，`ros::init()` が呼ばれた**後**に初期化される必要があります．`main()` で `ros::init()` を呼んでからオブジェクトを生成することで，これが保証されます．

---

## クラス版 listener（class_listener.cpp）

`~/catkin_ws/src/ros_tutorial/src/class_listener.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <std_msgs/String.h>

class Listener
{
public:
    Listener()
    {
        // メンバ関数をコールバックとして登録する書き方
        sub_ = nh_.subscribe("chatter", 10,
                              &Listener::chatterCallback, this);
        //                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^  ^^^^
        //                    メンバ関数のアドレス          このオブジェクト自身
    }

    void run()
    {
        ros::spin();
    }

private:
    void chatterCallback(const std_msgs::String::ConstPtr& msg)
    {
        ROS_INFO("受信: [%s]", msg->data.c_str());
        receive_count_++;
        ROS_INFO("受信回数: %d", receive_count_);
    }

    ros::NodeHandle nh_;
    ros::Subscriber sub_;
    int receive_count_ = 0;
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "listener");

    Listener listener;
    listener.run();

    return 0;
}
```

### コールバック登録の書き方

4 章ではグローバル関数をコールバックに登録していました：

```cpp
// 4章の書き方（グローバル関数）
ros::Subscriber sub = nh.subscribe("chatter", 10, chatterCallback);
```

クラスのメンバ関数をコールバックにするには少し違う書き方が必要です：

```cpp
// クラスのメンバ関数をコールバックにする書き方
sub_ = nh_.subscribe("chatter", 10, &Listener::chatterCallback, this);
```

| 引数 | 意味 |
|------|------|
| `"chatter"` | トピック名 |
| `10` | キューサイズ |
| `&Listener::chatterCallback` | 「Listener クラスの chatterCallback 関数のアドレス」 |
| `this` | 「今のオブジェクト自身」を渡す（どのオブジェクトのメンバ関数か ROS に教える） |

`this` とメンバ関数ポインタ（`&ClassName::function`）の詳細は **[付録C](appendix_cpp_class.md#ros-で登場するクラス関連の構文)** を参照してください．

---

## CMakeLists.txt に追加

```cmake
add_executable(class_talker src/class_talker.cpp)
target_link_libraries(class_talker ${catkin_LIBRARIES})

add_executable(class_listener src/class_listener.cpp)
target_link_libraries(class_listener ${catkin_LIBRARIES})
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
rosrun ros_tutorial class_talker
```

**ターミナル 3：**
```bash
rosrun ros_tutorial class_listener
```

---

## タイマーを使った書き方（推奨パターン）

`while (ros::ok()) { ... }` のループではなく，**ROS のタイマー機能** を使うとよりスマートに書けます．
この方法では `ros::spin()` 1 つだけで制御でき，コールバックと同じ仕組みで定期処理ができます．

`~/catkin_ws/src/ros_tutorial/src/timer_talker.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <std_msgs/String.h>
#include <sstream>

class TimerTalker
{
public:
    TimerTalker() : count_(0)
    {
        pub_ = nh_.advertise<std_msgs::String>("chatter", 10);

        // 0.1 秒（10Hz）ごとに timerCallback を呼ぶタイマーを作成
        timer_ = nh_.createTimer(ros::Duration(0.1),
                                  &TimerTalker::timerCallback, this);
    }

    void run()
    {
        ros::spin();   // while ループ不要！
    }

private:
    // タイマーが発火するたびに呼ばれる
    void timerCallback(const ros::TimerEvent& event)
    {
        std_msgs::String msg;
        std::stringstream ss;
        ss << "hello world " << count_++;
        msg.data = ss.str();

        ROS_INFO("%s", msg.data.c_str());
        pub_.publish(msg);
    }

    ros::NodeHandle nh_;
    ros::Publisher  pub_;
    ros::Timer      timer_;
    int             count_;
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "timer_talker");
    TimerTalker node;
    node.run();
    return 0;
}
```

```cmake
add_executable(timer_talker src/timer_talker.cpp)
target_link_libraries(timer_talker ${catkin_LIBRARIES})
```

### タイマー版の利点

| | while ループ版 | タイマー版 |
|-|----------------|-----------|
| メインループ | `while (ros::ok())` | `ros::spin()` |
| 周期処理 | `ros::Rate` + `rate.sleep()` | `createTimer()` |
| コールバックと共存 | やや複雑 | 自然に共存できる |
| 複数の周期処理 | ループが複雑になる | タイマーを複数作れば OK |

複数の処理を異なる周期で動かしたいとき（例：センサー読み取りは 100Hz，ステータス表示は 1Hz）はタイマー版が特に便利です．

```cpp
// 100Hz でセンサー読み取り
timer1_ = nh_.createTimer(ros::Duration(0.01), &MyNode::sensorCallback, this);

// 1Hz でステータス表示
timer2_ = nh_.createTimer(ros::Duration(1.0),  &MyNode::statusCallback,  this);
```

---

## Publisher と Subscriber を両方持つクラス

実際のノードでは，Publisher と Subscriber を両方持つことが多いです．
（例：センサーデータを受け取って処理し，結果を送信する）

```cpp
#include <ros/ros.h>
#include <std_msgs/Float64.h>

class FilterNode
{
public:
    FilterNode()
    {
        sub_ = nh_.subscribe("raw_value",      10,
                              &FilterNode::inputCallback, this);
        pub_ = nh_.advertise<std_msgs::Float64>("filtered_value", 10);
    }

    void run()
    {
        ros::spin();
    }

private:
    void inputCallback(const std_msgs::Float64::ConstPtr& msg)
    {
        // 単純な移動平均フィルタ
        sum_ += msg->data;
        count_++;

        std_msgs::Float64 out;
        out.data = sum_ / count_;

        pub_.publish(out);
        ROS_INFO("入力: %.2f  出力（平均）: %.2f", msg->data, out.data);
    }

    ros::NodeHandle nh_;
    ros::Subscriber sub_;
    ros::Publisher  pub_;

    double sum_   = 0.0;
    int    count_ = 0;
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "filter_node");
    FilterNode node;
    node.run();
    return 0;
}
```

このパターン（**受け取って → 処理して → 送る**）は ROS で最もよく使われる構造です．

---

## クラスを使った ROS ノードのまとめ

```
クラスを使った ROS ノードの典型的な構造：

class MyNode
{
public:
    MyNode()
    {
        // Publisher・Subscriber・Timer の初期化
    }

    void run()
    {
        ros::spin();  または  while ループ
    }

private:
    // コールバック関数たち

    ros::NodeHandle nh_;
    ros::Publisher  pub_;
    ros::Subscriber sub_;
    ros::Timer      timer_;
    // その他のメンバ変数
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "my_node");
    MyNode node;
    node.run();
    return 0;
}
```

この構造を基本テンプレートとして覚えておくと，どんなノードを書くときも迷いにくくなります．

---

## このチュートリアルのまとめ

| 章 | 学んだこと |
|---|-----------|
| 1 | ROS の概念（ノード・トピック・メッセージ・サービス・パラメータ） |
| 2 | 環境構築（インストール・ワークスペース） |
| 3 | パッケージ作成・CMakeLists.txt の基本 |
| 4 | Publisher / Subscriber の実装 |
| 5 | サービス（リクエスト・レスポンス型通信） |
| 6 | アクション通信（長時間処理・フィードバック・キャンセル） |
| 7 | カスタムメッセージ（.msg / .srv ファイル） |
| 8 | パラメータの設定と取得 |
| 9 | launch ファイルによる複数ノードの起動 |
| 10 | RViz（データの 3D 可視化） |
| 11 | rosbag（トピックの記録・再生） |
| 12 | tf / tf2（座標変換） |
| 13 | C++ クラスの概要（詳細は付録C） |
| 14 | ROS ノードのクラス化 |

---

## サービスをクラスで書く

5 章では**グローバル関数**をサービスのコールバックに登録しました．クラスを使うと，サービスサーバーをメンバ変数として持てるため，複数のサービスや Publisher との組み合わせが整理しやすくなります．

`~/catkin_ws/src/ros_tutorial/src/add_two_ints_server_class.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <ros_tutorial/AddTwoInts.h>

class AddTwoIntsServer
{
public:
    AddTwoIntsServer()
    {
        service_ = nh_.advertiseService("add_two_ints",
                                         &AddTwoIntsServer::add, this);
        ROS_INFO("add_two_ints サービスの準備完了");
    }

    void run() { ros::spin(); }

private:
    bool add(ros_tutorial::AddTwoInts::Request  &req,
             ros_tutorial::AddTwoInts::Response &res)
    {
        res.sum = req.a + req.b;
        ROS_INFO("a=%ld, b=%ld → sum=%ld", req.a, req.b, res.sum);
        return true;
    }

    ros::NodeHandle    nh_;
    ros::ServiceServer service_;
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "add_two_ints_server");
    AddTwoIntsServer node;
    node.run();
    return 0;
}
```

```cmake
add_executable(add_two_ints_server_class src/add_two_ints_server_class.cpp)
target_link_libraries(add_two_ints_server_class ${catkin_LIBRARIES})
add_dependencies(add_two_ints_server_class ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
```

5 章で作ったクライアント（`add_two_ints_client`）はそのまま使えます．

---

## アクションをクラスで書く

6 章では `boost::bind` でコールバックにサーバーのポインタを渡しました．クラスを使うと，サーバーをメンバ変数として保持できるため，コードの見通しが良くなります．

`~/catkin_ws/src/ros_tutorial/src/count_down_server_class.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <actionlib/server/simple_action_server.h>
#include <ros_tutorial/CountDownAction.h>

class CountDownServer
{
public:
    CountDownServer()
        : server_(nh_, "count_down",
                  boost::bind(&CountDownServer::executeCallback, this, _1),
                  false)
    {
        server_.start();
        ROS_INFO("CountDown アクションサーバー準備完了（クラス版）");
    }

    void run() { ros::spin(); }

private:
    void executeCallback(const ros_tutorial::CountDownGoalConstPtr &goal)
    {
        ros::Rate rate(1.0);
        ros_tutorial::CountDownFeedback feedback;
        ros_tutorial::CountDownResult   result;

        ROS_INFO("ゴール受信: target = %d", goal->target);

        for (int i = goal->target; i >= 0; --i)
        {
            if (server_.isPreemptRequested() || !ros::ok())
            {
                ROS_INFO("キャンセルされました");
                server_.setPreempted();
                return;
            }

            feedback.remaining = i;
            server_.publishFeedback(feedback);
            ROS_INFO("残り: %d", i);

            rate.sleep();
        }

        result.message = "カウントダウン完了！";
        server_.setSucceeded(result);
        ROS_INFO("完了");
    }

    ros::NodeHandle nh_;
    actionlib::SimpleActionServer<ros_tutorial::CountDownAction> server_;
};

int main(int argc, char **argv)
{
    ros::init(argc, argv, "count_down_server");
    CountDownServer server;
    server.run();
    return 0;
}
```

```cmake
add_executable(count_down_server_class src/count_down_server_class.cpp)
target_link_libraries(count_down_server_class ${catkin_LIBRARIES})
add_dependencies(count_down_server_class ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
```

クライアントは 6 章の `count_down_client` をそのまま使えます．

---

## 次のステップ

クラスを使ったプログラミングの基礎は以上です．次は実機ロボット（Kobuki）を使った演習に進みます．

発展的なトピック：
- **dynamic_reconfigure**：実行中のパラメータ動的変更
- **ROS2 への移行**：概念は共通，コードの書き方が変わる

[→ 15章: Kobuki 基礎](15_kobuki_basics.md)
