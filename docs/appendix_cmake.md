# 付録: CMake とビルドの仕組み

Visual Studio を使っていると，「ビルド」ボタンを押すだけでプログラムが動くようになります．  
その裏で何が起きているのか，ROS を使い始めると無視できなくなります．  
この付録では **「コンパイルとは何か」→「CMake とは何か」→「ROS ではどう変わるか」** の順で説明します．

---

## 1. Visual Studio は何をやっていたのか

Visual Studio で C++ のコードを書いてビルドボタンを押すと，IDE が以下の作業を**自動で**行っていました．

```
main.cpp
    │
    ▼  コンパイル（翻訳）
main.obj          ← CPU が読める形式（機械語）の中間ファイル
    │
    ▼  リンク（結合）
main.exe          ← 実行ファイル
```

**コンパイル** とは，人間が読める C++ のコード（テキスト）を，CPU が直接実行できる機械語に翻訳する作業です．  
**リンク** とは，複数の中間ファイルや外部ライブラリを1つの実行ファイルにまとめる作業です．

Visual Studio はこれらを GUI の設定ファイル（`.vcxproj`）を使って管理していました．  
Linux や ROS の世界では，この「設定ファイル」に相当するのが **CMakeLists.txt** です．

---

## 2. CMake とは

**CMake** は「ビルドの設定を書くためのツール」です．  
コンパイラ（g++）に直接指示を出すのではなく，**「何をどうビルドするか」という設定を書いておくと，CMake が実際のビルドコマンドを生成してくれます．**

```
CMakeLists.txt（設定ファイル）
    │
    ▼  cmake コマンドで処理
Makefile（ビルド手順書）
    │
    ▼  make コマンドで実行
実行ファイル
```

### なぜ CMake が必要なのか

コンパイルを手作業でやると，たとえば 3 つのファイルがあった場合：

```bash
# 手動でコンパイルする場合（3ファイルの例）
g++ -c main.cpp -o main.o
g++ -c utils.cpp -o utils.o
g++ -c math.cpp -o math.o
g++ main.o utils.o math.o -o my_program
```

ファイルが増えるたびにコマンドが増え，管理が大変です．  
CMake を使えばこれを `CMakeLists.txt` に一度書くだけで，あとは自動化できます．

---

## 3. 通常の C++ プロジェクトを CMake でビルドする

ROS なしで，シンプルな Hello World を CMake でビルドする手順を体験しましょう．

### ファイルを用意する

```bash
mkdir ~/cmake_example
cd ~/cmake_example
```

`main.cpp` を作成します：

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, CMake!" << std::endl;
    return 0;
}
```

`CMakeLists.txt` を同じディレクトリに作成します：

```cmake
cmake_minimum_required(VERSION 3.10)   # CMake の最低バージョン
project(hello_cmake)                   # プロジェクト名

add_executable(hello main.cpp)         # 実行ファイル名 ← ソースファイル名
```

**この3行の意味：**

| 行 | 意味 |
|---|------|
| `cmake_minimum_required` | この設定を動かすのに必要な CMake の最低バージョン |
| `project(hello_cmake)` | プロジェクトに名前をつける（何でもよい） |
| `add_executable(hello main.cpp)` | `main.cpp` をコンパイルして `hello` という実行ファイルを作る |

### ビルドする

CMake では**ソースと中間ファイルを分けるため**，`build` ディレクトリを作って作業するのが慣習です．

```bash
mkdir build
cd build
cmake ..        # 1つ上のディレクトリの CMakeLists.txt を処理する
make            # 実際にコンパイルする
```

```
cmake_example/
├── main.cpp
├── CMakeLists.txt
└── build/          ← ここでビルド作業をする
    ├── Makefile    ← cmake が生成した手順書
    └── hello       ← make が生成した実行ファイル
```

### 実行する

```bash
./hello
# → Hello, CMake!
```

`./` は「現在のディレクトリにある実行ファイル」を明示する書き方です．  
Linux ではセキュリティ上の慣習として，カレントディレクトリが実行ファイルの検索パス（`PATH`）に含まれていません．そのため `hello` だけでは「どこにある hello か」がわからず，`./hello` と場所を明示する必要があります．

### ファイルが複数ある場合

`add_executable` にソースファイルを並べるだけです：

```cmake
add_executable(my_program
    main.cpp
    utils.cpp
    math.cpp
)
```

### 外部ライブラリを使う場合

たとえば数学ライブラリ（`sin`, `cos` などが入っている `libm`）を使いたい場合：

```cmake
add_executable(my_program main.cpp)
target_link_libraries(my_program m)    # "m" = 数学ライブラリ（libm）を結合する
```

`target_link_libraries` は「この実行ファイルを作るとき，このライブラリも一緒に結合してください」という命令です．  
ライブラリ名は `lib` と拡張子を除いた部分を書きます（`libm.so` → `m`，`libopencv_core.so` → `opencv_core`）．

---

## 4. Visual Studio との対応

| Visual Studio | CMake / Linux |
|---------------|--------------|
| `.vcxproj` ファイル | `CMakeLists.txt` |
| ソリューション | ワークスペース（複数の `CMakeLists.txt`）|
| 「ビルド」ボタン | `cmake .. && make` |
| Debug / Release 構成 | `cmake -DCMAKE_BUILD_TYPE=Debug ..` / `Release` |
| プロジェクトのプロパティ → インクルードディレクトリ | `include_directories(...)` |
| プロジェクトのプロパティ → 追加のライブラリ依存ファイル | `target_link_libraries(...)` |
| 出力ディレクトリ | `build/` 以下に自動生成 |

---

## 5. ROS（catkin）のビルドシステムとの違い

ROS では通常の CMake をそのまま使うのではなく，**catkin** という ROS 専用の拡張を使います．  
見た目は CMake そのものですが，いくつかの「お作法」が加わっています．

### 通常の CMake との比較

**通常の CMake（最小構成）：**

```cmake
cmake_minimum_required(VERSION 3.10)
project(hello_cmake)

