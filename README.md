# Javascript初学者（初級者）から中級者へのステップアップに必要な基本 Part2

タイトルが大分雑いことになっていますが、「Javascript初学者（初級者）から中級者へのステップアップに必要な基本」で書いていなかった+αを書いていこうと思います。今回は少しECMAScript5, 6にも少し触れていきます。相も変わらず自分の理解度チェックも兼ねていきます。

## strictモード

処理の前に
```
'use strict';
```
と書くとstrictモードになります。strictモードは「厳密なコーディング」ができるようになる設定です。エラーをより厳密に扱うことができるようになる、と言った方が正しいかもしれません。Javascriptは色々な意味で非常に寛容な言語ですが、それにより繁雑なコードになることが往々にしてあります。より最適なコーディングをするためにある種の制限をかけるのがstrictモードという理解で大丈夫です。より最適なコードはパフォーマンスや可読性、保守性を向上させます。

### strictモードの罠

「strictモードいいじゃん！使おう！」というのは実に良い考えだと思います。ただし、外部ライブラリや他人が書いたコードを利用する時に、そのコードがstrictモードでは動かない（制限に引っ掛かる）コードであった場合は...動かなくなる可能性があります。となると、ライブラリや他人の書いたコードに依存してstrictモードを使うしかないの？となりますが、strictモードは「処理前に書けば効力を発揮」します。つまり...
```
var obj = function (hoge, piyo) {
    'use strict';
    this.hoge = hoge;
    this.getHoge = function () {
        ret = 'getHoge'; // error
        return ret;
    }
}
o = new obj();
console.log(o.getHoge());
```
上記のコードで言えば、objオブジェクト内ではstrictモード、グローバル領域ではstrictモードではない、となります。strictモードでは必ずvar, let, constで定義する必要があります。この他にもstrictモードで挙動が変わることはありますが、都度説明していこうと思います。

### エラーの黙殺

strictモードでは曖昧な動作やエラーの黙殺は許可されません。また、エラーの原因になり得る可能性のある演算子やプロパティへの操作も許可されません。例えばdelete演算子等は許可されていません。

### その他の制限

evalで定義した変数はeval内でしか使えないとかwithが禁止とかありますが、細かいところはあまり実装上に出てこないと思うので割愛します。


## thisの振る舞い

これがなかなかに癖者なわけですが、ひとつひとつ動かしながら確認していけば理解できるはずです。さっそくやっていきましょう。thisの振る舞いを理解するとコードの質がかなり変わってくるのでここはしっかり覚えておきたいところです。

### 関数からの参照

まずはコードを...
```
let func = function () {
    console.log(this); // Object [global] { ... }
}
func();
```
Globalオブジェクトが返ってきているのがわかりますね。実行環境によってGlobalオブジェクトの中身は変わりますがGlobalオブジェクト自体が返ってくることに変わりはありません。

### オブジェクト内からの参照

こちらもまずコードを...
```
let obj = {
    val: '999',
    inner1: function () {
        console.log(this); // { val: '999', inner: [Function: inner] }
        this.val = 777;
    }
}
obj.inner1();
console.log(obj.val); // 777
```
inner1内で参照しているthisはそのメソッドが属するオブジェクトを指していますね。直感的ですね。少しコードを変えてみましょう。
```
let obj = {
    val: '999',
    inner1: function () {
        console.log(obj.val); // 999
        console.log(this); // { val: '999', inner: [Function: inner] }
        this.val = 777;
        var inner2 = function () {
            console.log(this.val); // undefined
            console.log(this); // Object [global] { ... }
        }
        inner2();
    }
}
obj.inner1();
console.log(obj.val); // 777
```
inner2から参照したthisはオブジェクトではなくGlobalを指していますね...この辺りがthisの振る舞いの複雑なところです。
ここで覚えておく必要があるのは「メソッド」なのか「関数」なのかです。メソッドから参照した場合は自身が属するオブジェクト、関数から参照した場合はGlobalオブジェクト、と覚えておきましょう。それじゃあ関数からオブジェクト内の値を使いたい時は引数でもらうしかないの？となりそうですが、こんな逃げ方があります。
```
let obj = {
    val: '999',
    inner1: function () {
        let _this = this;
        var inner2 = function () {
            console.log(_this); // { val: '999', inner1: [Function: inner1] }
        }
        inner2();
    }
}
obj.inner1();
```
thisを逃して関数から参照できるようにしています。スコープチェーンが理解できていると「あーなるほど」となるかと思います。

### コンストラクタからの参照

