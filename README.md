# ROS1 入門チュートリアル

C++ の基礎知識はあるが，ROS は初めてという方を対象にした入門教材です．

## 動作環境

| 項目 | バージョン |
|------|-----------|
| OS | Ubuntu 20.04 LTS |
| ROS | Noetic Ninjemys（ROS1 の最終 LTS） |
| 言語 | C++14 |

## 目次

| # | 章タイトル | 概要 |
|---|-----------|------|
| 1 | [ROS とは](docs/01_introduction.md) | ROS の概要・基本概念 |
| 2 | [環境構築](docs/02_setup.md) | インストール・ワークスペース作成 |
| 3 | [最初のパッケージを作る](docs/03_first_package.md) | パッケージ作成・ビルドの流れ |
| 4 | [Publisher / Subscriber](docs/04_publisher_subscriber.md) | ROS の基本通信（トピック） |
| 5 | [サービス](docs/05_service.md) | リクエスト・レスポンス型の通信 |
| 6 | [アクション通信](docs/06_actionlib.md) | 長時間処理のフィードバックとキャンセル |
| 7 | [カスタムメッセージ](docs/07_custom_messages.md) | 独自のメッセージ型を定義する |
| 8 | [パラメータ](docs/08_parameters.md) | 実行時に値を設定・取得する |
| 9 | [launch ファイル](docs/09_launch_files.md) | 複数ノードをまとめて起動する |
| 10 | [RViz](docs/10_rviz.md) | データを 3D で視覚化する |
| 11 | [rosbag](docs/11_rosbag.md) | トピックの記録と再生 |
| 12 | [tf / tf2](docs/12_tf.md) | 座標フレーム間の変換を管理する |
| 13 | [C++ クラス入門](docs/13_cpp_class_basics.md) | クラスの概要（詳細は付録C） |
| 14 | [クラスを使った ROS プログラミング](docs/14_ros_with_class.md) | ノードをクラスで書き直す・サービス・アクションのクラス版 |
| 付録A | [コマンドリファレンス](docs/appendix_commands.md) | Ubuntu・ROS コマンド早見表 |
| 付録B | [CMake とビルドの仕組み](docs/appendix_cmake.md) | コンパイルの基礎・CMake・catkin の違い |
| 付録C | [C++ クラス入門](docs/appendix_cpp_class.md) | クラス・コンストラクタ・メンバ変数の基礎 |

## Kobuki 演習

| # | 章タイトル | 概要 |
|---|-----------|------|
| 15 | [Kobuki 基礎](docs/15_kobuki_basics.md) | 速度コマンド・前進・往復動作（演習 1〜2） |
| 16 | [Kobuki センサー](docs/16_kobuki_sensors.md) | バンパー・崖・オドメトリ（演習 3〜5） |
| 17 | [クラスを使った Kobuki プログラミング](docs/17_kobuki_class.md) | クラス設計・LED/サウンド制御（演習 6〜7） |

## 読み進め方

- **1 章から順番に**読み進めることを推奨します
- 詰まったときは `roscore` が起動しているか，`source` コマンドを実行したか，を先に確認しましょう

## パッケージ名について

このチュートリアルでは `ros_tutorial` というパッケージ名を使います．

---

> **注意**: ROS1 Noetic のサポート期間は **2025年5月** までです．新規プロジェクトでは ROS2 も検討してください．このチュートリアルで学ぶ概念（ノード・トピック・サービス等）は ROS2 でも共通しています．
