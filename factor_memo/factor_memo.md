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

---

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

# factor array

多言語でいうところのpushはprefix, suffixを使う。

```factor
IN: scratchpad auto-use { 2 3 } 1 prefix

--- Data stack:
{ 1 2 3 }
IN: scratchpad auto-use  4 suffix .
{ 1 2 3 4 }
```

http://docs.factorcode.org/content/word-prefix%2Csequences.html

http://docs.factorcode.org/content/word-suffix%2Csequences.html

任意の場所に挿入するにはinsert-nthが使える。

```factor
 IN: scratchpad auto-use { 1 2 3 4 }

--- Data stack:
{ 1 2 3 4 }
IN: scratchpad auto-use '[ 2.5 2 _ insert-nth ] call .

{ 1 2 2.5 3 4 }
```

http://docs.factorcode.org/content/word-insert-nth%2Csequences.html

削除はremove

```factor
IN: scratchpad auto-use 1 { 1 2 3 } remove .
{ 2 3 }
IN: scratchpad auto-use 2 { 1 2 3 } remove .
{ 1 3 }
IN: scratchpad auto-use 2 { 10 20 30 } remove .
{ 10 20 30 }
IN: scratchpad auto-use ! remove-eq
IN: scratchpad auto-use f { 1 t f "aaa" } remove-eq .
{ 1 t "aaa" }
```

http://docs.factorcode.org/content/word-remove%2Csequences.html

http://docs.factorcode.org/content/word-remove-eq%2Csequences.html

各種エレメントにアクセス

http://docs.factorcode.org/content/article-sequences-access.html

append周り

http://docs.factorcode.org/content/article-sequences-appending.html

---

# factor split

```factor
IN: scratchpad auto-use "hello,\nmy name is HAL." "\n" split .
{ "hello," "my name is HAL." }
IN: scratchpad auto-use "hello,\nmy name is HAL." "\s" split .
{ "hello,\nmy" "name" "is" "HAL." }
IN: scratchpad auto-use "hello,\nmy name is HAL." ",.\n" split .
{ "hello" "" "my name is HAL" "" }
```

http://docs.factorcode.org/content/word-split,splitting.html

---

# factor concat join

```factor
IN: scratchpad auto-use { "hello" "my" "name" "is" "SAL" } concat .
"hellomynameisSAL"
IN: scratchpad auto-use { "hello" "my" "name" "is" "SAL" } " " join .
"hello my name is SAL"
```
http://docs.factorcode.org/content/word-concat,sequences.html

http://docs.factorcode.org/content/word-join,sequences.html

---

# but-last

```factor
IN: scratchpad auto-use { "hello" "my" "name" "is" "SAL" } but-last "HAL" suffix

--- Data stack:
{ "hello" "my" "name" "is" "HAL" }
IN: scratchpad auto-use " " join write "." print
hello my name is HAL.
```
---

# factor max length string

一番長い文字列を取得してみる。

使えるパーツ。

```factor
IN: scratchpad auto-use "aaa" length .
3
IN: scratchpad auto-use 1 2 max .
2
IN: scratchpad auto-use { 1 2 3 } 0 [ max ] reduce .
3
```

文字列のリストに対してそれぞれの長さのリストをmapで作り、長さの最大値と一致する文字列だけfilterする。

```factor
IN: scratchpad auto-use { "aaa" "aaab" "aaabc" "aaacd" }

--- Data stack:
{ "aaa" "aaab" "aaabc" "aaacd" }
IN: scratchpad auto-use dup [ length ] map

--- Data stack:
{ "aaa" "aaab" "aaabc" "aaacd" }
{ 3 4 5 5 }
IN: scratchpad auto-use 0 [ max ] reduce

--- Data stack:
{ "aaa" "aaab" "aaabc" "aaacd" }
5
! _に5がセットされる
IN: scratchpad auto-use '[ length _ = ] filter .
{ "aaabc" "aaacd" }
```

---

# factor run-file

FactorのREPLにスクリプトファイルを読み込ませて実行するにはrun-fileを使う。

例えばこんなスクリプトがカレントディレクトリにあったとする。

```factor:hello.factor
USE: io
IN: hello
 
: hello ( -- )
    "Hello, Factor-Script!" print
;
 
hello
! こう書いてもOK
! MAIN: hello
```

これをREPLで読み込んで実行する。

