# Chapter 3. 型構築子

## 型シノニム（型の別名）
整数型や文字列型であっても、その整数型や文字列に特定の意味を持たせたい場合、以下のように別名を持たせることができる。
```Haskell
type BookID = Int
type BookTitle = String
type BookAuthors = [String]
```

上記の型シノニムを使うと、`Book` 構築子は、以下のように書けて分かりやすくなる。
```Haskell
data BookInfo = Book BookID BookTitle BookAuthors
```

`BookReview` 型も作ってみよう。<br>
```Haskell
type CustomerID = Int
type ReviewBody = String

data BookReview = BookReview CustomerID ReviewBody
```
上記のように、型構築子と値構築子を「同じ名前」にすることができる。

下記のように、「既存の型」のタプルに型シノニムを割り当てることができる。
```Haskell
type BookRecord = (BookInfo, BookReview)
```

## 型引数をとる型構築子
`data` は型構築子を定義するものですが、**型引数** を持たせることができます。
以下の例では、`a` が「型引数」となります。

```
ghci> data PointXY a = XY a a deriving Show

ghci> pt1 = XY 1 1
```
上記の `a` の部分が「型引数」と言われる部分になる。実際の使用例は以下の通り。
```
pt_1 :: Point_XY Int
pt_1 = XY 10 20
```
pt_1 は、`a` の部分に `Int` 型をとる、という意味になる。
`a` の部分に `Double` 型をとる場合、以下のように書くことになる。
```
pt_2 :: PointXY Double
pt_2 = XY 1.5 2.5
pt_3 :: PointXY Double = 2.5 3.5  -- このように１行で書くこともできる
pt_4 = 4.5 5.5 :: PointXY Double  -- このように書くこともできる
```
Haskell は、型を推論をさせることが普通であるので、以下のように書けばよい。
```
pt_5 = XY 20 30  -- この場合は、PointXY Int と推論される
pt_6 = XY 6.5 7.5  -- この場合は、PointXY Double と推論される
```

> Tips<br>
> 一般的なプログラミング言語では、型引数はジェネリクスとイメージが一致します。


## パターンマッチ
Haskell には強力なパターンマッチの機能が備わっている。
* 簡単な例
```
myNot True = False
myNot False = True
```
上記のものを `:load` すると、以下のように `not` と同じ機能が得られる。
```
ghci> myNot True
False

ghci> myNot False
True
```
また、以下の例は、リストの和を求めるものとなる。
```
sumList (x:xs) = x + sumList xs
sumList [] = 0
```
例えば、`[1,2,3] = (1:[2,3])` となることを考えれば、
```
sumList [1,2,3] = sumList (1:[2,3])
                = 1 + sumList [2,3]
                = 1 + sumList (2:[3])
                = 1 + 2 + sumList [3]
                = 1 + 2 + sumList (3:[])
                = 1 + 2 + 3 + sumList []
                = 1 + 2 + 3 + 0
                = 6
```
となると分かる。

* 少し複雑な例
```
data BookInfo = Book Int String [String]
bookID (Book id title authors) = id

myBook = Book 1 "Pure Code" ["Mike", "Bob"]
myBookId = bookID myBook
```
上記の例を `:load` すると、以下のようにできる。
```
ghci> myBookId
1

ghci> :type bookID
bookID :: BookInfo -> Int
```
また、`bookID` の定義を以下のように書くことができる。
```
bookID (Book id _ _) = id  -- title と authors は不要であるため、_ という記号で省略できる
```

* `_` の記号の使い方<br>
特定のパターンマッチに当てはまらないデフォルトの動作を指定するために、`_` を利用することができる。<br>
`sumList` は、以下のように書くことができる。
```
sumList (x:xs) = x + sumList xs
sumList _ = 0
```

## リスト
２つの値コンストラクタ `Cons`, `Nil` と、 型変数 `a` と再帰を用いて、一般的な「リスト」を作ってみると以下のようになる。
```
data List a = Cons a (List a)
            | Nil
              deriving (Show)
```
上記の `deriving (Show)` は、ghci で入力した際に、出力が表示されるようにするために書いているだけ。<br>
上記のファイルを `:load` し、"abc", "defghi", "jklm" という３つの文字列を要素とするリストを作ってみよう。
```
ghci> :load 「上記の３行が書かれたファイル名」
ghci> Cons "abc"  -- これにマッチするものはないのでエラーが表示される
エラーが表示される

ghci> Cons "abc" Nil  -- List String 型のデータが作られる（型変数 a を String と自動的に読み替えてくれる）
Cons "abc" Nil

ghci> :type it
it :: List String
```
`Nil` は、`List String` 型の値であるため、`Cons "abc" Nil` は `Cons String (List String)` にマッチする。<br>
従って、`Cons 10 Nil` によって、`List String` 型の値が作られる。<br>
続けて、以下のように入力して実験してみよう。
```
ghci> Cons "defghi" it
Cons "defghi" (Cons "abc" Nil)

ghci> :type it
it :: List String

ghci> Cons "jklm" it
Cons "jklm" (Cons "defghi" (Cons "abc" Nil))

ghci> :type it
it :: List String
```
`Cons "jklm" (Cons "defghi" (Cons "abc" Nil))` は、`List String` 型のデータである。

