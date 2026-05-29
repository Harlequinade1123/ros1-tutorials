# 10章: RViz ── データを視覚化する

ROS の各種データをリアルタイムに 3D 表示するツール **RViz** を学びます．

---

## RViz とは

**RViz**（ROS Visualization）は ROS 標準の 3D ビジュアライザです．

| 用途 | 例 |
|------|----|
| センサーデータの確認 | LiDAR スキャン・点群・カメラ画像 |
| ロボットの状態監視 | 位置・姿勢・座標フレーム（TF） |
| アルゴリズムのデバッグ | 経路・コストマップ・障害物 |
| 独自データの表示 | マーカー（矢印・球・テキストなど） |

ターミナルの `ROS_INFO` では数値しか見えませんが，RViz を使うと**空間的なデータを瞬時に把握**できます．

---

## 起動方法

roscore が動いている状態で：

```bash
rviz
# または
rosrun rviz rviz
```

> RViz 自体も「トピックを受け取って表示するノード」です．roscore が必要です．

---

## 画面構成

```
┌─────────────────────────────────────────────────────────┐
│  [ツールバー]  Interact / Move Camera / ...              │
├──────────────────┬────────────────────────┬─────────────┤
│                  │                        │             │
│  Displays        │   3D ビューポート      │  Views      │
│  パネル          │                        │  パネル     │
│                  │  ← 可視化の主役        │  (視点)     │
│  [Add]           │                        │             │
│  [Remove]        │                        │             │
│                  │                        │             │
├──────────────────┴────────────────────────┴─────────────┤
│  [Time]                                                  │
└─────────────────────────────────────────────────────────┘
```

| パネル | 役割 |
|--------|------|
| **Displays** | 表示する内容（Display）を追加・削除・設定する |
| **3D ビューポート** | データが描画されるメインの画面 |
| **Views** | 視点（カメラ）の種類と設定 |

---

## 基本的なマウス操作

| 操作 | 動作 |
|------|------|
| 左ドラッグ | 視点を回転 |
| 中ドラッグ（または Shift+左） | 視点を平行移動 |
| スクロール | ズームイン・アウト |
| `0` キー | 視点をリセット |

---

## Fixed Frame の設定

Displays パネル上部の「**Fixed Frame**」は，表示の基準座標フレームを指定します．

| 状況 | よく使う Fixed Frame |
|------|---------------------|
| 地図なし・絶対位置基準 | `map` |
| ロボット中心で表示 | `base_link` |
| オドメトリ基準（ロボットの移動基準） | `odom` |

> Fixed Frame に存在しないフレームを設定すると，Displays の項目に赤いエラーが表示されます．

---

## Displays パネルの操作

### Display の追加

1. **[Add]** ボタンをクリック
2. リストから Display タイプを選ぶ
3. **[OK]**

**「By topic」タブが便利** ── 現在 publish されているトピックを一覧表示し，そこから表示タイプを選べます．

### Display の設定

各 Display 名をクリックすると設定項目が展開されます．

```
✓ Odometry
  Topic: /odom
  Keep: 100
  Shape: Arrow
```

---

## 実際に試す：Marker を表示する

**Marker** は，C++ コードから RViz に好きな図形（球・矢印・テキストなど）を描くためのメッセージ型です．

### marker_publisher.cpp

`~/catkin_ws/src/ros_tutorial/src/marker_publisher.cpp` を作成します．

