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

# factor io

## with-file-writer

```factor
IN: scratchpad auto-use USE: io.encodings.utf8
IN: scratchpad auto-use "sample.txt" utf8 [ { 1 2 3 } [ "Number" write number>string print ] each ] with-file-writer

%less sample.txt
Number1
Number2
Number3
```

printにはstrを渡す必要があるので、number>stringを使う。

http://docs.factorcode.org/content/word-with-file-writer%2Cio.files.html

---

## with-file-reader

```factor
IN: scratchpad auto-use "sample.txt" utf8 [ [ print ] each-line ] with-file-reader
Number1
Number2
Number3
```

http://docs.factorcode.org/content/word-with-file-reader,io.files.html

---

---

# factor regexp

http://docs.factorcode.org/content/article-regexp-intro.html

```factor
IN: scratchpad auto-use "foo bar" R/ foo/ "bar" re-replace .
"bar bar"
```
※R/の後にスペースを！

---

## regexp words
sample2.txt
```
% less sample.txt
Number 1 stack
Number 2 words
Number 3 regex
```

---

## re-contains? / matches?
ファイルの内容をフィルターしてみる。

```factor
IN: scratchpad auto-use "sample_regex.txt" ascii file-lines [ R/ regex/ re-contains? ] filter

--- Data stack:
{ "Number 3 regex" }
```

re-contains?はその行に正規表現がマッチする文字列が含まれているか？をチェックするから"Number 3 regex"を取得できる。
行にマッチさせるというコンテキストではmatches?はこの使い方はうまくいかない。

```factor
IN: scratchpad auto-use "./sample.txt" ascii file-lines [ R/ regex/ matches? ] filter .
{ }
```

matches?を使うのであれば、行全体にマッチしなければいけないから、正しくはこうなる。

```factor
IN: scratchpad auto-use "sample_regex.txt" ascii file-lines [ R/ ^.+?regex.*?$/ matches? ] filter .
{ "Number 3 regex" }
```

つまり、sed的な使い方をするならre-contains?が適切ということ。

## re-replace

http://docs.factorcode.org/content/article-regexp-options.html

デフォルトではre-replaceはグローバルマッチとなる。

```factor
IN: scratchpad auto-use "foo foo bar baz" R/ foo/ "hoge" re-replace

--- Data stack:
"hoge hoge bar baz"
```

http://docs.factorcode.org/content/word-re-replace,regexp.html

最初のマッチだけ置換するには？

```factor
IN: scratchpad auto-use "foo foo bar baz" R/ foo/ first-match

--- Data stack:
T{ slice f 0 3 "foo foo bar baz" }
IN: scratchpad auto-use "foo foo bar baz" R/ xxx/ first-match .
fIN: scratchpad auto-use "hoge" 0 3 "foo foo bar baz" replace-slice .
"hoge foo bar baz"

--- Data stack:
T{ slice f 0 3 "foo foo bar baz" }
```

first-matchはTUPLEを返すから、fromとtoの値を取り出すにはこんな感じ。

```factor
IN: scratchpad auto-use "foo foo bar baz" R/ foo/ first-match from>> .
0
IN: scratchpad auto-use "foo foo bar baz" R/ foo/ first-match to>> .
3
```

---

# Running factor

http://concatenative.org/wiki/view/Factor/Running%20Factor

factor -run=vacab, -run=hello

http://stackoverflow.com/questions/7101575/main-not-executed-by-factor-on-command-line

factor command-line options

http://docs.factorcode.org/content/article-command-line.html

---

