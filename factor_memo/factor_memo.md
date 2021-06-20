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

# io

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

# regexp

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

# array

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

# split

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

# concat join

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

# max length string

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

# run-file

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

# add-vocab-root

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

# date

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

# bi

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

# remove-duplicates

Lispでいうところのremove-duplicatesに相当するのはuniqueだが、ハッシュを返してくるので、valuesで要素を抜き取る。

```factor
IN: scratchpad auto-use { 8 8 2 6 } unique values .
{ 8 2 6 }
```

---

# http-client, http-head, download-to

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

# http download recover

例えば複数の画像URLが含まれているリストを元にダウンロードしようとしたとき、リスト中のいずれかがエラー（404 Not Foundなど
になると、HTTP request failedエラーがthrowされてそこで止まってしまう。

```factor
IN: scratchpad auto-use USE: http.client

! include Bad request
! "http://basicwerk.com/image/bw_SSxxxxx.png" is not exist.
IN: scratchpad auto-use { "http://basicwerk.com/image/bw_SSxxxxx.png" "http://basicwerrk.com/image/bw_SS.png" }

--- Data stack:
{ "http://basicwerk.com/image/bw_SSxxxxx.png"...
IN: scratchpad auto-use USE: urls

--- Data stack:
{ "http://basicwerk.com/image/bw_SSxxxxx.png"...
IN: scratchpad auto-use [ >url download ] each
```

recoverで囲ってみると、

```factor
IN: scratchpad auto-use [ >url [ download ] [ ] recover ] each

--- Data stack:
URL" http://basicwerk.com/image/bw_SSxxxxx.png"
T{ download-failed f ~response~ }
URL" http://basicwerrk.com/image/bw_SS.png"
T{ addrinfo-error f 11001 "そのようなホストは不明です。" }
```

ダウンロードが失敗するとdownload-failedというTUPLEが返されていることがわかる。
download-faild?というエラーチェックワードがあるので、これを利用する。
それと、いきなりdownloadだと、失敗している場合でも同名のファイルがローカルに作られてしまう。これを回避するために、以下の手順にしてみる。

１．一度http-headでファイルの存在を確認する。
２．エラーが投げられてdownload-failed?がtなら、移行のifの為にfを残す。
３．recoverを経過した後、ヘッダー情報のTUPLEがTOSにあればtに評価される -> download

コードに直すとこんな感じ。


```factor
IN: scratchpad auto-use { "http://basicwerk.com/image/bw_SSxxxxx.png" "http://basicwerk.com/imaage/bw_SS.png" }

--- Data stack:
{ "http://basicwerk.com/image/bw_SSxxxxx.png"...
IN: scratchpad auto-use [ >url dup [ http-head ] [ download-failed? not ] recover [ drop download ] [ 2drop ] if ] each
```

これで実際には404になる画像はスルーされてeachが続行し、200の画像はちゃんとダウンロードされる。

http://docs.factorcode.org/content/word-recover%2Ccontinuations.html

http://docs.factorcode.org/content/vocab-http.client.html

http://docs.factorcode.org/content/word-download-failed__que__%2Chttp.client.html

---

# image save

factorは最低限の動作をするのに必要なライブラリをfactor.imageというファイルに保存している。
auto-useで勝手にUSE: & addしてくれるライブラリはfactor.imageにすでに保存されていいるということ。
逆に言えば、auto-useで勝手にloadしてくれないライブラリはfactor.imageの外にあるってこと。

よく使うライブラリ（vocab）がfactor.imageに含まれていないのなら、そのライブラリをUSE:した状態（現在のREPL乗でloadが完了している状態）でsaveしてやればよい。

```factor
IN: scratchpad USING: regexp http.client ;
Loading ...

IN : scrachpad save
```

こうすると、factorはfactor.imageを更新する（ライブラリが追加されるので肥大する）。
よって、auto-useで勝手意に追加されるようになるし、factorでスクリプトを実行する場合なども動作がはやくなる。

