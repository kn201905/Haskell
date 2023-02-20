# Chapter 7. 少し複雑な関数

## 複数の文を用いて定義する関数
今までは、１行で定義できる関数を扱ってきた。<br>
一般的なプログラム言語では、動作を分岐させる場合、`if` や `switch` を使うことが多いと思うが、Haskell はパターンマッチによって動作を分岐させる、ということが基本となる。<br>
パターンマッチを用いて、関数を複数に分けて作ることも多いため、その例を見ておいてほしい。
```Haskell
isLuckyNum :: Int -> String
isLuckyNum 7 = "Lucky!"
isLuckyNum _ = "Sorry..."
```

```
ghci> isLuckyNum 7
"Lucky!"

ghci> isLuckyNum 1
"Sorry..."
```
おおよそ想像できると思うが、`_` は受け取っても使わない値、ということを表している。<br>
上の例は、非常に簡単なものであるが、**上から順番にパターンマッチが実行され**、一致したところでパターンマッチは終了する、ということを理解してほしい。<br>
そのため、以下のように書くと思った通りの結果とならない。コンパイル時に **警告** が表示されるため、一度ソースファイルを作って、`:load` で読み込んで実験してみてほしい。
```Haskell
isLuckyNum :: Int -> String
isLuckyNum _ = "Sorry..."
isLuckyNum 7 = "Lucky!"
```
```
ghci> isLuckyNum 7
"Sorry..."
```

---
* よく見る階乗を求める関数を作ってみよう。これも２つに分けて定義すると以下のようになる。
```Haskell
fact :: Int -> Int
fact 0 = 1  
fact n = n * fact (n - 1)
```

参考までに、上の `fact` を１つの式でまとめて作ると以下のようになる。**ガード** を用いて、引数で場合分けをしてみたものである。
```
fact :: Int -> Int
fact n
   | n == 0 = 1
   | otherwise = n * fact (n - 1)
```

---
* `_` に慣れるために、タプルから要素を取り出す関数を作ってみよう。

```Haskell
first :: (a, b, c) -> a  
first (x, _, _) = x  

second :: (a, b, c) -> b  
second (_, y, _) = y  

third :: (a, b, c) -> c  
third (_, _, z) = z  
```

実行例
```
ghci> second (1, "hello", 4.0)
"hello"
```

## JSON を文字列に変換してみよう
Haskell に慣れるために、JSON を文字列として出力する関数を書いてみよう。

```Haskell
module JValue (JValue(..)) where
   
data JValue = JString String
            | JNumber Int
            | JBool Bool
            | JNull
            | JObject [(String, JValue)]
            | JArray [JValue]
              deriving (Show)
```

先頭行の `module JValue (JValue(..))` は、このファイルで定義された `JValue` という名前を、
他のファイルでも使えるようにする（エクスポートする）、という意味となる。<br>

`module` `モジュール名` (`エクスポートするもの`) の形で利用する。

`JValue(..)` と書くことによって、`JValue` に関する値コンストラクタの全てをエクスポートする、という意味になる。<br>
`where` は、ここ以降で `JValue` が定義されていく、という意味。<br>

`module` によるエクスポートは、ファイルの先頭に書くこと。<br>

もし、以下のように書くと、モジュール内の全ての名前をエクスポートすることになる。
```Haskell
module JValue where
```

逆に、以下のように書くと、モジュール内の全ての名前をエクスポート「しない」ことになる。
```Haskell
module JValue () where
```

## ソースファイルのコンパイル法（オブジェクトファイルの生成）
`ghc` を用いて、JValue.hs をコンパイルしてみよう。
```
ghc -c JValue.hs
```

もし、ghci に接続したまま（docker 等を利用している場合）コンパイルする場合は、以下のようにすればよい。
```
ghci> :!ghc -c JValue.hs
ghci> :!ls
JValue.hi  JValue.hs  JValue.o
```

`-c` オプションは、オブジェクトコードのみを生成する。<br>
もし、ソースファイル内に `main` が定義されている場合、`-c` オプションを外せば実行ファイルを生成することができる。<br>

コンパイルに成功すれば、拡張子が `.hi` と `.o` の２つのファイルが生成される。<br>
`.hi` ファイルにはモジュールからエクスポートされた名前が記録される。<br>
`.o` は、一般的なオブジェクトファイル。

## 試しに、実行ファイルを生成してみよう
以下の内容を持つ `Main.hs` ファイルを作る。
```Haskell
module Main (main) where  -- ghc は、Main モジュール内にある main を実行の開始点として実行ファイルを生成する

import JValue

main = print (JObject [("number", JNumber 1), ("bool", JBool False)])
```

`Main.hs` と `JValue.hs` の２つがあるディレクトリで、以下のコマンドを実行する。
```
ghc -o test Main.hs JValue.hs  -- -o オプションは、出力する実行ファイルの名前を指定するもの
```

