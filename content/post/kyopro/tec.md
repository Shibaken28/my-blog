---
title: "競プロで痒いところに手が届くかもしれない情報まとめ"
date: 2022-05-22T03:00:38+09:00
draft: false
---

## 注意
- 内容の正しさの保証はしません．
- その場しのぎの，実務で使うとしたらあまりよろしくないテクニックも載せてあります．

## 実装
### define
```cpp
#define all(a) a.begin(),a.end()
```
`sort(A.begin(),A.end())`や`lower_boud(A.begin(),A.end(),x)`を，
`sort(all(A))`，`lower(all(A),x)`のように短縮できる．
### auto
型を推定してくれる．例えば，次のコードで，`vector`の`X`を用意したが，`double`型に変えたくなったとする．
```cpp 
    vector<int> X(10);
    for(int a:X){
        cout<<a<<endl;
    }
```
すると，`vector`の`int`と`for`文の`int`の2つを`double`に書き換えないといけなくなってしまう．そこで，`for`文の`int`を`auto`にしておくことで`vector`の型さえ決めとけば楽になる．
```cpp
    vector<int> X(10);
    for(auto a:X){
        cout<<a<<endl;
    }
```
`using`という手もある
```cpp
    using val = int;
    vector<val> X(10);
    for(val a:X){
        cout<<a<<endl;
    }
```
その他，型名がよくわかないもの(イテレータ，関数など)に使うと便利である．例えば，次を書くのはしんどいので
```cpp
    vector<int> X(10);
    _Vector_iterator<_Vector_val<_Simple_types<int>>> a = X.begin();
```
`auto`を使う．
```cpp
    vector<int> X(10);
    auto a = X.begin();
```


### 関数について
#### 値渡し
引数の中身がコピーされるため，関数内で値をいじっても元の値は変わらない．
```cpp
void foo(int a){
    a = 100;
}

int main(void){
    int x = 10;
    foo(x);
    cout<<x<<endl;
}
```

#### 参照渡し
直接参照するため，引数をコピーするコストがない．
次の例では配列の引数を戻り値っぽく使っている．`&`がないと，コピーされた`A`変更が加わるので，結局`main`関数の配列`A`は空っぽのままになってしまう．
```cpp
//約数列挙
void factor(long N,vector<long>&A){
    A.clear();
    for(long i=1;i<=N;i++){
        if(N%i==0){
            A.push_back(i);
        }
    }
}

int main(void){
    long N=60;
    vector<long> A(0);
    factor(N,A);
    for(auto&f:A)cout<<f<<" ";
    //1 2 3 4 5 6 10 12 15 20 30 60 
}
```．
参照渡しするけど，中身を書き換えたくないときは`const`を付ける．安心．
```cpp
void foo(const vector<vector<int>>&G){
    /*
        Gを改変しない処理
    */
}
```．
参照渡しの引数に変数でないもの(リテラル表記)を突っ込むとエラーが出る．
```cpp
void swapping(int &a,int &b){
    int t=a;
    a=b;
    b=t;
}

int main(void){
    int x=5;
    //リテラル表記だとエラー
    swapping(3,x);
}
```

#### ラムダ式
関数を簡単に表現できる．
```cpp
    auto func = [](int a,int b,int x){return a*x+b;};
    cout<<func(10,5,2)<<endl;
```
後述する比較関数や，`count_if`で重宝する．
また，関数を引数や返り値に持つ関数や構造体でも便利(抽象化したセグ木など)．

### コンテナ
#### unique
次のコードで重複を削除する．`unique`は，重複した要素を配列の後ろに追いやるので，その部分を`erase`で削除する．`sort`する必要がある．計算量は$O(要素数)$
```
vector<int>  A= {3,1,4,1,5};
sort(all(A));
A.erase(unique(all(A)), A.end());
```
#### fill
指定した範囲に値を代入してくれる．
```cpp
vector<int> A(5);
fill(all(A),10);
```
#### reverse
反転される．
```cpp
vector<int> A={1,2,3,4,5};
reverse(all(A));
```
`reverse_copy`はコピー先が反転する．
```cpp
vector<int> A={1,2,3,4,5};
vector<int> B;
reverse_copy(all(A),back_inserter(B));
```
#### count_if
条件を満たす要素を数える．
```cpp
vector<int> A={1,2,2,3,3,3,4,4,4,4};
cout<<count_if(all(A),[](int &a){return a==3;})<<endl;
//3

string S="coffee";
auto findE = [](char&c){return c=='e';};
cout<<count_if(all(S),findE)<<endl;
//2
```