http://oss.infoscience.co.jp/factor/concatenative.org/wiki/view/Factor/FAQ/Install/index.html

---

# low level sqlite

http://concatenative.org/wiki/view/Factor/DB
を参考にsqlite3のDBをFactorがらいじってみる。

Factor 的には Tuple で操作することを押してるみたいなので、ここで紹介するオーソドックスな SQL Statement を使ったやり方は色んな意味で low-lebel ってことになるが・・・。

まずはこれらを USING:

```factor
IN: scratchpad USING: db.sqlite db io.files io.files.temp destructors ;
! sqlite3 をよく使うなら save しておこう。
```

最低限の使い方。

```factor
! DB につなげる。
IN: scratchpad "/full-path-to/sample.db" temp-file <sqlite-db> db-open db-connection set
 
! サンプルテーブル
IN: scratchpad "CREATE TABLE foo (id INTEGER PRIMARY KEY, name, age INTEGER);" sql-command
IN: scratchpad "INSERT INTO foo VALUES (1, 'Mary', 33), (2, 'Ann', 27), (3, 'John', 69);" sql-command
 
! query
IN: scratchpad "SELECT * FROM foo;" sql-query .
{ { "1" "Mary" "33" } { "2" "Ann" "27" } { "3" "John" "69" } }
```

DBを切り離す。

```factor
IN: scratchpad db-connection get dispose
```

もう一つの方法。
with-dbをつかってopen/closeの手間をなくす。

```factor
! 使う DB 用にワードを定義しておく
IN: scratchpad 
    : with-sample-db ( quot -- )
		"/full-path-to/sample.db" temp-file <sqlite-db> swap with-db 
    ; inline
 
IN: scratchpad [ "SELECT * FROM foo;" sql-query . ] with-sample-db
{ { "1" "Mary" "33" } { "2" "Ann" "27" } { "3" "John" "69" } }
```

http://docs.factorcode.org/content/article-db-lowlevel-tutorial.html

http://oss.infoscience.co.jp/factor/docs.factorcode.org/content/vocab-db.sqlite.html

http://docs.factorcode.org/content/article-db-tuples-tutorial.html

---

# pathname

http://docs.factorcode.org/content/article-io.pathnames.html

http://docs.factorcode.org/content/vocab-io.pathnames.html


---

# branch?, deep-filter

```factor
IN: scratchpad auto-use { "q" { 1 "16" } } [ [ number? not ]  [ branch? not ] bi  and ] deep-filter .
{ "q" "16" }
! as same as
IN: scratchpad auto-use { "q" { 1 "16" } } [ string? ] deep-filter .
{ "q" "16" }
 
IN: scratchpad auto-use { "q" { 1 "16" } } [ branch? not ] deep-filter .
{ "q" 1 "16" }
! as same as flatten
IN: scratchpad auto-use { "q" { 1 "16" } } flatten .
{ "q" 1 "16" }
 
IN: scratchpad auto-use { "q" { 1 "16" } } [ branch? ] deep-filter .
{ { "q" { 1 "16" } } { 1 "16" } }
IN: scratchpad auto-use { "q" { 1 "16" } } [ branch? ] deep-filter rest .
{ { 1 "16" } }
IN: scratchpad auto-use { "q" { 1 "16" } { 8 "ui" } } [ branch? ] deep-filter rest .
{ { 1 "16" } { 8 "ui" } }
 ```

 
http://docs.factorcode.org/content/word-branch__que__,sequences.deep.html

http://docs.factorcode.org/content/word-deep-filter,sequences.deep.html

---

# member-if, remove-dup

リスト操作に関してはLisp風のモノの方が慣れているので。

