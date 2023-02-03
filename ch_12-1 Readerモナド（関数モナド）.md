# Readerモナド（関数モナド）

Readerモナドの基本的な定義は以下のようになる。<br>
実際には、技術的な要請でもう少し複雑な定義になっているが、考え方や利用法を知る上では以下の定義で十分である。

```Haskell
newtype Reader e a = Reader { runReader :: (e -> a) }
 
instance Monad (Reader e) where 
   return a         = Reader $ \e -> a 
   (Reader f) >>= g = Reader $ \e -> runReader (g (f e)) e 
```

定義を一見しただけではその意図が分かりにくいと思う。

まず、以下のような関数のチェーンがあったとする。

```Haskell
app = f >>= g >>= h
```

このとき、`f`, `g`, `h` すべてに、`debug` モードであるか `release` モードであるかを知らせたい場合、どのようにしたら良いだろうか？

**Readerモナド** は、そのような場合に活用できるモナドなのである。<br>
もう少し正確に言うと、`f`, `g`, `h` すべてに、**同じ引数を適用したい場合** に利用するモナドである。

そう考えると、`>>=` において、同じ `e` が `f` と `g` に適用されていることが自然に見える。

次の簡単な例を見れば、理解しやすいと思う。

今、整数と、`debug` または `release` の文字列が与えられて、整数に 10 を足したものを文字列として表示するものとしよう。<br>
文字列を表示する際に、モードに応じて、`: debug` または `: release` の文字列を付加して表示するものとする。<br>
このプログラムは以下のように書ける。

```Haskell
{-
実際の Monad 型クラスを利用すると、Applicative 等の型クラス制約がかかるため、簡単のため Monad' 型クラスを作ることにする。
>>= の記号は、>>% で代用する。
-}

class Monad' m where
   return' :: a -> m a
   (>>%)   :: m a -> (a -> m b) -> m b

-- Reader 型の定義
newtype Reader e a = Reader {runReader :: e -> a}

instance Monad' (Reader e) where
   return' x        = Reader $ \_ -> x
   (Reader f) >>% g = Reader $ \e -> runReader (g (f e)) e
  

-- add10, intToStr, appendMode のチェーンを app としておく。
app :: Reader (Int, String) String
app = add10 >>% intToStr >>% appendMode

-- app には ３つの関数がチェーンされているが、その３つともに同じ引数 (5, "debug") を適用させることができる。
main = print $ runReader app (5, "debug")


add10 :: Reader (Int, String) Int
add10 = Reader (\(arg, _) -> arg + 10)

intToStr :: Int -> Reader (Int, String) String
intToStr x = Reader (\_ -> show x)  -- ここは、return' $ show x としてよい。

appendMode :: String -> Reader (Int, String) String
appendMode res_str = Reader (\(_, mode) -> res_str ++ " : " ++ mode)
```

以上の例を見て伝わると思うが、`Reader e` として `e` が利用されるのは、`e` が関数が実行される環境を表している、
と考えられるからである。<be>
一般的に、`e` は config 的なものを表している、と考えるとよい。

Reader モナドは、関数の後ろに書いたものは関数に適応される、という Haskell の記法上の性質をうまく利用したものである、といえる。

`add10` の定義を `add10 (arg, mode) =` のように書いて環境（この場合は `mode`）を自分で受け取っていく形で書けば Reader モナドは不要であるが、`add10` を抜ける際に、次の関数に環境を渡すために `arg` や `mode` を返す必要があるなどして、自分でそのようにチェーンを書くのは案外に面倒であるので試してみてほしい。そうすれば、Reader モナドの存在意義も理解しやすいと思う。

## 関数モナド

関数 `r -> a` は、型変数 `a` を１つとる型 `(->) r` と見ることができる。<br>
そこで、関数 `(->) r` も、モナドのインスタンスとして定義されている。定義は以下のとおり。

```Haskell
instance Monad ((->) r) where
   return x = \_ -> x
   f >>= g  = \w -> g (f w) w
```

上記の定義を見てわかる通り、Reader モナドと同じ考えである。<br>
つまり、関数のチェーンを作ったときに、すべての関数に同じ引数を与えることができる。<br>
Reader モナドのところであげた例を、関数モナドを利用して書き直すと以下のようになる。

```Haskell
app :: (Int, String) -> String
app = add10 >>= intToStr >>= appendMode

main = print $ app (5, "debug")


add10 :: (Int, String) -> Int
add10 (arg, _) = arg + 10

intToStr :: Int -> (Int, String) -> String
intToStr x = \_ -> show x
-- intToStr x = return $ show x と書いても良い
-- intToStr = return . show と書いても良い

appendMode :: String -> (Int, String) -> String
appendMode res_str = \(_, mode) -> res_str ++ " : " ++ mode
```

## 再び Reader モナド

Reader モナドには、Reader モナドを利用しやすくするために、以下のようなヘルパー関数がある。

