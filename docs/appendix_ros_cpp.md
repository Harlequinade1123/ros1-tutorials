# 付録D: ROSのコードをC++として読む

ROSのノードを初めて読むと「これはROSだけの特別な書き方なのか」と感じる構文が出てきます．実はそれらはすべて**標準的なC++の機能**の組み合わせです．この付録では，ROSのコードに頻出するパターンをC++の観点から整理します．

---

## 早見表

| ROSコードで見えるもの | C++としての正体 |
|---|---|
| `ros::NodeHandle nh` | クラスのインスタンス生成 |
| `nh.advertise<std_msgs::String>(...)` | テンプレート関数の呼び出し |
| `ros::Publisher`・`std_msgs::String` | 名前空間（`::`） + クラス/構造体 |
| `msg.data = "hello"` | 構造体のメンバアクセス |
| `ConstPtr` | `shared_ptr` の `typedef`（型の別名） |
| `msg->data` | スマートポインタの `->` 演算子 |
| `subscribe(..., chatterCallback)` | 関数ポインタを引数に渡す |
| `boost::bind(&Class::f, this, _1)` | 関数オブジェクトの生成（詳細は [付録C: ROS で登場するクラス関連の構文](appendix_cpp_class.md#ros-で登場するクラス関連の構文) 参照）|

---

## 名前空間 `::`

`ros::NodeHandle` や `std_msgs::String` の `::` は**C++の名前空間の区切り文字**です．

```cpp
ros::NodeHandle nh;    // ros 名前空間の NodeHandle クラス
std_msgs::String msg;  // std_msgs 名前空間の String 構造体
```

標準ライブラリの `std::string`・`std::cout` と同じ仕組みです．  
ROS は提供するクラスを `ros`・`std_msgs`・`geometry_msgs` などの名前空間にまとめているだけです．

---

## テンプレート `<>`

```cpp
ros::Publisher pub = nh.advertise<std_msgs::String>("chatter", 10);
```

`<std_msgs::String>` の部分はC++の**テンプレート引数**です．「どの型のメッセージを扱うか」を関数に伝えています．

```cpp
// 標準ライブラリでも同じ書き方が出てくる
std::vector<int>         v;  // int 型の vector
std::vector<std::string> s;  // string 型の vector
```

`advertise<T>` は「T 型のメッセージを送る Publisher を作る」テンプレート関数として定義されています．サービス・アクションでも同じパターンが出てきます：

```cpp
nh.serviceClient<ros_tutorial::AddTwoInts>("add_two_ints")
actionlib::SimpleActionClient<ros_tutorial::CountDownAction> client(...)
```

---

## メッセージ型は自動生成された構造体

`.msg` ファイルは**C++の構造体にビルド時に自動変換**されます．

`std_msgs/String.msg`（定義）：
```
string data
```

ビルド時に次のような構造体が生成されます（概念的な表現）：
```cpp
namespace std_msgs {
  struct String {
    std::string data;
  };
}
```

だから `msg.data = "hello"` は普通の**構造体メンバアクセス**です．`.srv` や `.action` ファイルも同様に構造体へ変換されます．

---

## `ConstPtr` とスマートポインタ

コールバックの引数によく出てくる型：

```cpp
void chatterCallback(const std_msgs::String::ConstPtr& msg)
{
    msg->data;   // ドット「.」ではなくアロー「->」を使う
}
```

`ConstPtr` は `boost::shared_ptr<const std_msgs::String>` の **typedef（型の別名）** です．ROSが自動生成するコードの中で次のように定義されています：

```cpp
typedef boost::shared_ptr<const std_msgs::String> ConstPtr;
```

`shared_ptr` はスマートポインタの一種で，生ポインタ（`T*`）と同様に `->` でメンバにアクセスします．`const ... &` で受け取るのはコピーを避けるためです．

```
       生ポインタ：     T*         ptr;   ptr->member
  スマートポインタ：shared_ptr<T>  ptr;   ptr->member  ← 同じ書き方
```

---

## コールバック = 関数ポインタ

```cpp
ros::Subscriber sub = nh.subscribe("chatter", 10, chatterCallback);
```

`chatterCallback` は**関数のアドレス**を渡しているだけです．「メッセージが届いたらこの関数を呼んでください」と登録する，C++の関数ポインタそのものです．

```cpp
// C++ の関数ポインタの基本
void hello() { std::cout << "hello" << std::endl; }
void callIt(void (*f)()) { f(); }  // 関数ポインタを受け取る
callIt(hello);                     // → hello() が呼ばれる
```

クラスのメンバ関数をコールバックに使うときは `&ClassName::function` と `this` が必要になります．これは [付録C: C++ クラス入門](appendix_cpp_class.md) で解説しています．

---

この付録で挙げたパターンはパブリッシャー・サブスクライバー・サービス・アクション・NodeHandle のすべてに共通して現れます．「ROSならでは」ではなく「C++をROSが使っている」と読むと，コードの見通しがよくなります．

---

[← 付録C: C++ クラス入門](appendix_cpp_class.md)