ghci の下で実行ファイルを生成して実行すると、以下のようになる。
```
ghci> :!ls
Main.hs  JValue.hs
ghci> :!ghc -o test Main.hs JValue.hs
...
ghci> :!ls
Main.hi  Main.hs  Main.o  JValue.hi  JValue.hs  JValue.o  test
ghci> :!./test
JObject [("number",JNumber 1),("bool",JBool False)]
```
`JObject [("number",JNumber 1),("bool",JBool False)]` の生成に成功し、それが表示されていると分かる。<br>
もし、どこかにエラーがあった場合、`JObject [("number",JNumber 1),("bool",JBool False)]` は表示されない。

## JValue を String に変換する
以下の JSON を `JValue` で作って、それを `String` に変換してみよう。
```
ryzen_7900X = {"Architecture": "Zen 4", "CPU Core", 12}
```

上の JSON を、JValue で書き直すと、以下のようになる。
```
ryzen_7900X = JObject [("Architecture", JString "Zen 4"), ("CPU Core", JNumber 12)]
```

上のようにすると、`ryzen_7900X` は、`JValue` 型の値になる。<br>
JSON の key-value ペアを作る場合、key は常に文字列でなければならないので、key は `String` で良いが、value には複数の形式が認められているため、value は `JString` などの **値コンストラクタ** を用いて値を生成している。
<br><br>

以下のファイルを `test.hs` とする。
```
import JValue

ryzen_7900X = JObject [("Architecture", JString "Zen 4"), ("CPU Core", JNumber 12)]
```

`JObject` が、正しく解釈されていることを確かめてみよう。
```
ghci> :load test.hs
...
ghci> ryzen_7900X
JObject [("Architecture", JString "Zen 4"), ("CPU Core", JNumber 12)]
```

上記の `ryzen_7900X` を `String` に変換する関数を作ってみよう。<br>
まずは、以下のファイルを `test.hs` として実行してみよう。
```Haskell
import JValue

showJValue :: JValue -> String
showJValue (JObject o) = "{" ++ (showJPair $ o !! 0) ++ "}" --「!! n」は、リストの n番目の要素を取り出す操作
   where
   showJPair :: (String, JValue) -> String
   showJPair (k, v) = k ++ ": " ++ f v
      where f (JString s) = s  -- 今は、JValue の内、JString だけに対応する

ryzen_7900X = JObject [("Architecture", JString "Zen 4"), ("CPU Core", JNumber 12)]
test = putStrLn $ showJValue ryzen_7900X
```

```
ghci> :load test.hs
...
ghci> test
{Architecture: Zen 4}  -- index 0 の要素が、正しく変換されたと分かる
```
<br>

このままでは、`JString` しか表示ができないので、以下のように変更してみよう。

```Haskell
import JValue

showJValue :: JValue -> String
showJValue (JString s)   = show s  -- show は書いても書かなくてもプログラムの実行は可能
showJValue (JNumber n)   = show n
showJValue (JBool True)  = "true"
showJValue (JBool False) = "false"
showJValue JNull         = "null"

showJValue (JObject o) = "{" ++ (showJPair $ o !! 0) ++ "}"
   where
   showJPair :: (String, JValue) -> String
   showJPair (k, v) = show k ++ ": " ++ showJValue v
     
ryzen_7900X = JObject [("Architecture", JString "Zen 4"), ("CPU Core", JNumber 12)] 
test = putStrLn $ showJValue ryzen_7900X
```

`show` を余分に入れている理由は、JSON の場合、文字列は `""` でくくらなければならないので入れているだけ。今は単なる練習であるため、`show` は書かなくても良い。`show` は書かなくても、プログラムの実行は可能である。
```
ghci> :load test.hs
...
ghci> test
{"Architecture": "Zen 4"} -- show を利用すると、文字列が "" でくくられて表示される
```

---
今のままでは `JObject` 内のリストの最初の項目しか表示できないため、`map` を使って、リスト内の全ての項目に `showJPair` を適用し、全ての項目を文字列に変換してみよう。
```Haskell
showJValue (JObject o) = "{" ++ (concat $ map showJPair o) ++ "}"
   where
   showJPair :: (String, JValue) -> String
   showJPair (k, v) = show k ++ ": " ++ showJValue v
```

`concat` で単純に連結しているだけなので正しくは表示できていないが、リスト内の項目が２つとも表示できていることが確認できる。
```
ghci> test
{"Architecture": "Zen 4""CPU Core": 12}
```

---
最後に `concat` の連結を `,` を用いた連結にすればよい。`Data.List` モジュールにある `intercalate` を利用すれば良いが、今は練習のため以下のような関数を１つ用意しよう。<br>
以下の４行をソースファイルに書き加える。

```Haskell
concatComma :: [String] -> String
concatComma [] = ""
concatComma [x] = x
concatComma (x:xs) = x ++ ", " ++ concatComma xs
```

そして、`concat` を `concatComma` に変えれば、`ryzen_7900X` の正しい表示が得られる。以下のように１行だけ書き換えて実行してみよう。

```Haskell
showJValue (JObject o) = "{" ++ (concatComma $ map showJPair o) ++ "}"
```

```
ghci> test
{"Architecture": "Zen 4", "CPU Core": 12}
```
<br>

`JArray` も表示できるようにするには、以下の１行を加えれば良いだけである。
```Haskell
showJValue (JArray a) = "[" ++ (concatComma $ map showJValue a) ++ "]"
```