```cpp
#include <ros/ros.h>
#include <visualization_msgs/Marker.h>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "marker_publisher");
    ros::NodeHandle nh;

    ros::Publisher marker_pub =
        nh.advertise<visualization_msgs::Marker>("visualization_marker", 1);

    ros::Rate rate(1);

    while (ros::ok())
    {
        visualization_msgs::Marker marker;

        marker.header.frame_id = "map";
        marker.header.stamp    = ros::Time::now();   // ROS の現在時刻をタイムスタンプとして設定
        marker.ns              = "tutorial";
        marker.id              = 0;

        // 表示タイプ：SPHERE（球）
        marker.type   = visualization_msgs::Marker::SPHERE;
        marker.action = visualization_msgs::Marker::ADD;

        // 位置・向き
        marker.pose.position.x    = 0.0;
        marker.pose.position.y    = 0.0;
        marker.pose.position.z    = 0.0;
        marker.pose.orientation.w = 1.0;

        // サイズ [m]
        marker.scale.x = 0.5;
        marker.scale.y = 0.5;
        marker.scale.z = 0.5;

        // 色（RGBA，0.0〜1.0）
        marker.color.r = 1.0f;
        marker.color.g = 0.0f;
        marker.color.b = 0.0f;
        marker.color.a = 1.0f;

        marker_pub.publish(marker);
        ros::spinOnce();
        rate.sleep();
    }
    return 0;
}
```

> **`ros::Time::now()` について**: ROS の現在時刻を返す関数です．メッセージのヘッダに設定することで「このデータがいつ生成されたか」を記録します．RViz や rosbag がタイムスタンプを使って表示・記録を管理するため，`header.stamp` には常に設定するのが慣習です．

### CMakeLists.txt に追記

```cmake
add_executable(marker_publisher src/marker_publisher.cpp)
target_link_libraries(marker_publisher ${catkin_LIBRARIES})
```

### ビルドと実行

**ターミナル 1（roscore）：**
```bash
roscore
```

**ターミナル 2（marker_publisher）：**
```bash
cd ~/catkin_ws && catkin build
rosrun ros_tutorial marker_publisher
```

**ターミナル 3（RViz）：**
```bash
rviz
```

### RViz での確認手順

1. Fixed Frame を `map` に変更
2. **[Add]** → **「By topic」タブ** → `/visualization_marker` → **Marker** → **[OK]**

原点（0, 0, 0）に赤い球が表示されます．

### 複数の Marker を出す

`marker.id` を変えると同時に複数の図形を表示できます．また `marker.type` を変えると形が変わります．

| 定数 | 形状 |
|------|------|
| `SPHERE` | 球 |
| `CUBE` | 直方体 |
| `CYLINDER` | 円柱 |
| `ARROW` | 矢印 |
| `LINE_STRIP` | 折れ線 |
| `TEXT_VIEW_FACING` | テキスト（常に正面を向く） |

---

## 主な Display タイプ一覧

| タイプ | 対応メッセージ型 | 用途 |
|--------|----------------|------|
| **Grid** | なし | グリッドライン（床面の目安） |
| **TF** | なし | 座標フレーム間の関係を矢印で表示 |
| **Axes** | なし | 特定フレームを XYZ 軸で表示 |
| **Marker** | `visualization_msgs/Marker` | 自由な図形を描く |
| **Odometry** | `nav_msgs/Odometry` | 位置・速度を矢印で表示 |
| **Path** | `nav_msgs/Path` | 軌跡を線で表示 |
| **LaserScan** | `sensor_msgs/LaserScan` | LiDAR のスキャンデータ |
| **PointCloud2** | `sensor_msgs/PointCloud2` | 3D 点群 |
| **Image** | `sensor_msgs/Image` | カメラ画像 |

---

## 設定の保存・読み込み

RViz の Displays 設定は `.rviz` ファイルに保存できます．

```bash
# GUI から保存：File → Save Config As
# GUI から読み込み：File → Open Config

# コマンドラインで設定ファイルを指定して起動
rviz -d ~/my_config.rviz
```

launch ファイルから設定ファイル付きで RViz を起動することもできます（9 章参照）：

```xml
<node name="rviz" pkg="rviz" type="rviz"
      args="-d $(find ros_tutorial)/rviz/default.rviz" />
```

---

## rqt_graph との使い分け

4 章で紹介した `rqt_graph` と RViz は用途が異なります．

| ツール | 見えるもの |
|--------|-----------|
| **rqt_graph** | ノードとトピックの**接続関係**（グラフ構造） |
| **RViz** | トピックのデータ内容（**空間データ**） |
| **rostopic echo** | トピックのデータ内容（**テキスト**） |

---

[→ 11章: rosbag](11_rosbag.md)