```factor
! Copyright (C) 2014 Shin Nakamura.
! See http://factorcode.org/license.txt for BSD license.
USING: 
    kernel sequences fry arrays
    assocs sets
;
IN: my-seq
 
! Lisp like. 見つかった elt を car にしたリストを返す
: member-if ( ... seq quot: ( ... elt -- ... ? ) -- ... subseq )
    over swap find
    drop dup
    [ tail-slice >array ] 
    [ nip ]
    if
; inline
 
! Lisp like. ハッシュではなくリストを返す
! 順序は保証されない
: remove-dup ( seq -- seq' )
    unique values
;
```
# if files

Factorのio関連一覧。
http://docs.factorcode.org/content/vocab-io.files.html

---

# lexical vars

```factor
IN: scratchpad auto-use "5" 3 [| m n | m string>number :> m m n + . ] call
8
 
IN: scratchpad auto-use "5" 3 [| m n | m string>number :> m m ] call
 
--- Data stack:
5
```

http://docs.factorcode.org/content/article-locals-examples.html

---

# maplist

```factor
 { 1 2 3 } dup { } swap prefix swap dup length 1 - [ rest dup '[ _ _ suffix _ ] call ] times drop .
{ { 1 2 3 } { 2 3 } { 3 } }
```

---

# tuck

```factor
! ( a b -- b a b )
USE: shuffle
tuck
```

http://hyperpolyglot.org/stack

---

# subseq?

```factor
IN: scratchpad auto-use "y" "xyz" subseq? .
t
IN: scratchpad auto-use "8" "xyz" subseq? .
f
```

http://docs.factorcode.org/content/word-subseq__que__,sequences.html

---

# char, array, string

```factor
IN: scratchpad auto-use "温故知新" second .
25925
 
IN: scratchpad auto-use "温故知新" second 1array >string .
"故"
 
IN: scratchpad auto-use 1 2 "温故知新" subseq .
"故"
```

---

# cut-slice, memo

レコードセット(Array)を一定数で分割した配列を作る。

 
例: 229295 件あるデータを 49000 件毎に分割する

```factor
IN: scratchpad auto-use "SELECT id FROM items" sql-query
--- Data stack:
{ ~array~ ~array~ ~array~ ~array~ ~array~ ~array~ ~array~...
 
! 割り算の結果を integer で -> /i
IN: scratchpad auto-use dup length 49000 /i
 
--- Data stack:
{ ~array~ ~array~ ~array~ ~array~ ~array~ ~array~ ~array~...
4
 
! cut-slice ( seq n -- before-slice after-slice )
! after-slice を 49000 で割る ✕ length を割った回数分(times)  
IN: scratchpad auto-use [ 49000 cut-slice ] times
 
--- Data stack:
T{ slice f 0 49000 ~array~ }
T{ slice f 49000 98000 ~array~ }
T{ slice f 98000 147000 ~array~ }
T{ slice f 147000 196000 ~array~ }
T{ slice f 196000 229295 ~array~ }
 
! 最後にスタックに積んだ全ての Slice を Array にまとめる
IN: scratchpad auto-use { } 4 1 + [ swap suffix ] times 
 
--- Data stack:
{ ~slice~ ~slice~ ~slice~ ~slice~ ~slice~ }
```
 
http://docs.factorcode.org/content/word-cut-slice,sequences.html

http://docs.factorcode.org/content/word-__slash__i%2Cmath.html

http://docs.factorcode.org/content/word-times,math.html

 
---

# hashtables

