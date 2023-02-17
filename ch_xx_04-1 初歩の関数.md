# Chapter 4. 初歩の関数

## 文字列を分割するプログラム
Haskell のプログラムを書く簡単な練習として、文字列を分割するプログラムを作ってみよう。<br>
改行コード `\r`, `\n`, `\r\n` を区切り文字として、文字列を分割する。<br>
例えば、`"abcd\refg\r\nhijk"` という文字列が入力されたら、`["abcd", "efg", "hijk"]` が出力となる。

このプログラムを作るために、リスト操作の標準関数 `break` を利用しよう。<br>
`break` のシグネチャを見てみよう。
```
ghci> :i break
break :: (a -> Bool) -> [a] -> ([a], [a])
```
まず、上の意味が分かるようになろう。<br>
`break` は `a -> Bool` と `[a]` を受け取り、`([a], [a])` を返すという意味になる。<br>

`a` は、もし、文字列のリストに対して `break` を使えば `a = Char` になるし、
整数のリストに対して `break` を使えば `a = Int` に自動的に読み替えられる、という意味である。<br>
今は、文字列の分割をしようとしているので、以下では `a = Char` と読み替えながら考えていこう。

`Char -> Bool` というのは、`Char` を受け取り、`Bool` を返す関数を表す。
ここでは、以下のようなものを考えよう。
```
isLineBreak :: Char -> Bool
isLineBreak c = c == '\r' || c == '\n'
```
> Tips<br>
> ghci で、複数行を入力したいときは、`:{` と入力するとよい。<br>
> そうすると複数行の入力モードとなるため、`isLineBreak` のような２行のものを簡単に入力できる。<br>
> 複数行の入力を終えたら `:}` と入力すると、元の１行入力モードに戻る。

`isLineBreak` は、`\r` か `\n` を受け取ったら `True` を返す関数である。<br>
その機能を試してみよう。
```
ghci> isLineBreak 'a'
False

ghci> isLineBreak '\r'
True
```

次に、`break` の機能を試してみよう。
```
ghci> break isLineBreak "abc\rde\r\nfghijk"
("abc","\rde\r\nfghijk")
```
これで `break` の機能の見当はついたと思う。<br>
`Char -> Bool` の関数でリストの先頭からチェックしていき、`True` となったところでリストを分割し、
タプルにして返す、というのが `break` の機能である。

上記の意味が分かれば、`break :: (a -> Bool) -> [a] -> ([a], [a])` の意味が理解できると思う。

以下の内容をファイルに書いて、`:load` してみてほしい。
```
isLineBreak :: Char -> Bool
isLineBreak c = c == '\r' || c == '\n'

splitLines :: [Char] -> [[Char]]  -- 文字列を受け取り、「文字列のリスト」を返す、という意味
splitLines cs =
    let (first, second) = break isLineBreak cs
    in first : [second]
```
ここで、以下のことを実行して結果を確かめてみてほしい。
```
ghci> splitLines "abcd\refg\r\nhijk"
["abcd","\refg\r\nhijk"]
```

`\r` で分割されたことが分かると思う。<br>
`in first : [second]` の `:` は、リストに先頭要素を加えるものだったことを思い出してほしい。<br>
`["\rde\r\nfghijk""]` というリストの先頭に `"abc"` を加えたものが返された、ということになる。<br>

ここまで出来たら、後は `second` の部分も再帰的に分割していけば良い。<br>
もう少し練習のために、以下のものを試してみよう。
```
isLineBreak :: Char -> Bool
isLineBreak c = c == '\r' || c == '\n'

splitLines :: [Char] -> [[Char]]
splitLines cs =
    let (first, second) = break isLineBreak cs
    in first : case second of
                  ('\r':rest) -> [rest]
                  _ -> ["???"]
```
`case` は「式」であるため、値を返す。上の場合、`[rest]` か `["???"]` を返す。<br>
`case` から返されたものは文字列のリストであるため、そのリストの先頭に `first` を挿入する、という流れになる。<br>
`('\r':rest)` は、`\r` から始まる文字列という意味になるため、上のものを実行すると、以下のようになる。
```
ghci> splitLines "abcd\refg\r\nhijk"
["abcd","efg\r\nhijk"]

ghci> splitLines "abc\nd"
["abc","???"]
```

`break` に区切り文字を含まない文字列を与えると以下のようになる。
```
ghci> break isLineBreak "abcde"
("abcde","")

ghci> break isLineBreak ""
("","")
```

以上のことを鑑みて、`rest` を同じ方法で分割するために、自分自身を呼び出して分割を進めるようにすると、以下のようになる。
```
isLineBreak :: Char -> Bool
isLineBreak c = c == '\r' || c == '\n'

splitLines :: [Char] -> [[Char]]
splitLines cs =
    let (first, second) = break isLineBreak cs
    in first : case second of
                  ('\r':'\n':rest) -> splitLines rest
                  ('\r':rest) -> splitLines rest
                  ('\n':rest) -> splitLines rest
                  [] -> []
```
これで完成である。
```
ghci> splitLines "abcd\refg\r\nhijk"
["abcd","efg","hijk"]
```

