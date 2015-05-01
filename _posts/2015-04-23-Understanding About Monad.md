---
layout: post
title: Understanding About Monad
tags : [intro, beginner,haskell]
---

Following last week's lecture on Functional Programming, this study note is my basic understanding about Monad while studying and implementing a simple [Little-Man-Computer(LMC)](http://en.wikipedia.org/wiki/Little_man_computer) simulator, which decodes and executes program written in assembly language.  
  
  
#####What is Monad for?  
Handling execption, e.g. divide by 0.  
<pre><code>data Expr = Num Int | Neg Expr | Add Expr Expr | Div Expr Expr</code></pre>
One solution is to use `Maybe` datatype. 
<pre><code>data Maybe a = Just a | Nothing </code></pre>  
Therefore, once divide by 0 occurs, `Nothing` is returned as a denote of execption.
<pre><code>eval (Div e1 e2) = case eval e2 of
                    Just 0 -> Nothing
                    Just v2 -> case eval e1 of
                                Just v1 -> Just (v1 `div` v2)
                                Nothing -> Nothing
                    Nothing -> Nothing</code></pre>
Here we may notice 2 branches of `eval e2`, either `Nothing` or `Just x`.
  
If it is `Nothing`, throw execption by returning `Nothing`.  

If it is `Just x`, we may pass `Just x` to subsequent function application.

The above 2 branches could be expressed by the `>>=` in Monad. Quite simple and clean.
<pre><code>(>>=) : Maybe a -> (a -> Maybe b) -> Maybe b
(Just x) >>= f = f x
Nothing >>= f = Nothing</code></pre>

Using the above `>>=` definition, `eval (Div e1 e2)` can be rewritten as,
<pre><code>eval (Div e1 e2) = eval e2 >>= \v2 ->
                   if v2 == 0 then mzero
                   else eval e1 >>= \v1 ->
                        return (v1 `div` v2)</code></pre>  
Note that `mzero` is a rename of `Nothing`.

In Haskell, Monad is defined as a type class.
<pre><code>class Monad m where
  (>>=) :: m a -> (a -> m b) -> m b
  return :: a -> m a</code></pre>  
  
    
#####Something on State Monad
State Monad is useful for stateful computation.  

A stateful computation is a function that takes some state and returns a value along with some new state.
<pre><code>s -> (a, s)</code></pre>
 

Example:
<pre><code>tick :: State Int Int
tick = do n <- get
          put (n+1)
          return n

plusOne :: Int -> Int
plusOne n = execState tick n

</code></pre>  
  
  
#####Something on ErrorT Monad
The Error monad is also called the Exception monad. As the name suggests, it deals with Exception. 
 
In the LMC program, there could be many errors while decoding the instructions, for example,

* currently Nothing in accumulator but a STA(store) instruction is to be executed

* Jumping to an undefined label
* Incrementing PC(program counter) but no more instructions
* ......

To deal with these errors more appropriately instead of simply calling error function, a good solution is to wrap the <code>IOEnv</code> into <code>ErrorT</code>.

>ErrorT monad transformer can be used to add error handling to another monad. [See an example to combine it with an IO monad](https://hackage.haskell.org/package/mtl-1.1.0.2/docs/Control-Monad-Error.html)











#####Reference from   
*[The Essence of Functional Programming](http://homepages.inf.ed.ac.uk/wadler/papers/essence/essence.ps)*  
*[单子(monad)入门(一)](http://www.iis.sinica.edu.tw/~scm/ncs/2009/11/a-monad-primer/)*  
*[For a Few Monads More](http://learnyouahaskell.com/for-a-few-monads-more)*