### 比較関数
#### sort
ラムダ式が便利．例は`info`構造体の`time`メンバの昇順で並び替えをしている．
```cpp
vector<info> A={{4,1},{3,2},{1,3},{3,4}};
sort(all(A),[](info&l,info&r){return l.time<r.time;});
for(info &a:A){
    cout<<a.pos<<" "<<a.time<<endl;
}
/*
4 1
3 2
1 3
3 4
*/
```
#### priority_queue
ラムダ式を使うのは，`function`を経由するらしい．
また，`priority_queue`の`top()`メソッドは，**比較関数を使って並び替えたときに，一番後ろにくる値**が返る．
```cpp
priority_queue<info,vector<info>,function<bool(info&,info&)>> qu([](info&l,info&r){return l.time<r.time;});
qu.push(info{4,1});
qu.push(info{3,2});
qu.push(info{1,3});
qu.push(info{3,4});
while(!qu.empty()){
    info a = qu.top();qu.pop();
    cout<<a.pos<<" "<<a.time<<endl;
}
/*
3 4
1 3
3 2
4 1
*/
```
直接ラムダ式を突っ込むのが嫌な場合は，`decltype`を使う．
```cpp
auto compare = [](info&l,info&r){return l.time<r.time;};
priority_queue<info,vector<info>,decltype(compare)> qu{compare};
```

### 小数関連
#### 表示　
`setprecision`や，c言語にもある`printf`が使える．
```cpp
double r = sqrt(2);
printf("%.10lf\n",r);
cout<<fixed<<setprecision(10);
cout<<r<<endl;
//どちらも1.4142135624
```
#### 切り上げ，切り捨て，四捨五入
```cpp
cout<<floor(1.5)<<endl;//切り捨て，床関数 1
cout<<ceil(1.5)<<endl;//切り上げ，天井関数 2
cout<<round(1.5)<<endl;//小数点以下を四捨五入 2
cout<<round(1.48)<<endl;//小数点以下を四捨五入 1
```

### 範囲for文
`vector`や`set`などの要素を列挙する．他の言語の`eachfor`文にあたる．
汎用性が高い．
#### vector
```cpp
vector<int> A={3,1,4,1,5,9,2,6,5,3,5};
for(auto&a:A)cout<<a<<" ";
//3 1 4 1 5 9 2 6 5 3 5
```
#### set
```cpp
set<int> A={5,1,3,2,4};
for(auto&a:A)cout<<a<<" ";
//1 2 3 4 5
```
#### map
`map`は`pair`型で返る．
```cpp
map<int,int> A={{1,5},{4,5},{2,3},{8,1}};
for(auto&a:A)cout<<"{"<<a.first<<","<<a.second<<"} ";
//{1,5} {2,3} {4,5} {8,1}
```
構造体束縛によって次のようなこともできる．
```cpp
map<int,int> A={{1,5},{4,5},{2,3},{8,1}};
for(auto&[key,value]:A)cout<<"{"<<key<<","<<value<<"} ";
//{1,5} {2,3} {4,5} {8,1}
```

### 二分探索
範囲が含まれていたり含まれていなかったり，ややこしいのでコピペでできるように整理しておきたい．
#### lower_bound
指定した数以上の数うち，最も左にある数のイテレータを返す．
指定した数以上が存在しない場合，`.end()`のイテレータが返るため，そのまま参照する際には注意．
```cpp
vector<int> A={1,1,2,4}; //ソート済みである
for(int i=0;i<=5;i++){
    auto a = lower_bound(all(A),i);
    if(a==A.end()){
        cout<<"Aには "<<i<<" 以上の数は含まれません"<<endl;
    }else{
        cout<<"Aに含まれる "<<i<<" 以上の最小の数 "<<*a<<endl;
        cout<<"そのインデックス "<<a-A.begin()<<endl;
    }
    cout<<"Aに含まれる "<<i<<" 以上の数の個数 "<<A.end()-a<<endl;
    cout<<"Aに含まれる "<<i<<" 未満の数の個数 "<<A.size()-(A.end()-a)<<endl;
    cout<<endl;
}
```

#### upper_bound
指定した数より大きい数のうち，最も左にある数のイテレータを返す．
これまた，存在にない場合の処理に注意．
```cpp
vector<int> A={1,1,2,4}; //ソート済みである
for(int i=0;i<=5;i++){
    auto a = upper_bound(all(A),i);
    if(a==A.end()){
        cout<<"Aには "<<i<<" より大きい数は含まれません"<<endl;
    }else{
        cout<<"Aに含まれる "<<i<<" より大きいの最小の数 "<<*a<<endl;
        cout<<"そのインデックス "<<a-A.begin()<<endl;
    }
    cout<<"Aに含まれる "<<i<<" より大きいの数の個数 "<<A.end()-a<<endl;
    cout<<"Aに含まれる "<<i<<" 以下の数の個数 "<<A.size()-(A.end()-a)<<endl;
    cout<<endl;
}
```
#### xの個数を数える
`lower_bound`と`upper_bound`を合わせて使うと，ある数の個数が得られる．
具体的には，要素数からx未満の数の個数とxより大きい数の個数を引けば良い．
```cpp
vector<int> A={1,1,2,2,2,2,4}; //ソート済みである
for(int i=0;i<=5;i++){
    auto u = upper_bound(all(A),i);
    auto l = lower_bound(all(A),i);
    int a = A.end()-u;
    int b = A.size()-(A.end()-l);
    //cout<<"Aに含まれる "<<i<<" より大きいの数の個数 "<<a<<endl;
    //cout<<"Aに含まれる "<<i<<" 未満の数の個数 "<<b<<endl;
    cout<<"Aに含まれる "<<i<<" の個数 "<<A.size()-a-b<<endl;
}
```
#### mapやsetに対して
mapとsetには，`lower_bound,upper_bound`メソッドがそれぞれ用意されているので，これを使う(間違えて`std::lower_bound`を使うと，正しい値は得られない．なぜなら，`begin`から`end`までのイテレータが連続でないからである．)

