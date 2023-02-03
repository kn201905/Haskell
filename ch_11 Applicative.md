# Applicative

## Functor

型クラスの章で `Functor` の型クラスを紹介した。Haskell においては、`Functor` のインスタンスになるためには `fmap` を実装すれば良いだけである。<br>
しかし、実用上で意味のある `Functor` になるためには以下の２つの性質を満たす必要がある。

* (1) `fmap id = id` （`id` とは恒等写像）

* (2) `fmap (f . g) = fmap f . fmap g`

今後、モナドを導入するにあたり、`Functor` のインスタンスは上の２つの性質を満たすものに限って話を進めることにする。<br>
`Maybe` や `Either` など、標準で提供されているものはすべて上の２つの性質を満たす。<br>
また、自分で型を作った場合でも、`fmap` を実装すると、ほとんどの場合、上の２つの性質を満たすものとなってしまうだろう。

---
`Maybe` について、`fmap` が上の性質を満たすことを確認してみよう。<br>
`Maybe` の `fmap` は以下のように定義されている。
```Haskell
instance functor Maybe where
   fmap f (Just x) = Just (f x)
   fmap Nothing = Nothing
```

以下、`Maybe A`（`A` は、`Int` や `String` などと考えてほしい）として考えよう。 

* (1) の性質について
```Haskell
x を A の任意の値とする。（x ∈ A）
このとき、
・fmap id (Just x) = Just (id x) = Just x
・fmap id Nothing = Nothing
であるから、fmap id = id となっている。
```

* (2) の性質について
```Haskell
x を A の任意の値とする。（x ∈ A）
このとき、
・fmap (f.g) (Just x) = Just ((f.g) x) = Just (f (g x))
　(fmap f . fmap g)(Just x) = fmap f (fmap g (Just x)) = fmap f (Just (g x)) = Just (f (g x))

・fmap (f.g) Nothing = Nothing
　(fmap f . fmap g) Nothing = fmap f (fmap g Nothing) = fmap f Nothing = Nothing

以上より、fmap (f . g) = fmap f . fmap g となる。
```

---
例として、(1), (2) の性質を満たさない `fmap` の例をあげておこう。

２分木探索のところで例として作成した `Tree a` に、以下のように `fmap` を実装すると、`Tree a` は Haskell の文法上 `Functor` のインスタンスになる（コンパイルできてしまう）が、(1), (2) の性質を満たさず、実用上意味のないものになってしまう。
```Haskell
instance Functor Tree where
   fmap f (Node x left right) = Node (f x) (fmap f left) (fmap f right)
   fmap f (Leaf x) = Node (f x) Empty Empty
   fmap f Empty = Empty
```

上の実装を見てほしいのだが、`Leaf x` に `f` を適用する際に、`Empty` というゴミを付けてしまっている。<br>
`f` が `f :: a -> b` である場合、`Empty` は確かに `Tree b` の値となるので、上の定義の仕方でコンパイルは通ってしまう。

しかし、容易に想像できるように、(1) の性質を満たすならば、`fmap f (Leaf x) = Leaf x` とならなければならないので、(1) の性質が満たされていないとすぐに分かる。従って、上のように `fmap` を実装した場合、実用上問題が多く発生するプログラムとなってしまうのである。

参考までにいうと、`Leaf x` に関する実装を以下のようにすれば、正しい `functor` となる。
```Haskell
   fmap f (Leaf x) = Leaf (f x)
```

上の例でもなんとなく伝わるかと思うが、元の構造を壊すような `fmap` を作らない限り、正しい `Functor` の実装となる。

## シグネチャ f (a -> b) について

型クラス `Functor` の `fmap` のシグネチャは以下のものであった。
```Haskell
fmap :: Functor f => (a -> b) -> f a -> f b
```

`fmap` に `g :: a -> b -> c`（= `a -> (b -> c)`）を適用すると以下のようになる。
```Haskell
fmap g :: f a -> f (b -> c)
```

例えば、以下が簡単な例となる。`fmap` に `(+)` を適用してみよう。
```Haskell
fmap (+) :: (Functor f, Num a) => f a -> f (a -> a)
```

上の方で出てきた `f (b -> c)` の意味をここで確認しておこう。<br>
例として、型構築子 `f` を `Maybe` と想像すると分かりやすいと思う。以下の例を見てみよう。
```
ghci> g = fmap (+) (Just 2)
ghci> :t g
g :: Num a => Maybe (a -> a)
```

下の例との違いを認識しておく必要がある。
```
ghci> fmap (1+) (Just 2)
Just 3

ghci> (1+) <$> (Just 2)
Just 3

ghci> (+) <$> (Just 2)
... Show できないとエラーが表示される。
```

`g` のシグネチャ `Maybe (a -> a)` とは何であろうか？<br>
以下の試みは失敗するので試してみてほしい。
```
ghci> g 3
エラー

ghci> g (Just 3)
エラー
```

いうなれば、`Maybe (a -> a)` とは、`Maybe` の中に関数 `a -> a` が格納されてできた値、ということになる。<br>
`Maybe Int` と比べれば、なんとなく想像できると思う。`(Just 2) 3` などとするのはおかしいのでは？<br>
`Maybe (a -> a)` は、あくまでも `Maybe` で構築された値であり、それは関数ではない。

