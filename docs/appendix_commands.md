# 付録: コマンドリファレンス

開発中によく使うコマンドをまとめた早見表です。詳細は各章を参照してください。

---

## Ubuntu 基本コマンド

### ファイル・ディレクトリ操作

| コマンド | 説明 | 例 |
|---------|------|----|
| `ls` | 現在のディレクトリの内容を一覧表示 | `ls` |
| `ls -la` | 隠しファイルも含めて詳細表示 | `ls -la` |
| `cd <パス>` | ディレクトリを移動 | `cd ~/catkin_ws/src` |
| `cd ..` | 1つ上のディレクトリへ移動 | `cd ..` |
| `cd ~` | ホームディレクトリへ移動 | `cd ~` |
| `pwd` | 現在のディレクトリのパスを表示 | `pwd` |
| `mkdir <名前>` | ディレクトリを作成 | `mkdir launch` |
| `mkdir -p <パス>` | 中間ディレクトリも含めて一括作成 | `mkdir -p ~/catkin_ws/src` |
| `cp <元> <先>` | ファイルをコピー | `cp talker.cpp talker_bak.cpp` |
| `cp -r <元> <先>` | ディレクトリをまるごとコピー | `cp -r ros_tutorial/ backup/` |
| `mv <元> <先>` | ファイルを移動またはリネーム | `mv old_name.cpp new_name.cpp` |
| `rm <ファイル>` | ファイルを削除 | `rm temp.cpp` |
| `rm -r <ディレクトリ>` | ディレクトリをまるごと削除（注意）| `rm -r build/` |
| `touch <ファイル>` | 空のファイルを作成 | `touch talker.cpp` |
| `cat <ファイル>` | ファイルの内容を表示 | `cat CMakeLists.txt` |

### テキストエディタ

| コマンド | 説明 |
|---------|------|
| `nano <ファイル>` | シンプルなターミナルエディタを開く（初心者向け） |
| `gedit <ファイル>` | GUI テキストエディタを開く（デスクトップ環境が必要） |
| `code <ファイル>` | VS Code で開く（インストール済みの場合） |

**nano の基本操作：**

| キー | 動作 |
|-----|------|
| `Ctrl + O` | 保存 |
| `Ctrl + X` | 終了 |
| `Ctrl + K` | 行を切り取り |
| `Ctrl + U` | 貼り付け |
| `Ctrl + W` | 検索 |

### テキスト検索

| コマンド | 説明 | 例 |
|---------|------|----|
| `grep <文字列> <ファイル>` | ファイル内の文字列を検索 | `grep "publish" talker.cpp` |
| `grep -r <文字列> <ディレクトリ>` | ディレクトリ内を再帰的に検索 | `grep -r "chatter" src/` |
| `grep -n <文字列> <ファイル>` | 行番号も表示して検索 | `grep -n "ros::init" talker.cpp` |

### システム・プロセス

| コマンド | 説明 | 例 |
|---------|------|----|
| `sudo <コマンド>` | 管理者権限でコマンドを実行 | `sudo apt install vim` |
| `apt install <パッケージ>` | ソフトウェアをインストール | `sudo apt install git` |
| `apt update` | パッケージリストを更新 | `sudo apt update` |
| `ps aux` | 実行中のプロセスを一覧表示 | `ps aux` |
| `ps aux \| grep ros` | ROS 関連プロセスだけ絞り込む | `ps aux \| grep ros` |
| `kill <PID>` | プロセスを終了（PID は ps で確認） | `kill 1234` |
| `killall <名前>` | 名前でプロセスを終了 | `killall roscore` |

### ターミナル操作のコツ

| 操作 | 説明 |
|------|------|
| `Tab` キー | コマンド・ファイル名を補完（2回押すと候補一覧） |
| `↑` / `↓` キー | コマンド履歴を遡る |
| `Ctrl + C` | 実行中のプログラムを強制終了 |
| `Ctrl + Z` | 実行中のプログラムを一時停止（バックグラウンドへ） |
| `Ctrl + L` | 画面をクリア（`clear` コマンドと同じ） |
| `history` | 過去のコマンド履歴を表示 |

### 環境変数

| コマンド | 説明 | 例 |
|---------|------|----|
| `echo $変数名` | 環境変数の値を表示 | `echo $ROS_MASTER_URI` |
| `export 変数=値` | 環境変数を設定（そのターミナルのみ有効）| `export ROS_MASTER_URI=http://localhost:11311` |
| `source <ファイル>` | ファイルの設定を現在のターミナルに読み込む | `source ~/.bashrc` |
| `echo $?` | 直前のコマンドの終了コードを表示（0=成功） | `echo $?` |

