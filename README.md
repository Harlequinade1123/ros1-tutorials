# ROS1 入門チュートリアル

C++ の基礎知識はあるが、ROS は初めてという方を対象にした入門教材です。

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
| 6 | [カスタムメッセージ](docs/06_custom_messages.md) | 独自のメッセージ型を定義する |
| 7 | [パラメータ](docs/07_parameters.md) | 実行時に値を設定・取得する |
| 8 | [launch ファイル](docs/08_launch_files.md) | 複数ノードをまとめて起動する |
| 9 | [rosbag](docs/09_rosbag.md) | トピックの記録と再生 |
| 10 | [C++ クラス入門](docs/10_cpp_class_basics.md) | クラスの基本をゼロから学ぶ |
| 11 | [クラスを使った ROS プログラミング](docs/11_ros_with_class.md) | ノードをクラスで書き直す |
| 付録 | [コマンドリファレンス](docs/appendix_commands.md) | Ubuntu・ROS コマンド早見表 |

## Kobuki 演習

| # | 章タイトル | 概要 |
|---|-----------|------|
| 12 | [Kobuki 基礎](docs/12_kobuki_basics.md) | 速度コマンド・前進・往復動作（演習 1〜2） |
| 13 | [Kobuki センサー](docs/13_kobuki_sensors.md) | バンパー・崖・オドメトリ（演習 3〜5） |
| 14 | [クラスを使った Kobuki プログラミング](docs/14_kobuki_class.md) | クラス設計・LED/サウンド制御（演習 6〜7） |

## 読み進め方

- **1 章から順番に**読み進めることを推奨します
- 詰まったときは `roscore` が起動しているか、`source` コマンドを実行したか、を先に確認しましょう

## パッケージ名について

このチュートリアルでは `ros_tutorial` というパッケージ名を使います。

---

> **注意**: ROS1 Noetic のサポート期間は **2025年5月** までです。新規プロジェクトでは ROS2 も検討してください。このチュートリアルで学ぶ概念（ノード・トピック・サービス等）は ROS2 でも共通しています。
