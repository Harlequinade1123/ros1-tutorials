# 11章: rosbag ── トピックの記録と再生

## rosbag とは

**rosbag** は，ROS トピックのデータを **ファイルに記録（録画）して後から再生** できるツールです．`.bag` という拡張子のファイルに保存されます．

### 何に使うのか

| 用途 | 説明 |
|------|------|
| **デバッグ** | 実機でセンサーデータを収録し，PCだけで何度でも再現して解析できる |
| **開発効率化** | ロボットがなくても収録済みデータでアルゴリズムを試せる |
| **記録・比較** | 改良前後の動作データを保存して比較できる |
| **データセット作成** | 機械学習の学習データとして活用できる |

---

## 基本的な使い方

### rosbag record ── 記録

```bash
# 全トピックを記録（ファイル名は自動生成）
rosbag record -a

# 特定のトピックだけ記録
rosbag record /chatter /odom

# ファイル名を指定して記録（.bag 拡張子は自動付与）
rosbag record -O my_data /chatter

# 指定した秒数だけ記録して自動終了
rosbag record --duration=10 -O sensor_10s /odom
```

> `Ctrl + C` で記録を終了します．

### rosbag play ── 再生

```bash
# 再生（記録時と同じトピック名で publish される）
rosbag play my_data.bag

# ループ再生（Ctrl+C で終了）
rosbag play -l my_data.bag

# 再生速度を変える（0.5 = 半速，2.0 = 2倍速）
rosbag play -r 0.5 my_data.bag

# 指定秒数後から再生
rosbag play -s 5 my_data.bag

# 一時停止状態で起動（スペースキーで再生/一時停止）
rosbag play --pause my_data.bag
```

### rosbag info ── 内容確認

```bash
rosbag info my_data.bag
```

出力例：
```
path:        my_data.bag
version:     2.0
duration:    10.2s
start:       May 28 2026 12:00:00.00
end:         May 28 2026 12:00:10.20
size:        45.2 KB
messages:    102
types:       std_msgs/String [992ce8a1687cec8c8bd2be6717a3b838]
topics:      /chatter   102 msgs @ 10.0 Hz : std_msgs/String
```

---

## 実際に試してみる

talker / listener を使って記録・再生の流れを体験します．

### 手順 1: talker を動かしながら記録する

**ターミナル 1（roscore）：**
```bash
roscore
```

**ターミナル 2（talker を起動）：**
```bash
rosrun ros_tutorial talker
```

**ターミナル 3（記録開始）：**
```bash
mkdir -p ~/rosbag_data
cd ~/rosbag_data
rosbag record -O chatter_data /chatter
```

10 秒ほど待ってから `Ctrl+C` で記録終了．

### 手順 2: talker を止めてバッグファイルを確認する

ターミナル 2 の talker を `Ctrl+C` で終了．

```bash
rosbag info ~/rosbag_data/chatter_data.bag
```

### 手順 3: バッグファイルを再生して listener で受け取る

**ターミナル 2（listener を起動）：**
```bash
rosrun ros_tutorial listener
```

**ターミナル 3（バッグファイルを再生）：**
```bash
rosbag play ~/rosbag_data/chatter_data.bag
```

listener のターミナルに，記録済みのメッセージが再生されて表示されます．**talker を動かしていなくてもデータが届く** ことを確認してください．

---

## rosbag と rostopic echo の組み合わせ

再生中に `rostopic echo` でリアルタイム確認できます．

```bash
# ターミナル A: 再生
rosbag play my_data.bag

# ターミナル B: 内容を確認
rostopic echo /chatter
```

---

## よく使うオプションまとめ

### record オプション

| オプション | 説明 | 例 |
|-----------|------|----|
| `-a` | 全トピックを記録 | `rosbag record -a` |
| `-O <名前>` | ファイル名を指定 | `rosbag record -O data /odom` |
| `--duration=<秒>` | 指定秒数で自動停止 | `--duration=30` |
| `--split --size=<MB>` | 指定サイズで分割保存 | `--split --size=100` |
| `-x <正規表現>` | 除外するトピックを指定 | `-x "/camera/.*"` |

### play オプション

| オプション | 説明 | 例 |
|-----------|------|----|
| `-l` | ループ再生 | `rosbag play -l data.bag` |
| `-r <倍率>` | 再生速度の変更 | `-r 2.0` |
| `-s <秒>` | 開始位置を指定 | `-s 10` |
| `--pause` | 一時停止状態で起動 | `rosbag play --pause data.bag` |
| `--topics <トピック>` | 指定トピックだけ再生 | `--topics /odom /chatter` |
| `--clock` | 記録時刻でクロックを発行（時刻に依存する処理に必要） | `--clock` |

---

## Kobuki のデータを記録する

実際のロボット開発では，センサーデータを記録して後でゆっくり分析するのがよくある使い方です．

```bash
# Kobuki の主要なトピックを記録
rosbag record -O kobuki_session \
  /mobile_base/events/bumper \
  /mobile_base/events/cliff \
  /odom
```

記録したデータは，ロボットなしで PC だけで再生・確認できます．

```bash
# 記録内容を確認
rosbag info kobuki_session.bag

# 再生して rostopic echo で確認
rosbag play kobuki_session.bag &
rostopic echo /mobile_base/events/bumper
```

---

## トピック名を変えて再生する

再生時にトピック名を変更（remap）することもできます．

```bash
# /chatter を /old_chatter として再生
rosbag play data.bag /chatter:=/old_chatter
```

---

## GUI で確認する（rqt_bag）

インストールされている場合は GUI でバッグファイルを視覚的に確認できます．

```bash
sudo apt install ros-noetic-rqt-bag -y
rqt_bag ~/rosbag_data/chatter_data.bag
```

タイムライン表示でメッセージの分布を確認したり，特定時刻のデータを見たりできます．

---

## よくあるトラブル

### 再生しても何も届かない

**原因**: listener が `/chatter` を subscribe しているが，rosbag も同じトピックに publish するには roscore が動いている必要があります．

```bash
# 確認
rosnode list    # ← roscore が起動していれば /rosout が見える
rostopic list   # ← 再生中は /chatter が見えるはず
```

### `--clock` が必要な場合

`ros::Time::now()` を使うコードと rosbag を組み合わせると，時刻のずれが問題になることがあります．その場合は：

```bash
# rosbag 側
rosbag play --clock data.bag

# roscore 側（事前に設定）
rosparam set /use_sim_time true
```

---

[→ 12章: tf / tf2](12_tf.md)
