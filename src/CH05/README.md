
# Notes

## Why Types?

Type systems in logic and mathematics have been designed to impose constraints that enforce correctness.  
A well-designed type system helps eliminate some classes of errors as well as concerns such as what the effect of a conditional over a non-boolean value may be.  
In Haskell, types as *static*. This means that typechecking happens at compile-time.  

When checking types in the REPL, you may notice that GHCi often gives types the broadest possible applicabilty it can infer. As an example:  

```haskell
Prelude> :type 13                -- Looks like an Int or Integer
13 :: Num a => a                 -- ... but it's actually a much more generic `Num`
Prelude> let x = 13 :: Integer
Prelude> :t x
Prelude> x :: Integer            -- We've forced GHCi to infer a concrete type
```

## The Function Type

The arrow, `(->)`, is actually the type constructor for functions in Haskell. It's baked into the language.  
It's similar to `Bool`, which we saw in the previous chapter, except that the `(->)` type constructor takes arguments and has no data constructors.  

```haskell
Prelude> :info (->)
data (->) (a :: TYPE q) (b :: TYPE r) 	-- Defined in ‘GHC.Prim’
infixr 0 `(->)`
```

This is similar to the Tuple data type we saw earlier:  

```haskell
Prelude> :info (,)
data (,) a b = (,) a b 	-- Defined in ‘GHC.Tuple’
```

An interesting revelations follows from this. Consider the following: using the Tuple constructor `(,)` yields Tuple values.  
Similarly, using the function constructor `(->)` yields values as well: functions! Thus: functions are values!  

Let's investigate type signatures again:  

```haskell
fst :: (a, b) -> a
--      [1]  [2] [3]
```

1.  The first paramater has type `(a, b)`.
2.  The function type, `(->)`, has two paramters  
    -   Left: (a, b)
    -   Right: b
3.  THe result of the function has type `a`. This the same type `a` as the one in the first parameter.

## Typeclass Constrained Type Variables

If we look at the type of most arithmetic functions, you'll see something like:  

```haskell
Prelude> :type (+)
(+) :: Num a => a -> a -> a
Prelude> :type (/)
(/) :: Fractional a => a -> a -> a
```

As you can see, these functions are not constrained to a concrete type. Instead, we get the most general type possible.  
`Num a` and `Fractional a` mean that `a` is a **typeclass-constrained polymorphic type variable**. Typeclasses offer a standard set of functions.  
When a typeclass is constraining a type variable, the variable could represent any concrete type that has an instance of that typeclass.  

```haskell
Prelude> let fifteen = 15
Prelude> :t fifteen
fifteen :: Num a => a
Prelude> let fifteenInt = fifteen :: Int
Prelude> let fifteenDouble = fifteen :: Double
Prelude> :t fifteenInt
fifteenInt :: Int
Prelude> :t fifteenDouble
fifteenDouble :: Double
```

The above works because `Int` and `Double` both have an instance of the `Num` typeclass.  

```haskell
Prelude> :info
instance Num Int -- Defined in ‘GHC.Num’
instance Num Double -- Defined in ‘GHC.Float’
```

Type variables can have more than one constraint:  

```haskell
(Num a, Num b) => a -> b -> b
(Ord a, Num a) => a -> a -> Ordering
```

## Currying And Partial Application

Haskell does, in fact, not allow for multiple arguments to be passed into a function. In the Lambda Calculus, all functions take precisely 1 argument.  
However, the language provides syntax for constructing **curried functions**. Currying refers to the nesting of multiple functions,  
each accepting one argument and returning one result, to allow the illusion of multiple-parameter functions.  

We already know this. After all, we've already seen this in the datatype: `data (->) a b`.  
So *exactly* one argument of type `a`, and *exactly* one return value of type `b`.  

This leads to the next subject: **partial application**. Take, for example, the following function:  

```haskell
addStuff :: Integer -> Integer -> Integer
addStuff a b = a + b + 5
```

Let's inspect it in the REPL:  

```haskell
Prelude> :t addStuff
addStuff :: Integer -> Integer -> Integer
Prelude> addStuff 5 5
15                                 -- Okay, works as intended
Prelude> let addTen = addStuff 5
Prelude> :t addTen
addTen :: Integer -> Integer       -- Passing in a single arguments returns a function of Integer -> Integer
Prelude> addTen 5
15                                 -- The partial function can then be applied to a second argument
```