## エラーの処理方法
* エラーが発生したとき、処理を即時中断する方法<br>
Haskell は標準関数として `error` を持っている。`error` は文字列を出力し、評価をそこで中断する。<br>
例として、リストの２番めの要素を取り出す関数を作ってみよう。
```
takeSecond :: [a] -> a  -- この行は書かなくても正しく動作するが、この関数の動作が分かりやすいように書いている
takeSecond xs
   = if null (tail xs)
     then error "リストの長さが不足しています"
     else head (tail xs)
```
`null` は、リストが空の場合 `True` となる関数。上記のファイルを `:load` して、以下の実験をしてみよう。
```
ghci> takeSecond [1,2,3,4]
2

ghci> takeSecond [1]
*** Exception: リストの長さが不足しています
CallStack (from HasCallStack):
  error, called at test_06.hs:4:11 in main:Main
```

* Haskell で一般的に行われるエラー処理方法<br>
Haskell では、標準で備わっている `Maybe` 型を用いてエラー処理を行うことが多い。<br>
`Maybe` 型は、２つの値コンストラクタ `Just` と `Nothing` を持つもので、以下のようになっている。
```
data Maybe a = Just a | Nothing
```
上のことを初めて見ると、`Just` とは？と思うのが当然だと思う。`Just` は、エラーが発生せずに、実行が正常に終了して何らかの値が得られた、ということを `Just` とマークしているだけである。<br>
「エラーが発生するかもしれない」計算を実行するとき、その関数は、`Just 数値` を返すか、`Nothing` を返すかするようにする、というのが Haskell の習慣だ、と今は軽く考えてもらえばよい。<br>
`Maybe` を使う場合、`takeSecond` は以下のように書き換えられる。
```
takeSecond :: [a] -> Maybe a

takeSecond [] = Nothing
takeSecond xs
    = if null (tail xs)
      then Nothing
      else Just (head (tail xs))
```
上の `takeSecond` を実行すると、以下のようになる。
```
ghci> takeSecond [1,2,3,4]
Just 2

ghci> takeSecond [1]
Nothing

ghci> takeSecond []
Nothing
```
上記の結果で、`Just 2` というのは、正常に処理が終了して `2` が返された、と読むと良い。

> `Just` という表記に違和感を感じると思うので少しだけ補足<br>
> 他のプログラム言語を利用したことがあれば、「例外処理」を便利に使っている人が多いと思う。<br>
> Haskell は「例外処理」を用いずに、効率よくエラー処理をするために `Just` というものを用いるのである。<br>
> 例外処理を使いたくなった時に、初めて `Just` の存在意義が分かるので、現段階では違和感を感じたまま先に進めば良いと思う。

## ローカル変数
* `let` を利用する方法<br>
ローカル変数を `let` で宣言し、`in` の中でそのローカル変数を利用する。<br>
【注意】`let` と `in` は同じインデントにする必要がある。
```
absolute :: Int -> Int -> Int  -- ２つの整数を受け取り、絶対値を返す
absolute x y
    = let z = x - y
      in if z >= 0 then z else -z
```
以下のような結果となる。
```
ghci> absolute 2 6
4
```

* `where` を利用する方法
ローカル変数を後方で `where` で宣言する。
```
absolute :: Int -> Int -> Int
absolute x y
    = if z >= 0 then z else -z
      where z = x - y
```

## グローバル変数
関数の外側で定義された値を利用することもできる。
```
val = 10 :: Int

sub :: Int -> Int  -- １つの整数を受け取り、10 との差を返す
sub x = 
    let y = x - val
    in if y >= 0 then y else -y
```

## {} を用いた表記法
改行が多くて読みづらくなる場合は、`{}` を用いた書き方もできる。以下の２つは同じものとなる。
```
x = let a = 1
        b = 2
        c = 3
    in a + b + c
```
```
x = let { a = 1; b = 2; c = 3 }
    in a + b + c
```

## case 式
case 式は以下のように利用する。
```
data Animal = Dog | Cat | Duck
cry :: Animal -> String
cry x = case x of
          Dog -> "bowwow"
          Cat -> "meow"
          Duck -> "quack"
```

## ガード（guard）
関数に与えられた **引数で場合分け** をしたい場合、`|` を用いて場合分けを読みやすく書くことができる。<br>
階乗を求める `fact` を、`if` を使わずに、ガードを用いて書くと以下のように書ける。
```
fact :: Int -> Int
fact n
  | n < 0  = error "0 以上の数を指定してください。"
  | n == 0 = 1
  | otherwise = n * fact (n-1)
```
ガードは単なるパターンマッチの一種であるため、以下のようにパターンマッチを混合させて書くこともできる。
```
fact :: Int -> Int
fact n
  | n < 0  = error "0 以上の数を指定してください。"
  | n == 0 = 1
fact n = n * fact (n-1)
```
