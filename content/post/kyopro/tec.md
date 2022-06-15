---
title: "競プロで痒いところに手が届くかもしれない情報まとめ"
date: 2022-05-22T03:00:38+09:00
draft: false
---

### 注意
- 内容の正しさの保証はしません．
- その場しのぎの，実務で使うとしたらあまりよろしくないテクニックも載せてあります．

### define
```cpp
#define all(a) a.begin(),a.end()
```
`sort(A.begin(),A.end())`や`lower_boud(A.begin(),A.end(),x)`を，
`sort(all(A))`，`lower(all(A),x)`のように短縮できる．

### vector
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