コンストラクタからの参照時は自身のインスタンスを参照します。
```
let obj = function (v) {
    this.val = v;
    this.func = function () {
        return this;
    }
}
let o = new obj('str');
o.val = 'hoge';
console.log(o.func()); // obj { val: 'hoge', func: [Function] }
```
インスタンスを参照するのでもちろんメンバの値が変更されれば変更された値を参照します。

### call, applyからの参照

callメソッド、applyメソッドは第一引数に指定されたオブジェクトを「thisとして束縛する」メソッドになります（ちょっと乱暴な言い方ですが）。callメソッドとapplyメソッドの違いについては第二引数以降を配列で受け取れるかどうかぐらいの違いです。call, applyメソッドは関数に付与される標準プロパティです。
```
let obj1 = {
    val: 1
}
let obj2 = {
    val: 2
}
let obj3 = {
    val: 3
}
let sum = function (v1, v2) {
    return this.val + v1 + v2;
}
// call
console.log(sum.call(obj1, 2, 3)); // 6
console.log(sum.call(obj2, 3, 4)); // 9
console.log(sum.call(obj3, 4, 5)); // 12
// apply
console.log(sum.apply(obj1, [2, 3])); // 6
console.log(sum.apply(obj2, [3, 4])); // 9
console.log(sum.apply(obj3, [4, 5])); // 12
```

### bindからの参照

bindもcall, applyと同じく「束縛する」メソッドです。ただしcall, applyと使い方は異なります。
```
let obj1 = {
    val: 1
}
let obj2 = {
    val: 2
}
let obj3 = {
    val: 3
}
let sum = function (v1, v2) {
    return this.val + v1 + v2;
}
let sum1 = sum.bind(obj1);
console.log(sum1(2, 3)); // 6
let sum2 = sum.bind(obj2);
console.log(sum2(2, 3)); // 7
let sum3 = sum.bind(obj3);
console.log(sum3(2, 3)); // 8
```
bindはthisだけではなく引数も束縛することができます。
```
let sum = function (v1, v2) {
    return v1 + v2;
}
let sum1 = sum.bind(null, 1);
console.log(sum1(2)); // 3
console.log(sum1(2, 3)); // 3
let sum2 = sum.bind(null, 1, 2);
console.log(sum2()); // 3
console.log(sum2(2, 3)); // 3
```
不要な引数は無視される形になります。thisを束縛する必要がない場合はnullを引数に与えればOKです。

### アロー関数からの参照

ついでにES6で実装されたアロー関数から参照した時のthisの挙動も覚えておきましょう。アロー関数から参照した場合は、「アロー関数が定義されたスコープのthisが参照される」といった形になります。
```
let obj = {
    val: 'hoge',
    funcO: function () {
        let funcI1 = function () {
            console.log(this);
        }
        let funcI2 = () => {
            console.log(this);
        }
        funcI1(); // Object [global] { ... }
        funcI2(); // { val: 'hoge', funcO: [Function: funcO] }
    }
}
obj.funcO();
```
メソッドの時と同じ振る舞いですね。

## 関数について

関数？いやいや、もう十二分です。となりそうですが、関数についてもう少しだけ補足しておきます。

### 関数巻き上げについて
以下のコードと実行結果を見てください。
```
console.log(func1()); // Hello func1.
function func1() {
    return 'Hello func1.';
}

console.log(func2()); // error
let func2 = function () {
    return 'Hello func2.'
}
```
2つの関数を定義しており、定義前に呼んでいるのでエラーになりそうですがfunction命令で定義した方はエラーになっていません。ここからわかることは「function命令で定義された関数は巻き上げされる」「関数リテラルで定義された関数は巻き上げされない」ということですね。使いどころによってはエラーになる可能性があるのでこの点は注意しておきましょう。予め定義されている関数へのアクセスか、変数を経由してしかアクセスできない関数へのアクセスか、の違いですね。

### オーバーロード

Javascriptにオーバーロードあるの！？となった方すいません。Javascriptにおいて関数はシグネチャを持ちません。が、それっぽいことはできます。以前ちょっとだけ出てきたargumentsオブジェクトを利用します。
```
let func = function (v1, v2) {
    switch (arguments.length) {
        case 0:
            return 'arguments.length(0)';
            break;
        case 1:
            return 'arguments.length(1)';
            break;
        case 2:
            return 'arguments.length(2)';
            break;
        default:
            return 'error.';
            break;
    }
}
console.log(func()); // arguments.length(0)
console.log(func(1)); // arguments.length(1)
console.log(func(1, 1)); // arguments.length(2)
console.log(func(1, 1, 1)); // error.
```
ちょっと無理矢理感あります。

