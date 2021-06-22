# はじめに

JavaScript では小数の計算が正しく出来ません。
試しに、以下の式を実行してみると…

``` javascript
0.3 - 0.2  // '0.09999999999999998'
```

0.1 になるはずが、誤差が出てしまいます。
これは丸め誤差と言って、計算処理の都合上、どうしても発生してしまうものです。

JavaScript では細かい数値計算をせず、サーバーサイドなりに任せるのがベストですが、どうしても必要な場合、数値計算ライブラリを使うのが現実的な対応になると思います。
この記事では、JavaScript で小数計算するためのライブラリ **bignumber.js** について紹介します。

# bignumber.js

[公式API](http://mikemcl.github.io/bignumber.js/)
[GitHubリポジトリ](https://github.com/MikeMcl/bignumber.js/)

英語ですが、公式APIも読みやすいです。

## インストール

npm でインストールできます。

``` 
npm install bignumber.js
```

または、[GitHubリポジトリ](https://github.com/MikeMcl/bignumber.js/) から bignumber.min.js を入手します。

```
git clone https://github.com/MikeMcl/bignumber.js.git
```

## 注意

まず最初に注意ですが、BigNumber に対して通常の演算子は使わないようにしましょう。  
例えば、以下のコードは正しく動きません。

``` javascript
// NG
BigNumber(0.3) - BigNumber(0.2);         // '0.09999999999999998'
```

誤差が出ますね。これでは BigNumber を使う意味がありません。
四則演算や比較処理などはメソッドとして提供されています。

``` javascript
// OK!
BigNumber(0.3).minus(BigNumber(0.2));    // '0.1'
BigNumber(0.3).minus(0.2);               // '0.1' (引数は数値も可)
```

## コンストラクタ

``` javascript
new BigNumber(123.4567);    // '123.4567'
BigNumber(123.4567);        // '123.4567' (newはなくてもOK)
new BigNumber('4.321e+4');  // '43210'    (指数表記も可)
new BigNumber(11, 2);       // '3'        (第2引数に基数'2'を指定)
``` 

BigNumber のインスタンスを返します。new は無くても大丈夫です。
第2引数で基数を指定することもできます。


## 四則演算

``` javascript
// 足し算
BigNumber(0.1).plus(0.2);  // '0.3'
// 引き算
BigNumber(0.4).minus(0.1); // '0.3'
// 掛け算
BigNumber(0.6).times(0.5); // '0.3'
// 割り算
BigNumber(0.15).div(0.5);  // '0.3'
// 割り算の整数部
BigNumber(5).idiv(2);      // '2'
// 割り算の剰余
BigNumber(1).mod(0.9);     // '0.1'
// べき乗
BigNumber(0.1).pow(2);     // '0.01'
```

基本的な算術演算です。
ちなみに掛け算の `times` は `multipliedBy` でもOKです。
このように、関数名が長いものには、短縮した関数が用意されています。（上記はすべて短縮したものです）

## 端数処理（丸め）

### config

端数処理の解説の前に、`config` の使い方を解説します。

`config` は、BigNumber のプロパティを操作するための関数です。
プロパティはいくつかありますが、端数処理に関係するのは `DECIMAL_PLACES` と `ROUNDING_MODE` で、
`DECIMAL_PLACES` は小数部の桁数、`ROUNDING_MODE`は丸めモード（切り上げ、切り捨てなど）が指定できます。

ちなみに、デフォルトは
`DECIMAL_PLACES : 20`
`ROUNDING_MODE : ROUND_HALF_UP(四捨五入)`
となっています。

``` javascript
BigNumber.config({
    DECIMAL_PLACES: 2,                // 小数部2桁
    ROUNDING_MODE: BigNumber.ROUND_UP // 切り上げ
});
```

上記の指定をすると、「小数部2桁を残して切り上げ」になります。
言い換えると、「小数第3位で切り上げ」ですね。

### decimalPlaces (dp)

`dp` を使用すると、指定した精度で端数処理することが出来ます。

``` javascript
// 引数なしで小数部の桁数
BigNumber(0.12345).dp();  // '5'

// 小数部の桁数(DECIMAL_PLACES)を指定。丸めモードはデフォルト(四捨五入)
BigNumber(0.12345).dp(3);  // '0.123'

// 丸めモードを変更
BigNumber.config({
    ROUNDING_MODE: BigNumber.ROUND_UP // 切り上げ
});

// config指定の丸めモードになる
BigNumber(0.12345).dp(3);  // '0.124'

// 第2引数に丸めモードを指定
BigNumber(0.12345).dp(3, BigNumber.ROUND_DOWN); // '0.123'
```

### 割り算での端数処理

割り算の結果について端数処理したい場合は、 `config` で設定するしかありません。

``` javascript
// 小数第3位を切り上げ
BigNumber.config({
    ROUNDING_MODE: BigNumber.ROUND_UP,
    DECIMAL_PLACES: 2
});

BigNumber(1.234).div(10);    // '0.13'
// BigNumber(1.234).div(10, BigNumber.ROUND_DOWN);    // このように、引数で指定することはできない
```

### (参考) clone

`config` でプロパティを変更すると、予期せぬ場所で計算結果が丸められてしまうことがあります。
ある程度処理が複雑な場合、`clone` で新しい BigNumber のコンストラクタを生成したほうが安全です。

``` javascript
RoundUpBN = BigNumber.clone({
    ROUNDING_MODE: BigNumber.ROUND_UP
});

RoundUpBN(1).div(3);  // '0.33333333333333333334'
BigNumber(1).div(3);  // '0.33333333333333333333'
```

## 比較演算

``` javascript
BigNumber(0.1).eq(0.1);  // ===
BigNumber(0.2).gt(0.1);  // >
BigNumber(0.2).gte(0.1); // >=
BigNumber(0.1).lt(0.2);  // <
BigNumber(0.1).lte(0.2); // <=
```

比較用の関数もあります。すべて結果として `boolean` を返します。上記はすべて `true` になりますね。

## 速度について

当然ですが、処理コストはかなり高いです。
試しに、1万回加算する処理を行ったところ、以下の結果になりました。

|通常の演算|BigNumberでの演算|
|---|---|
|0.06ミリ秒|35ミリ秒|

かなり大雑把な計測ですが、おおよそ500倍以上、処理時間がかかってます。
必要な処理でのみ、使うようにしましょう。