If we look at the type signature of `addStuff`:  

```haskell
addStuff :: Integer -> Integer -> Integer
```

you might notice that we can actually add explicit parentheses to this:  

```haskell
addStuff :: Integer -> (Integer -> Integer)
```

So as you can see, passing a single argument of type `Integer` to this function return a function of type `Integer -> Integer`.  

There's also **sectioning**: a specific form of partial application of infix operators, which has a special syntax and allows us to choose whether the argument you're partially applying the operator to is the first or second argument.  

```haskell
Prelude> let x = 5
Prelude> let y = (2^)
Prelude> let z = (^2)
Prelude> y x
32
Prelude> z x
25
```

Another example:  

```haskell
Prelude> let celebrate = (++ " woot!")
Prelude> celebrate "naptime"
"naptime woot!"
Prelude> celebrate "dogs"
"dogs woot!"
```

You can also use the backtick syntax to use this with functions that are normally prefix:  

```haskell
Prelude> elem 9 [1..10]
True
Prelude> 9 `elem` [1..10]
True
Prelude> let c (`elem` [1..10])
Prelude> c 9
True
Prelude> c 25
False
```

## Polymorphism

**Polymorphic** means "made of many forms". Polymorphic type variables allow us to write functions that act on different types whithout rewriting  
the entire function again.  

Haskell support two different types of polymorphism:  

-   **Paramatric polymorphism:** Implemented through type variables that are fully pollymorphic
-   **Constrained polymorphism:** Implemented through typeclasses

We have a simple way of working around certain type constraints. For example, in the `length` function we wrote previously, the following is impossible:  

```haskell
Prelude> 6 / length [1, 2, 3]

No instance for (Fractional Int) arising
  from a use of ‘/’

In the expression: 6 / length [1, 2, 3]
In an equation for ‘it’: it = 6 / length [1, 2, 3]
```

`length` just isn't polymorphic enough. Let's make a function called `fromIntegral` to work around that:  

```haskell
Prelude> :t fromIntegral
fromIntegral :: (Num b, Integral a) => a -> b
```

If we use this function:  

```haskell
6 / fromIntegral (length [1, 2, 3])
2.0
```

## Type Inference

Haskell can determine the types of expressions for us through a process called **type inference**. Haskell uses an expanded version of the so-called *Damas-Hindley-  
Milner type system*. It will always infer the most polymorphic type possible. For example:  

```haskell
Prelude> let myGreet x = x ++ " Julie"
Prelude> myGreet "hello"
"hello Julie"
Prelude> :type myGreet
myGreet :: [Char] -> [Char]
```

But if we were to change it to:  

```haskell
Prelude> let myGreet x y = x ++ y
Prelude> :type myGreet
myGreet :: [a] -> [a] -> [a]
```

# Exercises

## Exercise 1: Type Matching

1.  `not` -> `not :: Bool -> Bool`
2.  `length` -> `length :: [a] -> Int`
3.  `concat` -> `concat :: [[a]] -> [a]`
4.  `head` -> `head :: [a] -> a`
5.  `(<)` -> `(<) :: Ord a => a -> a -> Bool`

## Exercise 2: Type Arguments

1.  Resulting type for:  
    Given: `f :: a -> a -> a -> a`  
    And: `x :: Char`  
    Then: `f x :: Char -> Char -> Char`

2.  Resulting type for:  
    Given: `g :: a -> b -> c -> b`  
    Then: `g 0 'c' "woot" :: Char`

3.  Resulting type for:  
    Given: `h :: (Num a, Num b) => a -> b -> b`  
    Then: `h 1.0 2 :: Num b => b`

4.  Resulting type for:  
    Given: `h :: (Num a, Num b) => a -> b -> b`  
    Then: `h 1 (5.5 :: Double) :: Double`

5.  Resulting type for:  
    Given: `jackal :: (Ord a, Eq b) => a -> b -> a`  
    Then: `jackal "keyboard" "has the word jackal in it" :: [Char]`

6.  Resulting type for:  
    Given: `jackal :: (Ord a, Eq b) => a -> b -> a`  
    Then: `jackal "keyboard" :: Eq b => b -> [Char]`

7.  Resulting type for:  
    Given: `kessel :: (Ord a, Num b) => a -> b -> a`  
    Then: `kessel 1 2 :: (Num a, Ord a) => a`