### ビット演算
#### 2の冪数
2の`n`乗が欲しいとき，`1<<n`で実現できる．オーバーフローに注意．
```cpp
    cout<<(1<<30)<<endl;
    //1073741824,2の30乗
```
#### 2の冪数判定
次の論理式で2の冪数($2^k,(kは非負整数)$で表すことができる)かを判定できる．
```cpp
bool is2exp(long n){
    return (n&(n-1))==0;
}

int main(void){
    //false
    cout<<is2exp(10)<<endl;
    cout<<is2exp(98)<<endl;
    cout<<is2exp(65)<<endl;

    //true
    cout<<is2exp(32)<<endl;
    cout<<is2exp(64)<<endl;
    cout<<is2exp(2)<<endl;
    cout<<is2exp(1)<<endl;
    cout<<is2exp(0)<<endl;
}
```


### 未分類
#### Makefile
c++17の機能を使いたいので，`Makefile`でオプションを付けるように指定する．
```
#Makefile
CC=g++
CFLAGS=-Wall -std=c++17
.SUFFIXES = .cpp
objs:=$(wildcard *.cpp)
targets:=$(objs:.cpp= )

.PHONY:all
all: $(targets)
.cpp:
	$(CC) $(CFLAGS) -o $@ $<
```
#### 周囲4マスを見る
変化量を配列にする．周囲8マスにも応用できる．
```cpp
int x=2,y=2;
int dx[4]={1,-1,0,0};
int dy[4]={0,0,1,-1};
for(int i=0;i<4;i++){
    cout<<x+dx[i]<<" "<<y+dy[i]<<endl;
}
```
#### swap
型が同じであれば何でもswapしてくれる．$O(1)$．
```cpp
vector<int> A(100000,10),B(100000,20);
rep(i,1000000)swap(A,B);
```
#### 順列列挙
```cpp
vector<int> array(N);
for(int i=0;i<N;i++)array[i]=i;
do{
	A.push_back(array);
}while(next_permutation(array.begin(),array.end()));
```
#### clamp
範囲に値を収めてくれる．
```cpp
cout<<clamp(-5,0,100)<<endl; //0
cout<<clamp(104,0,100)<<endl;//100
cout<<clamp(4,0,100)<<endl;//4
```
#### hypot
ベクトルの大きさを求める．2次元と3次元で可能．
```cpp
cout<<hypot(1,1)<<endl;   //1.41421
cout<<hypot(1,1,1)<<endl; //1.73205
```

#### 部分集合全列挙
```cpp
int i=0b10001010;
cout<<(static_cast<std::bitset<8>>(i))<<" の部分集合"<<endl;
for(int T=i; ; T=(T-1)&i) {
    if(T==0)break;
    cout<<(static_cast<std::bitset<8>>(T))<<endl;
}
```

#### 基数変換
`a`進数から`b`進数に変換する．`b>10`だと出力が変になる．
```cpp
string atob(string N,long a,long b){
    long x=1;
    long R=0;//10進数
    for(int i=N.size()-1;i>=0;i--){
        R+=(N[i]-'0')*x;
        x*=a;
    }
    string S="";
    while(R>0){
        S.push_back('0'+R%b);
        R/=b;
    }
    if(S=="")S="0";
    reverse(all(S));
    return S;
}
```

## 考え方
### 余事象
- 包除原理
### 最短経路問題+α
曲がる回数を少なくする，一定回数グラフで頂点間の高速移動ができる，など．
拡張BFS，拡張ダイクストラを考える．

### 計算量
#### ある値が極端に小さい
その値を用いて全探索できないか
#### $N=10$
$O(N!)$の順列全探索
#### $N=20$
$O(2^N)$でビット全探索
#### $N=40$
半分列挙
#### $N=100$
$O(N^4),O(N^3\log X),O(N^3)$，区間dpなど
#### $N=1000$
$O(N^2\log X),O(N^2)$，dpなど
#### $N=2\times 10^5$
$O(N),O(N\log N)$
#### $N=10^9,10^18$
$O(\log N),O(1)$
ダブリング，最大公約数，桁dpなど


### 定数倍高速化
- 素因数分解をたくさんする場合，先に素数列挙しておく
  - これは定数倍高速化というのか微妙
- dpで，bool値を取っている場合，long型にして64倍高速化が可能かも

### 円環の構造
数列などの最後と最初を連続しているとみなすときに(人が円の形に並んでいる，などのシュチュエーション)，同じ数列を2つくっつけると楽である．

### 期待値，確率
- 独立に考えられないか

### ビット演算
- ビットごと独立に考えられる

### ゲーム
- Grundy数計算
- Nim
