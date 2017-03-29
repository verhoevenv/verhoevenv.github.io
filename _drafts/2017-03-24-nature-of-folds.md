a = [1, 2, 3]

b = 1 : 2 : 3 : []

sum a

map (\x -> x + 1) a

foldr (\e -> \a -> a + e) 0 a

foldr (+) 0 a

foldr (\e -> \l -> (e + 1) : l) [] a

foldr (\e -> \l -> e : l) [] a

foldr (:) [] a

foldl (\l -> \e -> e : l) [] a