8.  Resulting type for:  
    Given: `kessel :: (Ord a, Num b) => a -> b -> a`  
    Then: `kessel 1 (2 :: Integer) :: (Num a, Ord a) => a`

9.  Resulting type for:  
    Given: `kessel :: (Ord a, Num b) => a -> b -> a`  
    Then: `kessel (1 :: Integer) 2 :: Integer`

## Exercise 3: Parametricity

1.  This is indeed impossible. If, for example, we write a function along these lines:  
    
    ```haskell
    id' :: a -> a
    id' x = x + 1 
    ```
    
    The compiler will complain that `No instance for (Num a) arising from a use of ‘+’`. In other words, `a` is being constrained.

2.  The two variants of the function with this type signature are as follows:  
    
    ```haskell
    foo :: a -> a -> a
    foo x y = x
    ```
    
    or  
    
    ```haskell
    foo' :: a -> a -> a
    foo' x y = y
    ```

3.  Only 1 single implementation:  
    
    ```haskell
    bar :: a -> b -> b
    bar x y = y
    ```

## Exercise 4: Apply Yourself:

1.  Applying `myConcat x = x ++ " yo"`   
    With type signature `(++) :: [a] -> [a] -> [a]`   
    Will result in:  
    a) `myConcat :: [Char] -> [Char]`  
    b) `(++) :: [Char] -> [Char] -> [Char]`

2.  Applying `myMult x = (x / 3) * 5`   
    With type signature `(*) :: Num a => a -> a -> a`   
    Will result in:  
    a) `myMult :: Fractional a => a -> a`  
    b) `(*) :: Fractional a => a -> a -> a`

3.  Applying `myTake x = take x "hey you"`   
    With type signature `take :: Int -> [a] -> [a]`   
    Will result in:  
    a) `myTake :: Int -> [Char]`  
    b) `take :: Int -> [Char] -> [Char]`

4.  Applying `myCom x = x > (length [1..10])`   
    With type signature `(>) :: Ord a => a -> a -> Bool`   
    Will result in:  
    a) `myCom :: Int -> Bool`  
    b) `(>) :: Int -> Int -> Bool`

5.  Applying `myAlph x = x < 'z'`   
    With type signature `(<) :: Ord a => a -> a -> Bool`   
    Will result in:  
    a) `myAlph :: Char -> Bool`  
    b) `(<) :: [Char] -> [Char] -> Bool`

## Exercise 5: Chapter Summary

### Multiple Choice:

1.  A value of type `[a]` is:  
    Answer **c**: a list whose elements are all of some type `a`.

2.  A function of type `[[a]] -> [a]` could:  
    Answer **a**: take a list of strings as an argument.

3.  A function of type `[a] -> Int -> a`:  
    Answer **b**: returns one element of type `a` from a list.

4.  A function of type `(a, b) -> a`:  
    Answer **c**: takes a tuple argument and returns the first value.

### Determine the Type:

1.  Functions applications:  
    a) `(* 9) 6` returns `54 :: Num a => a`  
    b) `head [(0,"doge"),(1,"kitteh")]` returns `(0, "doge") :: Num a => (a, [Char])`  
    c) `head [(0 :: Integer ,"doge"),(1,"kitteh")]` returns `(0, "doge") :: (Int, [Char])`  
    d) `if False then True else False` returns `False :: Bool`  
    e) `length [1, 2, 3, 4, 5]` returns `5 :: Int`  
    f) `(length [1, 2, 3, 4]) > (length "TACOCAT")` returns `False :: Bool`

2.  Given:  
    
    ```haskell
    x = 5
    y = x + 5
    w = y * 10
    ```
    
    The type of `w` is `Num a => a`

3.  Given:  
    
    ```haskell
    x = 5
    y = x + 5
    z y = y * 10
    ```
    
    The type of `z` is `Num a => a -> a`

4.  Given:  
    
    ```haskell
    x = 5
    y = x + 5
    f = 4 / y
    ```
    
    The type of `f` is `Fractional a => a`

5.  Given:  
    
    ```haskell
    x = "Julie"
    y = " <3 "
    z = "Haskell"
    f = x ++ y ++ z
    ```
    
    The type of `f` is `[Char]`

### Does It Compile:

