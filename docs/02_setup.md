# 2章: 環境構築

## 前提条件

- **Ubuntu 20.04 LTS** がインストールされていること（仮想マシン・WSL2 でも可）
- インターネットに接続できること
- `sudo` コマンドが使えること

> **WSL2 を使う場合の注意**: WSL2 でも ROS は動作しますが，GUI ツール（rviz 等）の表示には追加設定が必要です．RViz を使う 10 章では GUI が必要になるため，Ubuntu のネイティブ環境または GUI を有効にした WSL2 環境（WSLg など）を推奨します．

---

## ROS Noetic のインストール

ターミナルを開いて，以下のコマンドを上から順に実行してください．

### 1. パッケージリストの追加

```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

### 2. 認証キーの追加

```bash
sudo apt install curl -y
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```

### 3. ROS Noetic のインストール

```bash
sudo apt update
sudo apt install ros-noetic-desktop-full -y
```

> `desktop-full` を選ぶと，rviz などの GUI ツールも一緒にインストールされます．インストールには数分〜十数分かかります．

### 4. rosdep の初期化

`rosdep` は依存パッケージを自動でインストールするツールです．

```bash
sudo apt install python3-rosdep -y
sudo rosdep init
rosdep update
```

### 5. 環境変数の設定

ROS のコマンドを使えるようにするため，`.bashrc`（ターミナル起動時に自動実行される設定ファイル）に追記します．

```bash
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 6. ビルドツールのインストール

```bash
sudo apt install python3-catkin-tools python3-osrf-pycommon -y
```

---

## インストールの確認

以下のコマンドを実行してバージョンが表示されれば成功です．

```bash
rosversion -d
```

出力例：
```
noetic
```

---

## catkin ワークスペースの作成

ROS では，自分で書いたコードを **ワークスペース** という専用のフォルダにまとめて管理します．

### ワークスペースの作成とビルド

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws
catkin build
```

コマンドを実行すると `build/`・`devel/`・`logs/` フォルダが生成されます．

```
~/catkin_ws/
├── src/        ← ここに自分のパッケージ（コード）を置く
├── build/      ← ビルドの中間ファイル（自動生成）
├── devel/      ← ビルド結果（実行ファイルなど）（自動生成）
└── logs/       ← ビルドログ（自動生成）
```

### ワークスペースを常に使えるように設定

```bash
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

> **なぜ `source` が必要なのか?**
> `source` は「このファイルに書いてある環境変数の設定を今のターミナルに読み込む」コマンドです．
> `.bashrc` に書いておくことで，ターミナルを開くたびに自動で実行されます．

---

## 動作確認

### roscore を起動してみる

新しいターミナルを開いて：

```bash
roscore
```

以下のような出力が出れば成功です．

```
... logging to /home/username/.ros/log/...
Checking log directory for disk usage. This may take a while.
Press Ctrl-C to interrupt
Done checking log file disk usage. Usage is <1GB.

started roslaunch server http://localhost:XXXXX/
ros_comm version 1.15.X

SUMMARY
========

PARAMETERS
 * /rosdistro: noetic
 * /rosversion: 1.15.X

NODES

auto-starting new master
process[master]: started with pid [XXXX]
ROS_MASTER_URI=http://localhost:11311/

setting /run_id to ...
process[rosout-1]: started with pid [XXXX]
started core service [/rosout]
```

`Ctrl + C` で停止できます．

---

## よくあるエラーと対処法

### `roscore: command not found`

→ `source /opt/ros/noetic/setup.bash` が実行されていません．
```bash
source /opt/ros/noetic/setup.bash
```
または `.bashrc` への追記が正しく行われているか確認してください．

### `catkin: command not found`

→ `python3-catkin-tools` がインストールされていないか，`source /opt/ros/noetic/setup.bash` が実行されていません．
```bash
sudo apt install python3-catkin-tools
source /opt/ros/noetic/setup.bash
```

---

環境の準備ができました．次の章では，最初のパッケージを作成します．

[→ 3章: 最初のパッケージを作る](03_first_package.md)