```factor
IN: scratchpad auto-use "hello.factor" run-file
Loading hello.factor
Hello, Factor-Script!
```

シェルで単にスクリプトを実行したいなら、facgtorに食わせれば良い。

```factor
% factor hello.factor
Hello, Factor-Script!
```

これも同じ。

```factor
% factor -f hello.factor
Hello, Factor-Script!
```

---

# factor add-vocab-root

add-vocab-rootを使うと、任意のpathをボキャブラリの検索パスに追加できる。

```factor
! カレントディレクトリを検索パスに追加
IN: scratchpad "." add-vocab-root
```

USE: / USING: ... ; はボキャブラリ検索パスの中でディレクトリ名とその中においてあるファイル内のIN:宣言見ているようである。

例えば、カレントディレクトリにtestというボキャブラリを構築したい場合はまず同名のディレクトリを作る。

```
% mkdir test
```

次に、testディレクトリ内にtest.factorファイルを配置する。

```
% cd test
% vi test.factor
```

例として、test.factorの中に、doubleというワードを定義してみる。

```factor:test.factor
USING: kernel math ;
IN: test
: double ( x -- x' ) dup + ;
```

REPLに戻って、

```factor
IN: scratchpad USE: test
Loading test/test.factor
! testボキャブラリのロードが完了する

! するとdoubleワードが使える。
IN: scratchpad 10 double .
20
```
http://docs.factorcode.org/content/vocab-vocabs.loader.html

---

# factor date

今日の日付をYYYY/MM/DDの形式で表示。

```factor
IN: scratchpad auto-use USE: formatting ! formatting is out of auto-use...
IN: scratchpad auto-use now [ year>> ] [ month>> ] [ day>> ] tri "%4d/%02d/%02d" sprintf .
"2021/06/20"
```

http://hyperpolyglot.org/stack#sprintf

http://hyperpolyglot.org/stack#current-date-time

http://docs.factorcode.org/content/word-__gt__date,formatting.private.html

---

# factor bi

１つの値に対して２通りの処理（クォーテーション）を適用したそれぞれの結果が欲しい場合はbi

```factor
IN: scratchpad auto-use 10 [ 2 / ] [ 2 * ] bi

--- Data stack:
5
20
```

２つの値に１つのクォーテーションをそれぞれ適用する場合はbi@
上のスタックに続けて

```factor
IN: scratchpad auto-use [ sq ] bi@

--- Data stack:
25
400
```

１つ目の値には１つ目のクォーテーション、２つ目の値には２つ目のクォーテーションを敵湯尾するにはbi*

```factor
IN: scratchpad auto-use [ 4 * ] [ neg ] bi*

--- Data stack:
100
-400

IN: scratchpad auto-use + .
-300
```

http://docs.factorcode.org/content/word-bi,kernel.html

http://docs.factorcode.org/content/word-bi__at__,kernel.html

http://docs.factorcode.org/content/word-bi__star__%2Ckernel.html

---

# factor remove-duplicates

Lispでいうところのremove-duplicatesに相当するのはuniqueだが、ハッシュを返してくるので、valuesで要素を抜き取る。

```factor
IN: scratchpad auto-use { 8 8 2 6 } unique values .
{ 8 2 6 }
```

---

# factor http-client, http-head, download-to

```factor
 USE: http.client
IN: scratchpad auto-use USE: urls
IN: scratchpad auto-use "http://basicwerk.com/" >url http-head drop .
T{ response
    { version "1.1" }
    { code 200 }
    { message "OK" }
    { header
        H{
            { "connection" "close" }
            { "date" "Sun, 20 Jun 2021 06:58:02 GMT" }
            { "content-length" "1582" }
            { "content-type" "text/html" }
            { "server" "nginx" }
            { "last-modified" "Fri, 30 Nov 2018 05:43:45 GMT" }
            { "accept-ranges" "bytes" }
            { "etag" "62e-57bdb4852e640" }
        }
    }
    { cookies { } }
    { content-type "text/html" }
    { content-charset "UTF-8" }
    { content-encoding utf8 }
    { body "" }
}

IN: scratchpad auto-use "http://basicwerk.com/" >url http-head drop code>> .
200
```
```
% mkdir image
```

```factor
IN: scratchpad auto-use "http://basicwerk.com/image/bw_SS.png" >url "./image/bw_S.png" download-to
```

```
% ls image
bw_SS.png
```

---