## 中置記法
Haskell では、関数が２つ以上の引数をとる場合、その関数名を引数の間に置いて書くことができる。<br>
ただそれだけであるが、ときに読みやすい記法となる。
```
plus a b = a + b
```
上の関数 `plus` を、２通りの書き方で利用できるようになる。
```
ghci> plus 1 2
3

ghci> 1 `plus` 2
3
```
上記のように、中置記法を利用する場合、バッククォートで関数名を囲んで利用する。

---
関数を「記号」で作った場合、バッククォートで囲まなくても中置記法が利用できるようになる。<br>
「記号」で関数を作る場合、`()` でくくって関数を定義する。
```
ghci> (%%) x y = x + y
ghci> 2 %% 5
7
```

## fold right（右からの畳み込み）
右からの畳み込み `foldr` の定義は以下の通り。
```
foldr :: Foldable t => (a -> b -> b) -> b -> t a -> b
```
`foldl` と同様に `t a` は `[a]` と考えると以下のようになる。
```
foldr :: (a -> b -> b) -> b -> [a] -> b

foldr f z (x:xs) = f x (foldr f z xs)
foldr _ z [] = z
```

`f :: a -> b -> a` を、中置記法で `a F b -> a` と表すと、下のようになる。
```
foldr f z [x1, x2, x3]
= x1 F (foldr f z [x2, x3])
= x1 F (x2 F foldr f z [x3])
= x1 F (x2 F (x3 F (foldr f z [])))
= x1 F (x2 F (x3 F z))
```

例として、`f = (+)` とすると、以下のようにパターンマッチが行われる。
```
foldr (+) 0 [1,2,3]
    = 1 + (foldr (+) 0 [2,3])
    = 1 + (2 + (foldr (+) 0, [3]))
    = 1 + (2 + (3 + (foldr (+) 0 [])))
    = 1 + (2 + (3 + 0))
```

`foldl` と同様に `foldr f init [..., x3, x2, x1]` をイメージすると、以下の感じになる。
```
acc0 = init
acc1 = f x1 acc0  -- x1 は末尾の項であることに注意
acc2 = f x2 acc1
acc3 = f x3 acc2
...
```

`foldr` は、リストの右側から評価をしていくため、リストからリストを生成する場合、
リストの順序を保ったままリストを得ることができる。<br>

例として、`filter` を `foldr` を用いて実装すると以下のようになる。
```
myFilter f xs = foldr chk [] xs
    where chk x xs 
            | f x       = x : xs
            | otherwise = xs
```

上のように `myFilter` を作ると、以下のように利用できる。
```
ghci> myFilter odd [1,2,3,4,5]
[1,3,5]
```

Haskell のリストは右結合であるため、Haskell のリスト処理は `foldr` と相性が良い。<br>
`foldr` で表現できる関数を、`primitive recursive` という。<br>
先述の `filter` や、`map`, `foldl` なども多くのリスト操作関数が `primitive recursive` となる。<br>

`foldl` を `foldr` で表すと、以下のようになる。
```
foldl f z xs
    = foldr s id xs z
          where s x g a = g (f a x)
```
上記の `id` は恒等関数 `id a = a` である。

少し複雑であるが、上の式がどのように変形されるのか見てみよう。<br>
まず、シグネチャを確かめてみる。
```
f :: a -> x -> a
id :: a -> a
g :: a -> a
s :: x -> (a -> a) -> (a -> a)
foldr :: (x -> (a -> a) -> (a -> a)) -> (a -> a) -> [x] -> (a -> a)
```

上のことを念頭において、式変形をしてみよう。
```
foldr s id [x1, x2, x3] z  -- foldr の評価結果は a -> a であることに注意
= s x1 (foldr s id [x2, x3]) z
= s x1 (s x2 (foldr s id [x3])) z
= s x1 (s x2 (s x3 (foldr s id []))) z
= s x1 (s x2 (s x3 id)) z
= (s x2 (s x3 id))(z F x1)
= (s x3 id)((z F x1) F x2)
= id(((z F x1) F x2) F x3)
= ((z F x1) F x2) F x3
= foldl f z [x1, x2, x3]
```

## ラムダ式
`foldl` や `foldr` などでは、関数を渡すことでリストから何らかの値を算出するなどした。<br>
Haskell では、「関数を渡して何かを実行する」ということをよく行う。<br>
そのようなときに、「渡す関数に名前を付けて定義する」ということが煩雑になりがちになる。<br>
そこで、名前を付けなくても関数を定義する方法がある。例えば、以下の式は値を２倍する関数を表す。
```
\x -> x * 2
```
上の式は、以下のように書いてもまったく同じ意味となる。
```
\y -> y * 2
```
このように名前を付けていない式をラムダ式という。関数とまったく同じように扱えるため、上の関数に 5 を適用すると 10 という答が返ってくる。
```
ghci> (\x -> x * 2) 5
10
```

