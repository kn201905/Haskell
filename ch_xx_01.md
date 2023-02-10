
## モジュールのロード
`:module` コマンドを使う。Data.Ratio module をロードする場合の例。
```
ghci> :module + Data.Ratio
```

## ツール名
* **ghc** -> コンパイラ
* **ghci** -> 対話型インタープリタ、デバッガ
* **runghc** -> インターブリタ


型の情報を知りたいと思った場合、`:set` コマンドで型情報を表示するように設定できる。型情報の表示が不要になったら、`:unset` コマンドで非表示にできる。

```
ghci> :set +t  -- 型情報が表示されるようになる

ghci> 'a'
'a'
it :: Char  -- 文字は「Char」という型で処理されると教えてくれている

ghci> True
True
it :: Bool  -- True は「Bool」という型で処理されると教えてくれている

ghci> :unset +t  -- 型情報が表示されなくなる
```
上記にある `it` は、直前の操作により得られたものを記憶している。以下の例では「:unset +t」としているが、`it` を利用することができる。
```
ghci> 2 + 3
5

ghci> it + 4
9
```
さらに、適当な名前の変数に値を記録しておくことにより、便利な計算機として利用できる。
```
ghci> a = 5 + 10
ghci> a
15

ghci> 2 * 3
6

ghci> a + it
21

ghci> b = 10
ghci> a + b
25

ghci> a + b + it
50
```

`:set +t` されていない状態で「型情報」を見たい場合、`:type` コマンドを利用する。
```
ghci> :type 'a'
'a' :: Char

ghci> :t False  -- :t は :type と同じ意味になる
False :: Bool
```