### パイプとリダイレクト

| 記号 | 説明 | 例 |
|-----|------|-----|
| `\|` | 前のコマンドの出力を次のコマンドへ渡す | `rostopic list \| grep sensor` |
| `>` | 出力をファイルに書き込む（上書き） | `rosnode list > nodes.txt` |
| `>>` | 出力をファイルに追記する | `echo "memo" >> notes.txt` |

---

## ROS コマンド

### 起動・実行

| コマンド | 説明 | 例 |
|---------|------|----|
| `roscore` | ROS マスターを起動する（必ず最初に起動）| `roscore` |
| `rosrun <pkg> <node>` | 指定したノードを起動 | `rosrun ros_tutorial talker` |
| `roslaunch <pkg> <file.launch>` | launch ファイルで複数ノードを起動（roscore 不要）| `roslaunch ros_tutorial talker_listener.launch` |
| `roslaunch <pkg> <file.launch> <arg>:=<値>` | 引数を渡して launch ファイルを実行 | `roslaunch ros_tutorial talker_listener.launch rate:=5` |

### ノード（rosnode）

| コマンド | 説明 |
|---------|------|
| `rosnode list` | 起動中のノード一覧を表示 |
| `rosnode info <ノード名>` | ノードの詳細（pub/sub しているトピック等）を表示 |
| `rosnode kill <ノード名>` | 指定したノードを終了させる |
| `rosnode ping <ノード名>` | ノードへの疎通確認 |

```bash
# 例
rosnode list
rosnode info /talker
rosnode kill /talker
```

### トピック（rostopic）

| コマンド | 説明 |
|---------|------|
| `rostopic list` | 現在アクティブなトピック一覧を表示 |
| `rostopic echo <トピック名>` | トピックの内容をリアルタイム表示 |
| `rostopic hz <トピック名>` | トピックの送信レートを計測 |
| `rostopic info <トピック名>` | トピックの型・pub/sub しているノードを表示 |
| `rostopic type <トピック名>` | トピックのメッセージ型を表示 |
| `rostopic pub <トピック> <型> <データ>` | コマンドラインからトピックにデータを送信 |

```bash
# 例
rostopic list
rostopic echo /chatter
rostopic hz /chatter
rostopic info /chatter

# コマンドラインからデータを送る（-1 で1回だけ送信）
rostopic pub -1 /chatter std_msgs/String "data: 'hello'"

# 10Hz で繰り返し送信
rostopic pub -r 10 /chatter std_msgs/String "data: 'hello'"
```

### メッセージ型（rosmsg）

| コマンド | 説明 |
|---------|------|
| `rosmsg list` | 利用可能なメッセージ型の一覧 |
| `rosmsg show <型名>` | メッセージ型のフィールドを表示 |
| `rosmsg package <パッケージ名>` | パッケージ内のメッセージ型一覧 |

```bash
# 例
rosmsg show std_msgs/String
rosmsg show sensor_msgs/Image
rosmsg show ros_tutorial/SensorData
```

### サービス（rosservice）

| コマンド | 説明 |
|---------|------|
| `rosservice list` | 現在アクティブなサービス一覧を表示 |
| `rosservice info <サービス名>` | サービスの詳細を表示 |
| `rosservice type <サービス名>` | サービスの型を表示 |
| `rosservice call <サービス名> <引数>` | コマンドラインからサービスを呼び出す |

```bash
# 例
rosservice list
rosservice info /add_two_ints
rosservice type /add_two_ints

# サービスを呼び出す
rosservice call /add_two_ints "a: 3
b: 7"
```

### サービス型（rossrv）

| コマンド | 説明 |
|---------|------|
| `rossrv show <型名>` | サービス型の定義を表示 |
| `rossrv list` | 利用可能なサービス型の一覧 |

```bash
rossrv show ros_tutorial/AddTwoInts
```

### パラメータ（rosparam）

| コマンド | 説明 |
|---------|------|
| `rosparam list` | 現在設定されているパラメータの一覧 |
| `rosparam get <名前>` | パラメータの値を取得 |
| `rosparam set <名前> <値>` | パラメータの値を設定 |
| `rosparam delete <名前>` | パラメータを削除 |
| `rosparam dump <ファイル.yaml>` | 全パラメータを YAML ファイルに保存 |
| `rosparam load <ファイル.yaml>` | YAML ファイルからパラメータを読み込む |

```bash
# 例
rosparam list
rosparam get /param_example/robot_name
rosparam set /param_example/publish_rate 5
rosparam dump backup.yaml
rosparam load backup.yaml
```

### ビルド・パッケージ作成（catkin）

