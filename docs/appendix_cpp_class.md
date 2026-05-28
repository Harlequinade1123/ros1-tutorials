# 付録C: C++ クラス入門

ROS のノードをクラスで書くために必要な C++ クラスの基礎をまとめます．  
[14章: クラスを使った ROS プログラミング](14_ros_with_class.md) の前に読んでおくことを推奨します．

---

## クラスとは

クラスは「**データ（変数）と処理（関数）を一つにまとめた設計図**」です．

設計図から実際のモノを作ることを **インスタンス化（オブジェクトの生成）** と呼びます．

```
クラス      ＝ 設計図（ロボットの仕様書）
オブジェクト ＝ 実際のモノ（その仕様書で作ったロボット）
```

1 つの設計図から複数のロボットを作れるように，1 つのクラスから複数のオブジェクトを生成できます．

---

## 最初の例

まずコード全体を見て，それからパーツごとに説明します．

```cpp
#include <iostream>
#include <string>

class Robot
{
public:
    // コンストラクタ
    Robot(std::string name, int speed)
        : name_(name), speed_(speed), distance_(0)
    {
        std::cout << name_ << " が起動しました" << std::endl;
    }

    // メンバ関数（メソッド）
    void move(int steps)
    {
        distance_ += steps * speed_;
        std::cout << name_ << " が " << steps << " 歩移動（累計: "
                  << distance_ << "）" << std::endl;
    }

    void showStatus() const
    {
        std::cout << "名前: " << name_
                  << "  速度: " << speed_
                  << "  累計距離: " << distance_ << std::endl;
    }

private:
    std::string name_;
    int         speed_;
    int         distance_;
};

int main()
{
    Robot robot1("Taro", 10);
    Robot robot2("Hanako", 5);

    robot1.move(3);
    robot2.move(5);
    robot1.move(2);

    robot1.showStatus();
    robot2.showStatus();

    return 0;
}
```

出力：
```
Taro が起動しました
Hanako が起動しました
Taro が 3 歩移動（累計: 30）
Hanako が 5 歩移動（累計: 25）
Taro が 2 歩移動（累計: 50）
名前: Taro  速度: 10  累計距離: 50
名前: Hanako  速度: 5  累計距離: 25
```

---

## パーツごとの解説

### クラスの定義

```cpp
class Robot
{
    // ここに中身を書く
};  // ← セミコロンが必要！
```

クラス名は慣習として**大文字始まり**にします（`Robot`・`MyClass`・`SensorNode` など）．

---

### public と private

クラスの中身は **公開（public）** か **非公開（private）** に分けます．

```cpp
class Robot
{
public:
    // ← ここは外から使える
    void move(int steps) { ... }

private:
    // ← ここは外から使えない（クラスの内部だけで使う）
    int distance_;
};
```

```cpp
Robot robot("Taro", 10);
robot.move(3);       // OK（public なので外から呼べる）
robot.distance_;     // エラー！（private なので外からアクセス不可）
```

**なぜ private にするのか？**
- 内部の変数を外から直接変えられると，予期しないバグが起きやすい
- 「このクラスを使う人は `move()` だけ呼べばよい」という設計にできる

---

### メンバ変数

クラスが「持っているデータ」です．

```cpp
private:
    std::string name_;
    int         speed_;
    int         distance_;
```

慣習として末尾に `_` をつけてメンバ変数だとわかるようにします（必須ではありません）．

---

### コンストラクタ

**オブジェクトを生成したときに自動で呼ばれる特別な関数**です．

- クラス名と同じ名前
- 戻り値の型は書かない

```cpp
Robot(std::string name, int speed)
    : name_(name), speed_(speed), distance_(0)    // 初期化リスト
{
    std::cout << name_ << " が起動しました" << std::endl;
}
```

`: name_(name), speed_(speed), distance_(0)` の部分を**初期化リスト**と呼びます．  
メンバ変数をここで初期化するのは C++ の慣習です（`{ }` の中で代入するより効率的）．

---

### メンバ関数（メソッド）

クラスが「できること」を定義した関数です．