add_executable(hello main.cpp)
target_link_libraries(hello m)
```

**catkin（ROS）の CMakeLists.txt：**

```cmake
cmake_minimum_required(VERSION 3.0.2)
project(ros_tutorial)

# ROS パッケージ（roscpp, std_msgs 等）を探して読み込む
find_package(catkin REQUIRED COMPONENTS
  roscpp
  std_msgs
)

# catkin のビルド設定を宣言する（他パッケージへのエクスポート情報）
catkin_package()

# ROS のヘッダファイルの場所を指定する
include_directories(
  ${catkin_INCLUDE_DIRS}
)

add_executable(talker src/talker.cpp)
target_link_libraries(talker ${catkin_LIBRARIES})   # ROS のライブラリをリンク
```

### 主な違い

| 項目 | 通常の CMake | catkin（ROS） |
|------|------------|--------------|
| ビルドコマンド | `cmake .. && make` | `catkin build` |
| 設定ファイル | `CMakeLists.txt` のみ | `CMakeLists.txt` + `package.xml` |
| ライブラリの指定 | 直接書く（例: `m`） | `${catkin_LIBRARIES}` に自動でまとまる |
| ヘッダの場所 | 直接書く | `${catkin_INCLUDE_DIRS}` に自動でまとまる |
| 実行ファイルの場所 | `build/` の中 | `~/catkin_ws/devel/lib/<パッケージ名>/` |
| 複数パッケージの管理 | 自前で整理 | ワークスペース（`catkin_ws`）で一元管理 |
| 依存パッケージの宣言 | 必要なければ不要 | `package.xml` に必ず書く |

### `package.xml` の役割

通常の CMake には対応するファイルがありません．  
catkin では **このパッケージが何に依存しているか** を `package.xml` に宣言します．  
これにより，`catkin build` が依存関係を解決して正しい順番でビルドしてくれます．

```xml
<!-- package.xml の依存宣言（抜粋） -->
<build_depend>roscpp</build_depend>   <!-- コンパイル時に必要 -->
<exec_depend>roscpp</exec_depend>     <!-- 実行時に必要 -->
```

| タグ | 意味 |
|------|------|
| `<build_depend>` | コンパイル時に必要なパッケージ（ヘッダファイルをインクルードする場合など） |
| `<exec_depend>` | 実行時に必要なパッケージ（共有ライブラリをリンクする場合など） |

`roscpp` のように両方で使うパッケージには，`<depend>roscpp</depend>` と1行で書く方法もあります（`build_depend` と `exec_depend` を兼ねます）．

CMakeLists.txt の `find_package` と package.xml の依存宣言は**必ずセットで書く**必要があります．片方だけ書いても動かない場合があります．

### `find_package` とは

```cmake
find_package(catkin REQUIRED COMPONENTS roscpp std_msgs)
```

「`roscpp` と `std_msgs` というパッケージを探して，そのヘッダファイルやライブラリの場所を `${catkin_INCLUDE_DIRS}` と `${catkin_LIBRARIES}` という変数に入れてください」という命令です．

`${変数名}` は CMake の変数展開の記法です．C++ の変数と同様に，`${catkin_INCLUDE_DIRS}` と書くとその変数に格納されたパス一覧に置き換えられます．

通常の CMake で外部ライブラリを使う場合も `find_package` を使います（例: `find_package(OpenCV REQUIRED)`）．ROS 特有の機能ではありません．

### `source devel/setup.bash` が必要な理由

通常の CMake でビルドした実行ファイルは，そのパス（例: `./build/hello`）を直接指定して実行できます．

catkin でビルドした ROS ノードは `~/catkin_ws/devel/lib/` 以下に置かれますが，`rosrun` がその場所を知るには **環境変数の設定** が必要です．

```bash
source ~/catkin_ws/devel/setup.bash
```

このコマンドが「`rosrun` がパッケージを見つけられる場所」などを環境変数に追記します．  
ターミナルを開くたびに必要な理由はこれです（`~/.bashrc` に書いておけば自動化できます）．

---

## 6. よくあるエラーと対処

### `catkin build` でエラーが出る

```
Could not find a package configuration file provided by "roscpp"
```

→ `source /opt/ros/noetic/setup.bash` が実行されていません．`~/.bashrc` に書かれているか確認してください．

### ビルドは通るが `rosrun` で見つからない

```
[rosrun] Couldn't find executable named talker below ...
```

→ `source ~/catkin_ws/devel/setup.bash` を実行していません．

### CMakeLists.txt を変更したのに反映されない

`catkin build` はソースの変更を自動検出しますが，`CMakeLists.txt` の変更は確実に反映するために一度クリーンすることがあります：

```bash
catkin clean
catkin build
```

---

## まとめ

```
Visual Studio（Windows）
    │  やっていること自体は同じ
    ▼
CMake + make（Linux 標準）
    │  ROS 用の「お作法」を追加
    ▼
catkin + CMake（ROS）
```

ROS のチュートリアルで出てくる `CMakeLists.txt` の変更は，ほとんどが以下の2行のパターンです：

```cmake
add_executable(ノード名 src/ソースファイル.cpp)
target_link_libraries(ノード名 ${catkin_LIBRARIES})
```

この2行が「このソースファイルをコンパイルして，ROS のライブラリと結合する」という意味だと分かれば，チュートリアルのビルド手順は読み解けます．