### Getter/Setter

JavascriptにもGetter/Setterはあります。
```
let obj = {
    _v: 0,
    get v() {
        return this._v;
    },
    set v(v) {
        this._v = v;
    }
}

console.log(obj.v); // 0
obj.v = 99;
console.log(obj.v); // 99
console.log(obj._v); // 99
```
できると言ってもカプセル化できるとは言っていない（ダメ）。ということでカプセル化をしてみようと思います。
```
let obj = function (v) {
    let _v = v;

    this.setV = function (v) {
        _v = v;
    }

    this.getV = function () {
        return _v;
    }
}

let o = new obj('val.');
console.log(o.getV()); // val.
o.setV('hoge.');
console.log(o.getV()); // hoge.
console.log(o._name); // undefined
```
こんな感じです。標準のGetter/Setterを使っていないので是非はありますが、クラスライクにカプセル化したい場合は、この方法も一手かと思います。即時関数＋クロージャでも似たようなことはできますが、クラスライクにカプセル化する時はこうするのが一番かなと思います（ES5の段階では、の話です）。

## プロパティ

自由気ままなJavascript、風が吹けば変わるオブジェクト。そんなオブジェクトのプロパティについて少し細くしていこうと思います。

### 存在確認

hogeオブジェクトにpiyoプロパティがある時に処理を行いたい。そういう時に以下のようなコードは書くべきではありません。
```
if(hoge.piyo) { ... }
```
0, NaN, Nullもfalseとして認識されてしまうので思わぬ動作をしてしまう可能性があります。プロパティの有無を確認したい場合は「in演算子」または「hasOwnPropertyメソッド」を利用します。「in演算子はオブジェクト（prototypeを含め）」がプロパティを有しているか、「hasOwnPropertyメソッドはオブジェクト（prototypeを除く）」がプロパティを有しているか判定します。
```
let hoge = {
    piyo: 'piyopiyo'
}
// 1
// 2
// 3
if ('piyo' in hoge) console.log('1');
if ('toString' in hoge) console.log('2');
if (hoge.hasOwnProperty('piyo')) console.log('3');
if (hoge.hasOwnProperty('toString')) console.log('4');
```
挙動は厳密に異なるので使い所は注意しましょう。

### 属性

実はJavascriptのオブジェクトプロパティは属性を持っているのです...最近知りました。Javascriptの中でも大分濃ゆいところですが、難しいことじゃないのでササッと覚えて行きましょう。

#### データプロパティ

見たままそのままデータを格納するためのプロパティです。データプロパティはvalueとwritableの属性を持ちます。とりあえずこんなプロパティがあるんだとだけ覚えておいてください。

#### アクセサプロパティ

Getter/Setterのことです。アクセサプロパティにはgetとsetのプロパティが存在します、当たり前ですね。それぞれのプロパティには各関数が格納されます。

#### 共通プロパティ

データプロパティ及びアクセサプロパティ共にenumerableとconfigurableというプロパティが存在します。enumerableは「プロパティが走査可能か」、configurableは「プロパティの削除、属性の変更を可能とするか」をbooleanで定義します。走査可能＝列挙可能とするとわかりやすいかもしれないですね。configurableの属性の変更ですが、writableはこれに該当しないので注意が必要です。configurableをfalseにしたけど書き換えできてしまうので正常な動作となります。

#### 属性の操作

それでは各プロパティの属性を実際に操作してみましょう。属性操作には2つの方法があります。

##### definePropertyメソッド

扱い方は簡単です。先の属性を理解できていれば問題ないですね。
```
let obj = {
    v: 'v',
    i: 0,
    m: 'm'
}

// オブジェクトのプロパティを列挙
for (let p in obj) {
    console.log(p); // v i m
}

// プロパティiを列挙＆書き換え不可にする
Object.defineProperty(obj, 'i', {
    enumerable: false,
    writable: false
});

// オブジェクトのプロパティを列挙
for (let p in obj) {
    console.log(p); // v m
}

// 値を格納してみようとする
obj.i = 99;
console.log(obj.i); // 0
```

##### definePropertiesメソッド

こちらは名前の通り、複数のプロパティの属性を変更できます。ただしこちらは注意が必要です。理由は「プロパティディスクリプタに指定していない属性は全てfalseとして定義される」ことにあります。使う場合は必ずプロパティディスクリプタに全ての属性を指定する等、コーディング規約に記載しておくべきです。

#### プロパティディスクリプタの取得