```cpp
void move(int steps)
{
    distance_ += steps * speed_;
    // ← メンバ変数に直接アクセスできる
}
```

クラスの中（`{ }` 内）の関数は，同じクラスのメンバ変数に `this->` なしで直接アクセスできます．

#### `const` メンバ関数

```cpp
void showStatus() const { ... }
```

`const` をつけると「この関数はメンバ変数を変更しない」という約束になります．読み取り専用の関数には `const` をつける習慣があります．

---

### オブジェクトの生成と使い方

```cpp
// 生成（コンストラクタに引数を渡す）
Robot robot1("Taro", 10);

// メンバ関数の呼び出し（ドット演算子）
robot1.move(3);
robot1.showStatus();
```

ポインタ経由の場合はアロー演算子 `->` を使います：

```cpp
Robot* ptr = new Robot("Jiro", 8);
ptr->move(2);
delete ptr;   // new したら必ず delete
```

---

## もう少し現実的な例：カウンター

次の章で使うパターンに近い，「状態を持つ」クラスの例です．

```cpp
#include <iostream>

class Counter
{
public:
    Counter() : count_(0) {}

    void increment() { count_++; }
    void reset()     { count_ = 0; }
    int  getCount() const { return count_; }

private:
    int count_;
};

int main()
{
    Counter c;
    c.increment();
    c.increment();
    c.increment();
    std::cout << c.getCount() << std::endl;  // 3

    c.reset();
    std::cout << c.getCount() << std::endl;  // 0

    return 0;
}
```

**状態（`count_`）をクラスの中で管理** できるのがクラスの強みです．  
グローバル変数を使わずに済むので，複数のカウンターを独立して動かすこともできます．

---

## クラスを複数のファイルに分ける

大きなプログラムでは，クラスをヘッダファイル（`.h`）とソースファイル（`.cpp`）に分けて書きます．

**Robot.h**（クラスの宣言）：
```cpp
#pragma once
#include <string>

class Robot
{
public:
    Robot(std::string name, int speed);
    void move(int steps);
    void showStatus() const;

private:
    std::string name_;
    int         speed_;
    int         distance_;
};
```

**Robot.cpp**（クラスの実装）：
```cpp
#include "Robot.h"
#include <iostream>

Robot::Robot(std::string name, int speed)
    : name_(name), speed_(speed), distance_(0)
{
    std::cout << name_ << " が起動しました" << std::endl;
}

void Robot::move(int steps)
{
    distance_ += steps * speed_;
    std::cout << name_ << " が " << steps << " 歩移動（累計: "
              << distance_ << "）" << std::endl;
}

void Robot::showStatus() const
{
    std::cout << "名前: " << name_
              << "  速度: " << speed_
              << "  累計距離: " << distance_ << std::endl;
}
```

`Robot::move(...)` の `Robot::` は「Robot クラスの `move` 関数」という意味です．

---

## ROS で登場するクラス関連の構文

ROS のコードを読むと，見慣れない書き方がいくつか出てきます．付録として整理しておきます．

### `this` キーワード

クラスの中で使える特別なキーワードで，「自分自身のオブジェクトへのポインタ」です．

```cpp
class Listener
{
public:
    Listener()
    {
        // ROS にコールバックを登録するとき，
        // 「どのオブジェクトのメンバ関数か」を this で教える
        sub_ = nh_.subscribe("topic", 10, &Listener::callback, this);
    }
    ...
};
```

### メンバ関数ポインタ `&ClassName::functionName`

`&Listener::callback` は「`Listener` クラスの `callback` 関数のアドレス」という意味です．  
ROS がコールバックを呼び出す際に必要な情報です．

### `boost::bind`

アクション通信（6章）で出てくる構文です．

```cpp
boost::bind(&MyClass::callback, this, _1)
```

「`this` のオブジェクトの `callback` を，引数 `_1` に受け取った値を渡して呼ぶ」という意味です．  
メンバ関数ポインタ + `this` をまとめて扱う C++ の仕組みです．

---

[→ 14章: クラスを使った ROS プログラミング](14_ros_with_class.md)
