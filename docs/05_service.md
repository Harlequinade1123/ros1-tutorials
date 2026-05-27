# 5章: サービス

ROS の通信方式には、前章で学んだ**トピック**のほかに**サービス**があります。

---

## トピックとサービスの違い

| 比較項目 | トピック | サービス |
|---------|---------|---------|
| 通信モデル | 非同期（送りっぱなし）| 同期（要求を送り、応答を受け取る）|
| 向き | 単方向（Publisher → Subscriber）| 双方向（Client ⇄ Server）|
| 主な用途 | センサーデータの連続配信 | 計算依頼・設定変更・一時的な操作 |
| 応答 | なし | あり |

**トピック**は「毎秒 10 回センサーデータを送り続ける」ような継続的な配信に向いています。  
**サービス**は「2つの数値を足してその結果を返す」「ロボットの設定を変更する」といった、**一回の要求に対して一回の応答を得る**用途に向いています。

```
【トピック】
Publisher ──────────────────→ Subscriber（応答なし）

【サービス】
Client ──(Request)──→ Server
Client ←─(Response)── Server
```

---

## サービス定義ファイル（.srv）

サービスの型は `.srv` ファイルで定義します。`---` より上が**リクエスト**（クライアントが送る）、下が**レスポンス**（サーバーが返す）です。

パッケージ内に `srv/` フォルダを作ります。

```bash
mkdir -p ~/catkin_ws/src/ros_tutorial/srv
```

`~/catkin_ws/src/ros_tutorial/srv/AddTwoInts.srv` を作成：

```
int64 a
int64 b
---
int64 sum
```

---

## ビルド設定の変更

### CMakeLists.txt の変更

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

#### 2. `add_service_files` と `generate_messages` を追加（`catkin_package()` の前）

```cmake
add_service_files(
  FILES
  AddTwoInts.srv
)

generate_messages(
  DEPENDENCIES
  std_msgs
)
```

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

### package.xml の変更

`<buildtool_depend>catkin</buildtool_depend>` の下に追加：

```xml
<build_depend>message_generation</build_depend>
<exec_depend>message_runtime</exec_depend>
```

> **メモ**: `.msg` ファイルでカスタムメッセージを定義するときも同じ設定が必要です（詳しくは 6章）。

---

## サービスサーバーを実装する

`~/catkin_ws/src/ros_tutorial/src/add_two_ints_server.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <ros_tutorial/AddTwoInts.h>

// コールバック：クライアントからリクエストが届いたとき呼ばれる
bool add(ros_tutorial::AddTwoInts::Request  &req,
         ros_tutorial::AddTwoInts::Response &res)
{
    res.sum = req.a + req.b;
    ROS_INFO("リクエスト: a=%ld, b=%ld → レスポンス: sum=%ld",
             req.a, req.b, res.sum);
    return true;  // true で成功、false で失敗
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "add_two_ints_server");
    ros::NodeHandle nh;

    ros::ServiceServer service = nh.advertiseService("add_two_ints", add);
    ROS_INFO("add_two_ints サービスの準備完了");

    ros::spin();
    return 0;
}
```

### コードのポイント

| コード | 意味 |
|--------|------|
| `nh.advertiseService("add_two_ints", add)` | サービスを公開（`nh.advertise` のサービス版）|
| コールバック引数 `Request &req` | クライアントから受け取った値 |
| コールバック引数 `Response &res` | クライアントに返す値（ここに書き込む）|
| `return true` | サービス成功を示す |

---

## サービスクライアントを実装する

`~/catkin_ws/src/ros_tutorial/src/add_two_ints_client.cpp` を作成：

```cpp
#include <ros/ros.h>
#include <ros_tutorial/AddTwoInts.h>
#include <cstdlib>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "add_two_ints_client");

    if (argc != 3)
    {
        ROS_INFO("使い方: add_two_ints_client <a> <b>");
        return 1;
    }

    ros::NodeHandle nh;

    // サービスクライアントの作成
    ros::ServiceClient client =
        nh.serviceClient<ros_tutorial::AddTwoInts>("add_two_ints");

    // リクエストを組み立てて送信
    ros_tutorial::AddTwoInts srv;
    srv.request.a = std::atoll(argv[1]);
    srv.request.b = std::atoll(argv[2]);

    if (client.call(srv))
    {
        ROS_INFO("結果: %ld", srv.response.sum);
    }
    else
    {
        ROS_ERROR("サービスの呼び出しに失敗しました");
        return 1;
    }

    return 0;
}
```

### コードのポイント

| コード | 意味 |
|--------|------|
| `nh.serviceClient<型>("名前")` | サービスクライアントを作る（`nh.subscribe` のサービス版）|
| `client.call(srv)` | サービスを呼び出す（**同期**：レスポンスが返るまでブロック）|
| `srv.request.a = ...` | リクエストに値をセット |
| `srv.response.sum` | レスポンスの値を読む |

---

## CMakeLists.txt に実行ファイルの設定を追加

```cmake
add_executable(add_two_ints_server src/add_two_ints_server.cpp)
target_link_libraries(add_two_ints_server ${catkin_LIBRARIES})
add_dependencies(add_two_ints_server ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

add_executable(add_two_ints_client src/add_two_ints_client.cpp)
target_link_libraries(add_two_ints_client ${catkin_LIBRARIES})
add_dependencies(add_two_ints_client ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
```

---

## ビルドと実行

```bash
cd ~/catkin_ws
catkin build
```

**ターミナル 1：roscore**
```bash
roscore
```

**ターミナル 2：サーバーを起動**
```bash
rosrun ros_tutorial add_two_ints_server
```

出力：
```
[ INFO]: add_two_ints サービスの準備完了
```

**ターミナル 3：クライアントを実行**
```bash
rosrun ros_tutorial add_two_ints_client 3 7
```

クライアント側の出力：
```
[ INFO]: 結果: 10
```

サーバー側の出力：
```
[ INFO]: リクエスト: a=3, b=7 → レスポンス: sum=10
```

---

## rosservice コマンド

```bash
# 利用可能なサービス一覧
rosservice list

# サービスの詳細確認
rosservice info /add_two_ints

# サービスの型を確認
rosservice type /add_two_ints

# コマンドラインから直接呼び出す
rosservice call /add_two_ints "a: 10
b: 20"
```

サービス定義の確認：

```bash
rossrv show ros_tutorial/AddTwoInts
```

---

## トピックかサービスか

| 状況 | 使うべき通信 |
|------|------------|
| センサーデータを常に配信する | トピック |
| 速度指令を送り続ける | トピック |
| 計算して結果を返してほしい | サービス |
| 設定値を変更してほしい | サービス |
| 一度だけ特定の動作をしてほしい | サービス |

---

[→ 6章: カスタムメッセージ](06_custom_messages.md)