`catkin build` や `catkin create` は `python3-catkin-tools` パッケージが提供するコマンドです（2章のインストール手順で導入済み）。

| コマンド | 説明 |
|---------|------|
| `catkin build` | ワークスペース全体をビルド |
| `catkin build <パッケージ名>` | 特定のパッケージだけビルド |
| `catkin build --cmake-args -DCMAKE_BUILD_TYPE=Release` | リリースビルド（最適化あり）|
| `catkin clean` | ビルド成果物を削除してクリーンにする |
| `catkin config` | ワークスペースの設定を確認 |
| `catkin create pkg <名前> --catkin-deps <依存...>` | 新しいパッケージを作成 |

```bash
# 例
cd ~/catkin_ws
catkin build
catkin build ros_tutorial
catkin clean   # 全パッケージをクリーン

# パッケージ作成
cd ~/catkin_ws/src
catkin create pkg my_pkg --catkin-deps roscpp std_msgs
```

> ビルド後は必ず `source ~/catkin_ws/devel/setup.bash` を実行するか、ターミナルを開き直してください。

#### 旧コマンド（catkin_make / catkin_create_pkg）

`catkin_tools` が普及する前から使われている旧来のコマンドです。ネット上のサンプルで見かけることがあります。

| 旧コマンド | 新コマンド | 説明 |
|-----------|-----------|------|
| `catkin_make` | `catkin build` | ワークスペースのビルド |
| `catkin_make --pkg <名前>` | `catkin build <名前>` | 特定パッケージのみビルド |
| `catkin_create_pkg <名前> [依存...]` | `catkin create pkg <名前> --catkin-deps [依存...]` | パッケージ作成 |

`catkin_make` と `catkin build` は**同一ワークスペース内で混在させると壊れます**。このチュートリアルでは一貫して `catkin build` を使ってください。

### パッケージ検索・移動（rospack・roscd・rosls）

| コマンド | 説明 |
|---------|------|
| `rospack find <パッケージ名>` | パッケージのインストールパスを表示 |
| `roscd <パッケージ名>` | パッケージのディレクトリへ移動 |
| `rosls <パッケージ名>` | パッケージ内のファイルを一覧表示 |

```bash
# 例
rospack find ros_tutorial
roscd ros_tutorial
rosls ros_tutorial
```

### バッグファイル（rosbag）──トピックの記録・再生

| コマンド | 説明 |
|---------|------|
| `rosbag record -a` | 全トピックを記録 |
| `rosbag record <トピック1> <トピック2>` | 指定トピックのみ記録 |
| `rosbag record -O <ファイル名> <トピック>` | ファイル名を指定して記録 |
| `rosbag play <ファイル.bag>` | バッグファイルを再生 |
| `rosbag play -r 2 <ファイル.bag>` | 2倍速で再生 |
| `rosbag info <ファイル.bag>` | バッグファイルの内容を確認 |

```bash
# 例：/chatter を記録
rosbag record -O my_record.bag /chatter

# 再生
rosbag play my_record.bag

# 内容確認
rosbag info my_record.bag
```

### 診断・その他

| コマンド | 説明 |
|---------|------|
| `roswtf` | ROS の設定や接続の問題を自動診断する |
| `rosversion -d` | 現在の ROS ディストリビューション名を表示 |
| `rosversion roscpp` | 特定パッケージのバージョンを表示 |
| `rqt_graph` | ノード・トピックの接続関係をグラフで可視化（GUI）|
| `rqt_plot` | トピックの数値をグラフでリアルタイム表示（GUI）|
| `rviz` | 3D 可視化ツールを起動（GUI）|

---

## よく使うコマンドの組み合わせ

```bash
# 特定のトピック名が含まれるものだけ表示
rostopic list | grep sensor

# メッセージ型を確認してから内容を表示
rostopic type /chatter
rostopic echo /chatter

# ノードが落ちていないか・トピックが来ているか確認する
rosnode list
rostopic hz /chatter

# ビルドして即実行
cd ~/catkin_ws && catkin build && source devel/setup.bash && rosrun ros_tutorial talker
```

---

## 環境変数（ROS 関連）

| 変数 | 説明 | 確認方法 |
|-----|------|---------|
| `ROS_MASTER_URI` | roscore のアドレス（デフォルト: `http://localhost:11311`）| `echo $ROS_MASTER_URI` |
| `ROS_PACKAGE_PATH` | パッケージの検索パス | `echo $ROS_PACKAGE_PATH` |
| `ROS_DISTRO` | 現在の ROS ディストリビューション | `echo $ROS_DISTRO` |

> 複数の PC で ROS を使う場合（分散システム）は `ROS_MASTER_URI` と `ROS_IP` を適切に設定する必要があります。
