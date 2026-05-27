# 3章: 最初のパッケージを作る

## パッケージとは

ROS では、ひとまとまりのプログラムを **パッケージ** という単位で管理します。  
Visual Studio のプロジェクトに相当するものだと思ってください。

パッケージには以下が含まれます：

- C++ のソースファイル（`.cpp`）
- ビルド設定ファイル（`CMakeLists.txt`）
- パッケージ情報ファイル（`package.xml`）
- メッセージ・サービス定義ファイル（後の章で説明）

---

## パッケージを作成する

### 作業ディレクトリへ移動

```bash
cd ~/catkin_ws/src
```

### パッケージ作成コマンド

```bash
catkin create pkg ros_tutorial --catkin-deps roscpp std_msgs
```

**コマンドの意味：**
- `catkin create pkg` ：`catkin_tools` のパッケージ作成コマンド
- `ros_tutorial` ：作成するパッケージの名前
- `--catkin-deps roscpp std_msgs` ：このパッケージが使用する catkin 依存パッケージ
  - `roscpp`：C++ で ROS を使うために必要
  - `std_msgs`：標準的なメッセージ型（文字列・数値など）を使うために必要

> **旧コマンドについて**: `catkin_create_pkg` という古いコマンドも同じ機能を持ちます。
> ```bash
> catkin_create_pkg ros_tutorial roscpp std_msgs
> ```
> `catkin_create_pkg` は `catkin` コアパッケージに含まれており、依存パッケージをフラグなしで並べて指定します。`catkin create pkg` は `catkin_tools` が提供する新しい書き方です。どちらも生成されるファイルは同じです。

実行すると `src/ros_tutorial/` フォルダが作成されます。

```
~/catkin_ws/src/ros_tutorial/
├── CMakeLists.txt    ← ビルド設定
├── package.xml       ← パッケージ情報
└── src/              ← C++ ソースファイルを置く場所（最初は空）
```

---

## 各ファイルの役割

### CMakeLists.txt

**「どのファイルをどうビルドするか」** を記述するファイルです。

Visual Studio では GUI でプロジェクト設定をしていましたが、ROS ではこのテキストファイルに書きます。

最初は難しく見えますが、実際に変更するのは限られた箇所だけです。

デフォルトで生成された内容を確認してみましょう（長いのでコメント行 `#` は省略して重要な部分を示します）：

```cmake
cmake_minimum_required(VERSION 3.0.2)
project(ros_tutorial)           # パッケージ名

# 依存パッケージの検索
find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
)

catkin_package()

include_directories(
  ${catkin_INCLUDE_DIRS}        # ROS のヘッダファイルの場所
)

# ここに実行ファイルの設定を追加していく（後述）
```

### package.xml

**パッケージのメタ情報（名前・バージョン・依存関係など）** を記述するファイルです。

```xml
<?xml version="1.0"?>
<package format="2">
  <name>ros_tutorial</name>
  <version>0.0.0</version>
  <description>ROS Tutorial Package</description>
  <maintainer email="your@email.com">Your Name</maintainer>
  <license>MIT</license>

  <buildtool_depend>catkin</buildtool_depend>

  <!-- ビルド時に必要なパッケージ -->
  <build_depend>roscpp</build_depend>
  <build_depend>std_msgs</build_depend>

  <!-- 実行時に必要なパッケージ -->
  <exec_depend>roscpp</exec_depend>
  <exec_depend>std_msgs</exec_depend>
</package>
```

> **今は内容を全部理解しなくて大丈夫です。** 新しい機能を使うたびに、変更が必要な箇所だけを説明します。

---

## ビルドしてみる

```bash
cd ~/catkin_ws
catkin build
```

エラーなく完了すれば成功です。今はまだソースファイルが何もないので、これだけです。

---

## 試しにソースファイルを置いてみる

動作確認のために、簡単な C++ ファイルを作ってビルドしてみましょう。

### ソースファイルの作成

```bash
nano ~/catkin_ws/src/ros_tutorial/src/hello.cpp
```

以下の内容を入力して保存（`Ctrl+O` → `Enter` → `Ctrl+X`）：

```cpp
#include <ros/ros.h>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "hello_node");
    ros::NodeHandle nh;

    ROS_INFO("Hello, ROS!");   // ROS 版の printf / cout

    return 0;
}
```

**`ROS_INFO`** は ROS 用のログ出力関数です。`printf` や `std::cout` の代わりに使います。タイムスタンプやノード名も自動でつきます。

### CMakeLists.txt に実行ファイルの設定を追加

`~/catkin_ws/src/ros_tutorial/CMakeLists.txt` を開き、末尾に以下を追加します：

```bash
nano ~/catkin_ws/src/ros_tutorial/CMakeLists.txt
```

ファイルの一番下に追記：

```cmake
add_executable(hello_node src/hello.cpp)
target_link_libraries(hello_node ${catkin_LIBRARIES})
```

**意味：**
- `add_executable(hello_node src/hello.cpp)` → `src/hello.cpp` をコンパイルして `hello_node` という実行ファイルを作る
- `target_link_libraries(...)` → ROS のライブラリをリンクする（おまじないと思っておいてよい）

### ビルド

```bash
cd ~/catkin_ws
catkin build
```

### 実行

ターミナルを **2 つ** 使います。

**ターミナル 1（roscore の起動）：**
```bash
roscore
```

**ターミナル 2（ノードの実行）：**
```bash
rosrun ros_tutorial hello_node
```

出力例：
```
[ INFO] [1234567890.123]: Hello, ROS!
```

`rosrun <パッケージ名> <ノード名>` でノードを起動します。

---

## まとめ

パッケージを作る基本的な流れをまとめます。

```
1. catkin create pkg でパッケージ作成
2. src/ に .cpp ファイルを書く
3. CMakeLists.txt に add_executable / target_link_libraries を追加
4. catkin build でビルド
5. rosrun で実行
```

この流れは以降の章でも同じです。

---

[→ 4章: Publisher / Subscriber](04_publisher_subscriber.md)
