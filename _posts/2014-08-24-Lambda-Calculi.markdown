---
layout: post
title:  "Lambda Calculi"
date:   2014-08-24 17:34:47
categories: jekyll update
---

true = λt.λs. t

false = λt.λs. s

pair = λa.λb. p p a b

fst = λpair. pair true

snd = λpair. pair false

zero = λs.λz. z

one = λs.λz. s z

two = λs.λz. s (s z)

...

iszero = λm. m (λx. false) true

succ = λm.λs.λz. m s (s z)

plus = λm.λn. m s (n s z)

time = λm.λn. m (plus n) zero

zz = pair zero zero

ss = λp. pair (snd p) (succ (snd p))

prd = λm. fst (m ss zz)

Y = λf. (λx. f (λy. x x y)) (λx. f (λy. x x y))