getOwnPropertyDescriptorメソッドでプロパティディスクリプタを取得できます。そのまんまですね。
```
let obj = {
    v: 'v',
    i: 0,
    m: 'm'
}

Object.defineProperty(obj, 'i', {
    enumerable: false,
    writable: false
});

o = Object.getOwnPropertyDescriptor(obj, 'i');
console.log(o.value); // 0
console.log(o.writable); // false
console.log(o.enumerable); // false
console.log(o.configurable); // true
```

## オブジェクト

オブジェクトは野生動物でも何でも無いですが、Javascriptの自由さによって姿形が四季折々変わる可能性があります。風景は四季折々変わって良いのですが、オブジェクトはあまり変わって欲しくありません。ということでオブジェクトの保護について説明していこうと思います。ついでに継承もちょっと説明します。おまけ程度ですけど。

### オブジェクトの拡張禁止

オブジェクトの拡張禁止にはpreventExtensionsメソッドを利用します。このメソッドは拡張禁止とは言っているものの新しいプロパティの追加のみ禁止とする仕様です。拡張可能か調べる場合はisExtensibleメソッドを利用します。
```
let obj = {
    v: 'v'
}
console.log(Object.isExtensible(obj)); // true
obj.i = 0;
Object.preventExtensions(obj);
obj.m = 'm';
console.log(Object.isExtensible(obj)); // false
for (p in obj) {
    console.log(p); // v i
}
```
strictモードであれば「obj.m = 'm';」の時点でエラーになります。一度拡張禁止にすると再度拡張できるよう元に戻すことはできません。

### オブジェクトの封印

封印は拡張禁止で行った「プロパティの追加」以外に、「プロパティの削除」と「プロパティの属性変更」ができなくなります。要するにプロパティの読み書きのみができる状態になります。拡張禁止と同じで元に戻すことはできません。オブジェクトの封印はsealメソッドで行い、封印状態の確認はisSealedメソッドで行います。
```
let obj = {
    v: 'v'
}
console.log(Object.isSealed(obj)); // false
Object.seal(obj);
console.log(Object.isSealed(obj)); // true
// プロパティの追加や削除は実際に試してみてください。
```

### オブジェクトの凍結

凍結は封印に加え、「値の書き込み」もできなくなります。要するに参照しかできない状態になります。システムで定義するようなデータオブジェクトを用意する場合は、有効ですね。凍結させるにはfreezeメソッドを利用し、凍結状態の確認はisFrozenメソッドで行います。
```
let obj = {
    v: 'v'
}
console.log(Object.isFrozen(obj)); // false
Object.freeze(obj);
console.log(Object.isFrozen(obj)); // true
// プロパティの追加や削除、値の変更は実際に試してみてください。
```
オブジェクトの保護については以上です。そこまで難しくなかったですね。お察しの通りオブジェクトの保護は各メソッドを経由して属性を変更させています。属性の理解を深めるとこの辺りもスッと入ってくるかもしれません。

### オブジェクトの継承

最後にオブジェクトの継承です。オブジェクトの継承ってプロトタイプチェーンでやるんじゃなかったっけ？？となると思いますが、EC5からはcreateメソッドで継承できるんです。createメソッドは任意のプロトタイプオブジェクト及びプロパティを持ったアブジェクトを生成するメソッドです。
```
let objParent = {
    v: 'v',
    getV: function () {
        return this.v;
    }
}
let objChild = Object.create(objParent, {
    i: {
        value: 0
    },
    m: {
        value: 'm'
    }
});
console.log(objParent.m);
console.log(objChild.m);
let o = Object.getOwnPropertyDescriptor(objChild, 'm');
/**
 * {
 *   value: 'm',
 *   writable: false,
 *   enumerable: false,
 *   configurable: false
 * }
 */
console.log(o);
```
definePropertiesメソッドと同じでプロパティディスクリプタを指定しないとfalseで設定されるので、その点は注意が必要です。継承するプロトタイプオブジェクトが無い場合はnullを指定します。
```
let o = Object.create(null);
```
何のプロトタイプオブジェクトも持たないオブジェクトです。一体何に使うのかと言えば...何でしょう。プロトタイプオブジェクトと名前がぶつかる可能性がある場合とかに使えそうです。jsonをパースして格納したりhashで使ったり、等々。実際の開発で使ったことはありません。

## 最後に

先に書いた内容はあくまでも知っておくと良いよというものでこれを上手に使えて上級者なのかなと思います。とは言えES6で書くとここに書いたことの何割が役に立つか...という感じですが基本は大事だよねということで。