```factor
! hashtable
IN: scratchpad auto-use SYMBOL: h
IN: scratchpad auto-use H{ } h set
 
! sample: each-index
IN: scratchpad auto-use { "one" "two" } [ 1 + swap 2array . ] each-index
{ 1 "one" }
{ 2 "two" }
 
! set-at
IN: scratchpad auto-use { "one" "two" } [ 1 + h get set-at ] each-index
IN: scratchpad auto-use h get .
H{ { 1 "one" } { 2 "two" } }
IN: scratchpad auto-use "three" 3 h get set-at
IN: scratchpad auto-use h get .
H{ { 1 "one" } { 2 "two" } { 3 "three" } }
 
! keys
IN: scratchpad auto-use h get keys .
{ 1 2 3 }
 
! values
IN: scratchpad auto-use h get values .
{ "one" "two" "three" }
 
 
! value が "t" で始まる要素のみに filter
IN: scratchpad auto-use h get keys
 
--- Data stack:
{ 1 2 3 }
 
IN: scratchpad auto-use [ h get at R/ ^t/i re-contains? ] filter
 
--- Data stack:
{ 2 3 }
 
IN: scratchpad auto-use [ dup h get at 2array ] map >hashtable
 
--- Data stack:
H{ { 2 "two" } { 3 "three" } }
```
 
http://hyperpolyglot.org/stack

http://docs.factorcode.org/content/word-each-index,sequences.html

http://docs.factorcode.org/content/vocab-hashtables.html

---

let and execute

Factorで、歴史家るスコープを作る[| bindings | ... ]という構文ではクオーテーションをスタックに積む。

http://docs.factorcode.org/content/word-[__pipe__,locals.html

よって、実行するにはcallしなければいけない。

```factor
IN: scratchpad auto-use "aaa" [| str | str print ]
 
--- Data stack:
"aaa"
[ 1 load-locals 0 get-local 1 drop-locals print ]
 
! ↑ "aaa" というリテラルとクォーテーションが積まれる。
 
! call で print が実行される
IN: scratchpad auto-use call
aaa
```
これに対し、[let ... ]というparsing wordを使った構文では、内部の処理はletを抜けた時点で即実行される（クォーテーション化されない）。

http://docs.factorcode.org/content/word-[let,locals.html

```factor
! これだけで "aaa" が print される
IN: scratchpad auto-use [let "aaa" :> str str print ]
aaa
```