`Maybe a -> a` とはまったく異なるもの、と認識してほしい。

では、その格納された関数をどう利用するか、ということが次の **Applicative** の話につながる。

## Applicative

`Applicative` は型クラスの１つで、以下のような定義となる。
```
ghci> :i Applicative
class Functor f => Applicative f where
  pure :: a -> f a
  (<*>) :: f (a -> b) -> f a -> f b
  GHC.Base.liftA2 :: (a -> b -> c) -> f a -> f b -> f c
  (*>) :: f a -> f b -> f b
  (<*) :: f a -> f b -> f a
  {-# MINIMAL pure, ((<*>) | liftA2) #-}
```

`Applicative` というのは、`Functor` である型構築子 `f` に、さらに制約を付けるものである。<br>
ここで `<*>` のシグネチャに注目してほしい。さきほどはどうにも扱えなかった `f (a -> b)` が現れている。そして、`f (a -> b)` に `f a` を適用すると `f b` が返される、となっている。

`f` を `Maybe` に読み替えると、想像しやすくなると思う。<br>

`<*> :: Maybe (a -> b) -> Maybe a -> Maybe b`

では、実際にやってみよう。
```
ghci> g = fmap (+) (Just 2)
ghci> :t g
g :: Num a => Maybe (a -> a)

ghci> g <*> (Just 3)
5
```

これが `Applicative` の主な機能となる。型構築子の中に格納された関数を利用できるようにする、ということである。<br>
一般的に、型構築子の中に格納することを、リフト（持ち上げる）する、という。<br>
演算子 `<*>` は、リフトした関数に、リフトした値を適用するもの、といえる。

一般的に **`<*>` の実装** は、リフトされた関数と値を取り出して、関数に値を適用した結果を再度リフトして返す、というものになる。

なんだか繰り返し同じ単語を綴っているように見えるが、理解が進めばこの文の意味が自然と伝わるようになると思う。

そのように考えると `pure` の意味も分かりやすいと思う。これは、ターゲットとなるものを単純にリフトさせるだけの関数である。そのため、以下のように書くことができる。
```
ghci> (pure (2+)) <*> (Just 3)
Just 5
```

関数適用の優先度が最高であるため、かっこを省いて書くことが多い。
```
-- 型構築子 f を具体的に与えなくても正しく解釈される
ghci> :t pure 1
pure 1 :: (Applicative f, Num a) => f a

ghci> pure 1 :: Maybe Int
Just 1

ghci> pure (2+) <*> Right 3
Right 5

ghci> pure (2+) <*> Nothing
Nothing

ghci> pure (2+) <*> [1,2,3,4,5]
[3,4,5,6,7]
```

`Functor` のときと同様に、`<*>` に `f (a -> b -> c)` を適用してみよう。
```
ghci> :t (<*>) (pure (+))
(<*>) (pure (+)) :: (Applicative f, Num a) => f a -> f (a -> a)
```

`f` を `Maybe` にすると、やはり分かりやすいと思う。<br>
`a -> b -> c` は `a -> (b -> c)` であるため、以下のようにイメージすると良いと思う。

`(<*>) (Maybe (a -> b -> c)) :: Maybe a -> Maybe (b -> c)`

`Maybe a` を与えると、`Maybe (b -> c)` が得られる。実際にやってみよう。
```
ghci> g = pure (+) <*> Just 1
ghci> :t g
g :: Num a => Maybe (a -> a)
```

`g` は、再び `Maybe` にリフトされた関数を表しているので、リフトされた値を適用してみよう。
```
ghci> g <*> Just 2
Just 3
```

`g` にそのまま `Just 2` は適用できない、ということを理解してほしい。<br>
`g` は、`Maybe (a -> a)` という値で、これはそのままでは関数としては利用できない。<br>
`Maybe` の中で関数を適用する、というイメージになる。だから `<*>` が必要となるのである。

あくまでもリフトされた値と演算をしてほしいだけなので、以下のように書いてもよい。
```
ghci> g <*> pure 2
Just 3
```

以上のことをまとめると、以下のように見ると分かりやすいと思う。
```
ghci> Just (+) <*> Just 1 <*> Just 2
Just 3

ghci> Just (+) <*> pure 1 <*> pure 2
Just 3

ghci> pure (+) <*> Just 1 <*> Just 2
Just 3
```

以上のような計算ができるのは、「`Maybe` が `Applicative` だからである」と分かるとよい。

`Applicative` な型構築子 `f` は、`(<*>) :: f (a -> b) -> f a -> f b` によって、演算を型の中に格納して、その型の中で演算ができる、というイメージである。

最後に、`Functor` の `fmap`（`<$>`）によって、関数をリフトできることを思い出すと、下のように簡単に書けるようになる。
```
ghci> (+) <$> Just 1 <*> Just 2
Just 3
```

このように書くのが一番分かりやすいと思う。

---
理解しやすくするため、`Maybe` の `pure` と `<*>` の実装を参考までにのせておく。
```
instance Applicative Maybe where
   pure = Just

   Just f <*> m = fmap f m
   Nothing <*> _m = Nothing
```

## 関数は Applicative となる

関数の型は `(->) r a` であるため、`(->) a` は型変数を１つとる型構築子であるとみることができる。<br>

