# factor memo

## auto-use
自動的に必要なライブラリを読み込む。

```factor
IN: scratchpad auto-use
IN: scratchpad auto-use 10 '[ 10 sq _ / ] call .
1: Note:
Added "fry" vocabulary to search path
10
```

https://docs.factorcode.org/content/word-auto-use,parser.html

---

## Wordの定義を参照する
\とseeを組み合わせる。

```factor
IN: scratchpad auto-use \ sq see
USING: kernel ;
IN: math
: sq ( x -- y ) dup * ; inline
```

http://docs.factorcode.org/content/word-see,see.html

---

## Stackの中を見るだけ
Stackの中をいじらずに見るだけなら.sを使う。

```factor
IN: scratchpad auto-use 1 2 3

--- Data stack:
1
2
3
IN: scratchpad auto-use .s
1
2
3

--- Data stack:
1
2
3
```
---

# if
ifはTOSの真偽を評価してクォーテーションを評価する。
TOSは消費される。
また、f以外はtである。
空のクォーテーションや空の配列に至ってもtである。

```factor
IN: scratchpad auto-use 1 [ "T" ] [ "F" ] if print
T
IN: scratchpad auto-use 0 [ "T" ] [ "F" ] if print
T
IN: scratchpad auto-use -1 [ "T" ] [ "F" ] if print
T
IN: scratchpad [ ] [ "T" ] [ "F" ] if print
T
IN: scratchpad { } [ "T" ] [ "F" ] if print
T
IN: scratchpad auto-use "" [ "T" ] [ "F" ] if print
T
IN: scratchpad auto-use t [ "T" ] [ "F" ] if print
T
IN: scratchpad auto-use f [ "T" ] [ "F" ] if print
F
```

---

## times
TOSの数値を回数としてクォーテーションを繰り返し評価する。

```factor
IN: scratchpad auto-use 3 [ "I might be wrong." print ] times
I might be wrong.
I might be wrong.
I might be wrong.
```

---

## each
配列の各要素に対して処理を実行したいときはeachを使う。

eachは配列の各要素をpushする動作を伴う。

```factor
IN: scratchpad auto-use { 1 2 3 } [ ] each

--- Data stack:
1
2
3
IN: scratchpad auto-use clear
IN: scratchpad auto-use 10 { 1 2 3 } [ over swap - . ] each
9
8
7

--- Data stack:
10
```

---

## map

mapはクォーテーションを適用した配列をpushする。

```factor
IN: scratchpad auto-use { 1 2 3 } [ sq ] map .
{ 1 4 9 }
```

---

## filter
filterは配列をクォーテーションでフィルターする。

```factor
IN: scratchpad auto-use { 0 1 2 } [ zero? not ] filter .
{ 1 2 }
```

http://oss.infoscience.co.jp/factor/docs.factorcode.org/content/article-cookbook-combinators.html

---

## シンボルのダイナミックスコープ
SYMBOL: で宣言したシンボルはあダイナミックスコープを持つ。
with-scopeはスコープを局所化する。

```factor
IN: scratchpad auto-use SYMBOL: foo
IN: scratchpad auto-use "outer" foo set [ "inner" foo set foo get print ] with-scope foo get print
inner
outer
```

http://oss.infoscience.co.jp/factor/docs.factorcode.org/content/article-cookbook-variables.html

---

