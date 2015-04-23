---
layout: post
title: Understanding About Monad
---

Following last week's lecture, here is some of my basic understanding and note about Monad.   

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
Here we may notice 2 branches of `eval e2`. That is, either `Nothing` or `Just x`.
  
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


















Reference from:   
*[The Essence of Functional Programming](http://homepages.inf.ed.ac.uk/wadler/papers/essence/essence.ps)*  
*[单子(monad)入门(一)](http://www.iis.sinica.edu.tw/~scm/ncs/2009/11/a-monad-primer/)*

