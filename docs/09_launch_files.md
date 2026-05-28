# 9章: launch ファイル

4 章では talker・listener・roscore を **3 つのターミナル** で別々に起動しました．ノードが増えるたびにターミナルを開くのは不便です．**launch ファイル** を使うと，複数のノードを 1 コマンドで起動できます．

---

## launch ファイルとは

XML 形式で書かれた設定ファイルで，以下のことができます：

- 複数のノードをまとめて起動する
- パラメータを設定する
- トピック名を変更する（リマップ）
- 別の launch ファイルを読み込む

**`roslaunch` コマンドを使うと，roscore を別途起動しなくてよい**（自動で起動される）のも大きなメリットです．

---

## launch ファイルの作成

パッケージ内に `launch/` フォルダを作って，そこに置くのが慣例です．

```bash
mkdir -p ~/catkin_ws/src/ros_tutorial/launch
```

### 基本的な launch ファイル

`~/catkin_ws/src/ros_tutorial/launch/talker_listener.launch` を作成：

```xml
<launch>
  <!-- talker ノードを起動 -->
  <node pkg="ros_tutorial" type="talker" name="talker" output="screen"/>

  <!-- listener ノードを起動 -->
  <node pkg="ros_tutorial" type="listener" name="listener" output="screen"/>
</launch>
```

**`<node>` タグの属性：**

| 属性 | 意味 |
|------|------|
| `pkg` | パッケージ名 |
| `type` | 実行ファイル名（`rosrun` の第2引数と同じ） |
| `name` | このノードにつける名前（ROS 内での識別名） |
| `output="screen"` | ログをターミナルに表示する（省略すると表示されない） |

### 実行

```bash
roslaunch ros_tutorial talker_listener.launch
```

1 コマンドで talker と listener が同時に起動します．`Ctrl+C` で全ノードが終了します．

---

## パラメータを渡す

launch ファイルでパラメータを設定し，ノード内で読み取ることができます．

### `<param>` タグ

```xml
<launch>
  <node pkg="ros_tutorial" type="talker" name="talker" output="screen">
    <!-- このノードに "publish_rate" パラメータを設定 -->
    <param name="publish_rate" value="5"/>
  </node>

  <node pkg="ros_tutorial" type="listener" name="listener" output="screen"/>
</launch>
```

パラメータの読み取り方は [8章: パラメータ](08_parameters.md) で既に説明しました．

---

## 引数（arg）を使う

launch ファイルに **変数** を定義して，起動時に値を渡すことができます．

```xml
<launch>
  <!-- 引数の定義（デフォルト値あり） -->
  <arg name="rate" default="10"/>

  <node pkg="ros_tutorial" type="talker" name="talker" output="screen">
    <!-- arg の値を param として渡す -->
    <param name="publish_rate" value="$(arg rate)"/>
  </node>

  <node pkg="ros_tutorial" type="listener" name="listener" output="screen"/>
</launch>
```

起動時に引数を変えたい場合：

```bash
# デフォルト値（rate=10）で起動
roslaunch ros_tutorial talker_listener.launch

# rate を 1 に変えて起動
roslaunch ros_tutorial talker_listener.launch rate:=1
```

---

## トピック名の変更（remap）

同じノードを使いながら，トピック名だけ変えたいことがあります．

```xml
<launch>
  <node pkg="ros_tutorial" type="talker" name="talker" output="screen">
    <!-- "chatter" というトピックを "my_topic" に変更 -->
    <remap from="chatter" to="my_topic"/>
  </node>

  <node pkg="ros_tutorial" type="listener" name="listener" output="screen">
    <remap from="chatter" to="my_topic"/>
  </node>
</launch>
```

これによりソースコードを変更せずにトピック名を変えられます．

---

## 別の launch ファイルを読み込む

大規模なシステムでは launch ファイルを分割して管理します．

```xml
<launch>
  <!-- 別の launch ファイルを読み込む -->
  <include file="$(find ros_tutorial)/launch/talker_listener.launch"/>
</launch>
```

`$(find パッケージ名)` でそのパッケージのインストールパスを取得できます．

---

## launch ファイルの主なタグまとめ

| タグ | 用途 |
|------|------|
| `<launch>` | ルート要素（必須） |
| `<node>` | ノードの起動 |
| `<param>` | パラメータの設定 |
| `<arg>` | 引数の定義 |
| `<remap>` | トピック名の変更 |
| `<include>` | 別の launch ファイルの読み込み |
| `<group>` | 複数のタグをグループ化（名前空間の設定等） |

---

[→ 10章: RViz ── データを視覚化する](10_rviz.md)