1.  Nope: `bigNum` is op type `bigNum :: Num a => a`. You can't apply it to something else.  
    Fix:  
    
    ```haskell
    bigNum = (^) 5
    wahoo = bigNum $ 10
    ```

2.  Yes

3.  Nope. This makes too little sense to explain.  
    Fix: Really depends on what the hell you would want to do here.

4.  Nope. `c` is never defined.

### Type Variable or Constructor:

1.  `f :: Num a => a -> b -> Int -> Int`  
    a) Constrained  
    b) Fully polymorphic  
    c) Concrete  
    d) Concrete

2.  `f :: zed -> Zed -> Blah`  
    a) Fully polymorphic  
    b) Concrete  
    c) Concrete

3.  `f :: Enum b => a -> b -> C`  
    a) Fully polymorphic  
    b) Constrained  
    c) Concrete

4.  `f :: f -> g -> C`  
    a) Fully polymorphic  
    b) Fully polymorphic  
    c) Concrete

### Write a Type Signature

1.  `functionH :: [a] -> a`
2.  `functionC :: (Ord a) => a -> a -> Bool`
3.  `functionS :: (a, b) -> b`

### Given a Type, Write a Function:

```haskell
module WriteAFunc where

i :: a -> a
i x = x

c :: a -> b -> a
c x y = x

c'' :: b -> a -> b
c'' x y = x         -- c and c'' are the same

c' :: a -> b -> b
c' x y = y

r :: [a] -> [a]
r xs = xs

r' :: [a] -> [a] -- Another example of the above
r' = reverse

co :: (b -> c) -> (a -> b) -> a -> c
co x y z = (x . y) z

a :: (a -> c) -> a -> a
a _ y = y

a' :: (a -> b) -> a -> b
a' x y = x y
```

### Fix It:

1.  Fixed module:  
    
    ```haskell
    module Sing where                   -- `sing` should be `Sing`
    
    fstString :: String -> String       -- `++` should be `->`
    fstString x = x ++ " in the rain"
    
    sndString :: String -> String       -- `Char` should be `[Char]`
    sndString x = x ++ " over the rainbow"
    
    sing =
      if x > y
        then fstString x
        else sndString y                -- `or` should be `else`
      where
        x = "Singin'"
        y = "Somwhere"                  -- `x` should be `y`
    ```

2.  Sing the other song:  
    
    ```haskell
    module Sing where                   -- `sing` should be `Sing`
    
    fstString :: String -> String       -- `++` should be `->`
    fstString x = x ++ " in the rain"
    
    sndString :: String -> String       -- `Char` should be `[Char]`
    sndString x = x ++ " over the rainbow"
    
    sing =
      if x < y
        then fstString x
        else sndString y                -- `or` should be `else`
      where
        x = "Singin'"
        y = "Somwhere"                  -- `x` should be `y`
    ```

3.  Fixed `arith3broken.hs`:  
    
    ```haskell
    module Arith3Broken where
    
    main :: IO()
    main = do
      print (1 + 2)         -- Should be enclosed in brackets or use `$`
      print 10              -- Should use `print` or `putStrLn $ show 10`
      print (negate (-1))   -- Should enclose `-1` in brackets
      print ((+) 0 blah)
      where blah = negate 1
    ```

### Type-Kwon-Do:

1.  Typecheck 1:  
    
    ```haskell
    module TypeCheck1 where
    
    f :: Int -> String
    f = undefined
    
    g :: String -> Char
    g = undefined
    
    h :: Int -> Char
    h x = (g . f) x
    ```

2.  Typecheck 2:  
    
    ```haskell
    module TypeCheck2 where
    
    data A
    data B
    data C
    
    q :: A -> B
    q = undefined
    
    w :: B -> C
    w = undefined
    
    e :: A -> C
    e = w . q
    ```

3.  Typecheck 3:  
    
    ```haskell
    module TypeCheck3 where
    
    data X
    data Y
    data Z
    
    xz :: X -> Z
    xz = undefined
    
    yz :: Y -> Z
    yz = undefined
    
    xform :: (X, Y) -> (Z, Z)
    xform (x, y) = (xz x, yz y)
    ```

4.  Typecheck 4:  
    
    ```haskell
    module TypeCheck4 where
    
    munge :: (x -> y) -> (y -> (w, z)) -> x -> w
    munge fx fy x = fst $ (fy . fx) x
    ```
