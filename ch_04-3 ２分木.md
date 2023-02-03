## ２分木探索の例

まずは、２分木の構造を以下のように決める。
```
data Tree a = 
   Node a (Tree a) (Tree a)
   | Leaf a
   | Empty
   deriving (Show)
```

次に、木構造に要素を挿入する関数を作る。
```
treeInsert :: (Ord a) => a -> Tree a -> Tree a  
treeInsert x Empty = Leaf x

treeInsert x leaf@(Leaf a)
   | x == a = leaf
   | x < a  = Node a (Leaf x) Empty
   | x > a  = Node a Empty (Leaf x)
   
treeInsert x node@(Node a left right)
   | x == a = node
   | x < a  = Node a (treeInsert x left) right
   | x > a  = Node a left (treeInsert x right)
```

`treeInsert` の動作確認
```
ghci> nums = [10, 6, 4, 1, 6, 2, 1, 7, 5]
ghci> t = foldr treeInsert Empty nums
ghci> t
Node 5 (Node 1 Empty (Node 2 Empty (Leaf 4))) (Node 7 (Leaf 6) (Leaf 10))
```

要素を検索する関数を作ってみよう。
```
treeElem :: (Ord a) => a -> Tree a -> Bool
treeElem x Empty = False

treeElem x (Leaf a)
   | x == a    = True
   | otherwise = False

treeElem x (Node a left right)
   | x == a = True
   | x < a  = treeElem x left
   | x > a  = treeElem x right
```

動作確認。`t` は先程作ったものと同じもの。
```
ghci> treeElem 2 t
True
ghci> treeElem 3 t
False
```