そのた、[letの使い方参考。

```factor
IN: scratchpad auto-use { "112" "Nakamura" "Shin" }
 
--- Data stack:
{ "112" "Nakamura" "Shin" }
 
IN: scratchpad auto-use 
[let :> row 
    "id:" row first 2array " " join print 
    row rest "name:" prefix " " join print 
] 
id: 112
name: Nakamura Shin
 
 
IN: scratchpad auto-use { "112" "Nakamura" "Shin" }
 
--- Data stack:
{ "112" "Nakamura" "Shin" }
 
IN: scratchpad auto-use 
[let :> row 
    row first :> id 
    row rest " " join :> name 
    "id: " write id print 
    "name: " write name print 
]
id: 112
name: Nakamura Shin
```

---

# let and quotation

http://basicwerk.com/memoize/20140717155341_factor_let_and_execute.html

続き。

[| binding | ... ]はクォーテーション化されるが、[let ... ]は単なるブロックである。

だからクォーテーションを受け取るようなWordには[let ... ]は使えない。

例：

```factor
! each のコンテクストでは quotation を期待しているのでこれは error
IN: scratchpad auto-use { 1 2 3 } [let number>string :> n n print ] each
Generic word >base does not define a method for the array class.
Dispatching on object: { 1 2 3 }
 
Type :help for debugging help.
 
! 上記のコンテクストならこう
IN: scratchpad auto-use { 1 2 3 } [| n | n number>string print ] each
1
2
3
```

---

# http head

```factor
! 存在しない url(string)
IN: scratchpad auto-use "http://basicwerk.com/image/bw_SSxxxxx.png"
 
--- Data stack:
"http://basicwerk.com/image/bw_SSxxxxx.png"
 
! string -> url
IN: scratchpad auto-use >url
 
--- Data stack:
URL" http://basicwerk.com/image/bw_SSxxxxx.png"
 
! url -> head request
IN: scratchpad auto-use <head-request>
 
--- Data stack:
T{ request f "HEAD"...
IN: scratchpad auto-use dup .
    T{ request
        { method "HEAD" }
        { url URL" http://basicwerk.com/image/bw_SSxxxxx.png" }
        { version "1.1" }
        { header
            H{
                { "user-agent" "Factor http.client" }
                { "connection" "close" }
            }
        }
        { cookies V{ } }
        { redirects 10 }
    }
 
--- Data stack:
    T{ request f "HEAD"...
 
! recover でエラーをキャッチ
! エラーの際は response tuple が返されるので
! response から code を取り出す
IN: scratchpad auto-use [ http-request drop code>> ] [ response>> code>> nip ] recover
 
--- Data stack:
404
```

---

# pair, array, sequence

```factor
IN: scratchpad auto-use { 1 2 3 4 } pair? .
f
 
! pair? は2要素の array の時に t
IN: scratchpad auto-use { 1 2 } pair? .
t
 
! あくまで array
IN: scratchpad auto-use "aa" pair? .
f
 
! "aa" >array -> { 97 97 }
IN: scratchpad auto-use "aa" >array pair? .
t
 
! qw{ a a a } -> { "a" "a" "a" }
IN: scratchpad auto-use qw{ a a a } array? .
t
 
IN: scratchpad auto-use "aaa" array? .
f
 
! 文字列は sequence
IN: scratchpad auto-use "aaa" sequence? .
t
 
! string? も用意されてる
IN: scratchpad auto-use "aaa" string? .
t
```

---

# regexp white space and parsing word

regexp の parsing word (R/ など) は後ろに「1個以上」のスペースを必要とする。

きっかり1個ではなくて、1個以上のスペースが parsing の為のスペースとみなされる。

 

例えばこんな文字列があったとする。

```
"bbb aaa bbb bbb ccc"
```

先頭がスペースで始まるbbbのみを消したい場合、
もしこれをsedで書くならこれだけで良い。

```factor
% sed 's/ bbb//g' <(echo -n "bbb aaa bbb bbb ccc")
bbb aaa ccc
```

sedのs/ bbb//gはs/から次の/までが単純に正規表現とみなされるからだ。
ところがfactorで同じ書き方をすると・・・

```factor
IN: scratchpad auto-use "bbb aaa bbb bbb ccc" R/  bbb/ "" re-replace .
" aaa   ccc"
```

R/のあと、bbbまでの間にある半角スペースは「全て」wordとwordを区切るセパレータとみなされて、bbb直前のスペースは正規表現の一部とはならない。
よって、対象文字の中間にあるターゲットだけでなく、先頭にあるbbbも消されてしまう（re-replaceは常にグローバル置換を行うため）。

これを回避する方法は２つある。

```factor
! 正規表現の文字クラス(ブラケット)でホワイトスペースを表現
IN: scratchpad auto-use "bbb aaa bbb bbb ccc" R/ [ ]bbb/ "" re-replace .
"bbb aaa ccc"
 
! 文字列から正規表現オブジェクトを生成
IN: scratchpad auto-use "bbb aaa bbb bbb ccc" " bbb" <regexp> "" re-replace .
"bbb aaa ccc"
```

---

# auto-used-vocab

IN: scratchpadのREPLでコードを試して、それではスクリプトにまとめようと思うとき、USING:に列挙しなければいけないボキャブラリを闇雲に探すのは非効率的である。

factorはmanifestというオブジェクト（TUPLE）にUSING:しているvocabの一覧をauto-usedというスロットで格納しているからそれを取り出せばよい。

```factor
IN: scratchpad auto-use manifest get auto-used>> .
V{
    "math.parser"
    "qw"
    "regexp"
    "splitting"
    "lexer"
    "vocabs.parser"
}
```

http://docs.factorcode.org/content/vocab-vocabs.parser.html

http://docs.factorcode.org/content/word-current-vocab%2Cvocabs.parser.html

---