# 1章: ROS とは

## ROS の概要

**ROS（Robot Operating System）** は，ロボットソフトウェア開発のための「ミドルウェア」です．
OS という名前がついていますが，Windows や Linux のような OS ではなく，Linux の上で動作するソフトウェア基盤です．

### なぜ ROS を使うのか？

ロボットのソフトウェアには，カメラ・センサー・モーターなど，異なるハードウェアを連携させる必要があります．これを一から作ると非常に大変です．ROS を使うと：

- **部品化**：センサー処理・運動制御・認識などを独立したプログラムとして書ける
- **通信の仕組みが標準化**：部品間のデータ受け渡しが簡単に書ける
- **豊富なライブラリ**：カメラ・LiDAR・ロボットアームなどの既製部品が多数ある
- **可視化ツール**：`rviz`・`rqt` などのデバッグツールがある

### このチュートリアルで使う ROS バージョン

**ROS1 Noetic**（Ubuntu 20.04 向け）を使います．

---

## 基本概念

ROS を理解するうえで最初に押さえるべき 5 つの概念を説明します．

### ノード（Node）

**ノード = 1 つのプログラム（プロセス）** のことです．

ROS では 1 つの大きなプログラムを書くのではなく，小さなプログラム（ノード）をたくさん起動して，それらを連携させます．

```
例：
  camera_node      ← カメラ画像を取得するプログラム
  detection_node   ← 画像から物体を検出するプログラム
  motor_node       ← モーターを制御するプログラム
```

それぞれのノードは独立して動作し，お互いにデータを送り合います．

### トピック（Topic）

**トピック = データの流れ道（チャンネル）** のことです．

ノード同士はトピックを通じてデータをやり取りします．

```mermaid
graph LR
    camera_node["camera_node"] -->|"publish"| topic(["/camera/image<br/>（Topic）"])
    topic -->|"subscribe"| detection_node["detection_node"]
```

- データを送る側を **Publisher（パブリッシャー）**
- データを受け取る側を **Subscriber（サブスクライバー）**

と呼びます．トピックを購読（subscribe）している全ノードにデータが届きます（放送のようなイメージ）．

### サービス（Service）

**サービス = 要求・応答型の通信** のことです．

トピックが「送りっぱなし（応答なし）」なのに対し，サービスは **1回の要求に対して必ず1回の応答が返ってくる** 通信です．

```mermaid
graph LR
    client["client_node"] -->|"① Request"| server["server_node"]
    server -->|"② Response"| client
```

1. client が Request を送る
2. server が処理して Response を **1回だけ** 返す

例：「今の位置を教えて」→「現在地は (x=1.0, y=2.0) です」

### アクション（Action）

**アクション = 長時間処理向けの通信** のことです．

サービスが「1回の要求・応答で完結」するのに対し，アクションは **処理中に何度もフィードバックが届き，途中でキャンセルもできる** 通信です．

```mermaid
graph LR
    client["client_node"] -->|"① Goal"| server["server_node"]
    server -->|"② Feedback（繰り返し）"| client
    server -->|"③ Result"| client
    client -.->|"Cancel（任意）"| server
```

1. client が Goal を送る
2. server が処理しながら Feedback を **繰り返し** 送る
3. 完了したら server が Result を返す
4. 途中でキャンセルしたい場合は Cancel を送れる（任意）

例：「目標地点まで移動して」→「30%…50%…80%…」→「到着しました」

詳しくは [6章: アクション通信](06_actionlib.md) で学びます．

### メッセージ（Message）

**メッセージ = 各通信方式でやり取りするデータの型** のことです．

通信方式ごとに，型を定義するファイル形式が異なります．

| 通信方式 | ファイル形式 | 定義内容 |
|---------|------------|---------|
| トピック | `.msg` | 送受信するデータの構造 |
| サービス | `.srv` | Request 型 + Response 型 |
| アクション | `.action` | Goal 型 + Feedback 型 + Result 型 |

トピックで使う `.msg` の標準型の例：

```
std_msgs/String     ← 文字列
std_msgs/Int32      ← 32bit 整数
std_msgs/Float64    ← 64bit 浮動小数点
sensor_msgs/Image   ← 画像
geometry_msgs/Twist ← 速度コマンド（ロボット移動に使う）
```

### パラメータ（Parameter）

**パラメータ = プログラムの設定値** のことです．

ソースコードを書き直さずに，起動時に値を変えられる仕組みです．

```
例：
  センサーのサンプリング周波数
  ロボットの最大速度
  閾値・ゲイン値 など
```

---

## ROS の全体像

```mermaid
graph BT
    rosmaster["rosmaster（roscore）<br/>命名・登録サービス"]
    NodeA["NodeA"]
    subgraph direct["ノード間の直接通信（rosmaster を経由しない）"]
        NodeB["NodeB"]
        topic(["/data"])
        NodeC["NodeC"]
    end
    NodeA -.->|"登録"| rosmaster
    NodeB -.->|"登録"| rosmaster
    NodeC -.->|"登録"| rosmaster
    NodeB -->|"publish"| topic
    topic -->|"subscribe"| NodeC
```

`roscore` はすべてのノードに対して**命名・登録サービス**（ネームサービス）を提供します．ノードが起動すると roscore に自分の名前とアドレスを登録し，他のノードのアドレスを問い合わせます．実際のデータはノード同士が直接やり取りするため，roscore はデータの仲介は行いません．ROS を使うときは必ず最初に `roscore` を起動します．

---

## よく使うコマンドの一覧（参考）

実際の使い方は各章で説明します．ここでは「こういうコマンドがある」と把握しておくだけで大丈夫です．

| コマンド | 用途 |
|---------|------|
| `roscore` | ROS マスターの起動 |
| `rosrun <pkg> <node>` | ノードを起動する |
| `roslaunch <pkg> <file.launch>` | launch ファイルで複数ノードを起動 |
| `rosnode list` | 起動中のノード一覧 |
| `rostopic list` | 現在のトピック一覧 |
| `rostopic echo /topic_name` | トピックの内容をリアルタイム表示 |
| `rosmsg show <型名>` | メッセージ型の定義を確認 |
| `rosservice list` | 現在のサービス一覧 |
| `rosparam list` | 現在のパラメータ一覧 |

---

次の章では，実際に ROS をインストールして使える状態にします．

[→ 2章: 環境構築](02_setup.md)