引数を２つとる場合は、以下のように書ける。
```
\x -> \y -> x - y
```
上のように書いたとき、関数に値を適用すると左側（外側）から値が適用される。引き算のされ方に注目して、以下の例を見てほしい。
```
ghci> (\x -> \y -> x - y) 10 3
7
ghci> (\x -> \y -> x - y) 3 10
-7
```

実際にラムダ式を `foldl` に与えて、その結果を見てみよう。<br>
まず、ラムダ式を使わない場合。
```
ghci> add x y = x + y
ghci> foldl add 2 [3,4,5]
14  -- 2 + 3 + 4 + 5 = 14 となる
```

次にラムダ式を使う場合。
```
ghci> foldl (\x -> \y -> x + y) 2 [3,4,5]
14
```

ラムダ式を使う例。
```
ghci> map (\(a, b) -> a + b) [(1, 2), (3, 4), (5, 6)]
[3,7,11]
```

## asパターン
`x@y` で、`y` のパターンにマッチした値を `x` に束縛させる。
```
f str@(x:xs) = do
    print str
    print x
    print xs
```
このとき、以下のようにできる。
```
ghci> f "abcde"
"abcde"
'a'
"bcde"
```

## 評価の強制
実行効率を上げたいとき、遅延評価を行うのではなく、ある時点で評価を実行したいときがあると思う。<br>
評価を実行させる関数が `seq` となる。
```
seq :: a -> b -> b
```
`seq` は、`a` を評価した後、`b` を受け取り `b` を返す。<br>
関数を呼び出すとき、引数を評価した後、関数に引数を渡す、というために利用されるケースが多いと思う。<br>
そのため、関数を呼び出す時に使いやすいように `$!` 演算子が定義されている。
```
($!) :: (a -> b) -> a -> b
f $! x = x `seq` f x
```

> Tips<br>
> `seq` の評価は、**弱頭部正規形**（WHNF：Weak Head Normal Form）という形まで評価を進める、
> という意味なので、実際にプログラムの実行効率を上げるためには慎重に考えなければならない。<br>
> WHNF について知識を増やした後に、`seq` を使うようにした方がよい。

## group expression
`$` は、group expression と呼ばれ、`$` から後ろの部分を可能な限り１つの（）としてまとめる。
`f (g x y)` と `f $ g x y` は同じ意味。
`f (g (h x y)) z` と `f (g $ h x y) z` は同じ意味。

## 関数の合成
`f.g` で関数の合成ができる。
```
f :: Int -> Int
f x = x + 10

g :: Int -> Int
g x = x^2
```
このとき、以下のようにできる。
```
ghci> g 5
25

ghci> f 25
35

ghci> h = f.g
ghci> h 5
35
```

## $（group expression）と .（合成）について

* `$` も `.` も右結合

`f $ g $ h == f $ (g $ h)`

`f . g . h == f . (g . h)`

関数を並べて書いたときは左結合になるため注意。<br>
`f g h = (f g) h`

* 演算子の優先順位

`$` の優先順位は最低。

`.` の優先順位は最高。しかし、関数適用よりは低いため注意。<br>
`f . g 10` は `f . (g 10)` と解釈されるため注意が必要。<br>
`f(g(10))` を意図するなら、`(f.g) 10` と書く必要がある。

## ポイントフリースタイル
`h x y = f (g x y)` のようなものがあった場合、`x` や `y` の部分をポイントと呼ぶ。<br>
引数を用いずに関数を表現することを、ポイントフリースタイルという。<br>
関数の合成 `f (g x) = (f.g) x` を用いると、様々な変形ができる。以下の例を見て慣れてほしい。<br>

Haskell では、「関数への適用」の優先順位が最も高いため、`()` を使う場所に注意すること。<br>
例えば、`f.(g x) y` は、`f.((g x) y)` という意味になるので注意
```
h x y =   f  (g x y)
      =   f. (g x) y  -- f と g x を合成した
      = ((f.).g) x y  -- f. と g を合成した
```
以上により、`h = (f.).g` となると分かる。

もう１つ例を見ておこう。<br>
`h x y = f x (g y)` をポイントフリーに書き換えてみる。
```
h x y = f x (g y)
      = (f x.g) y  -- f x と g を合成した
      =  (.g)(f x) y  -- x を外に出しやすいように、f x と g の場所を入れ替えた
      = ((.g).f) x y  -- .g と f を合成した
```
以上により、`h = (.g).f` となると分かる。

> Tips<br>
> `(.g)` は関数であり、`(.g) f = f.g` と解釈される。<br>
> `(.g).f` は、`.g` と `f` の合成であるため、解釈を間違えないように。
