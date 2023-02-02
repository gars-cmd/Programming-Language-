To resume : We had model of substitution but with it we lose a lot of time during the eval step each time . 
Then we decide to use the model of SubstCache , where we store during the runtime each time every identifier with his value in a list .

given this exemple : 
```racket 
(run "{with {x 3}
	{with {f {y} {+ x y}}}
		{with {x 5}
			{call f 4}}}}")

; we expect it to return 7 
```

With the substitution model :  We first look the with {x 3} then replace all the free instance of x with 3 in runtime   , that give us : 
```
(run "{with {x 3}
	{with {f {y} {+ x=3 y}}}
		{with {x 5}
			{call f 4}}}}")
```
and we get from this 7 as expected 

Now let's inspect how is going for SubstCache model : 
At the beginning on the first with we store (x 3) in our SubstCache as follow : 
```
(run "{with {x 3} ;[]
	{with {f {y} {+ x y}}} [(x 3)] 
		{with {x 5} 
			{call f x}}}}")
```
Then in the next line i enter into the `with {f {y} {+ x y}}}`  so i add the f with the value {+ x y} into the SubstCache  : 
```racket
(run "{with {x 3} ;[]
	{with {f {y} {+ x y}}} ;[(x 3)]
		{with {x 5}  ;[(f {fun {y} {+  x y}}) (x 3)]
			{call f x}}}}")
```
And for the last line we add the new id x for the val 5 into the SubstCache :

```racket
(run "{with {x 3} ;[]
	{with {f {y} {+ x y}}} ;[(x 3)]
		{with {x 5}  ;[(f {fun {y} {+  x y}}) (x 3)]
			{call f x}}}}") ;[(x 5) (f {fun {y} {+  x y}}) (x 3)]
```
We see that we have a free instance of x at the last line so we replace by our first instance that correspond to x that is (x 5) in our SubstCache and eval the `call f x` with x =5 , in the function f = fun {y} {+ x y} we passed y to be 5 and we fix x to be 5 also then we will get as result 10 .


We see that  in the substitution  model the replacement of the free instances is made in the construction of the function .
But in the SubstCache model the free instances are replaced only when i call the function .

We will introduce two different type of scope : 
**Static Scoping** : the  value free instance of the function is set at the time of the construction of the function . 
**Dynamic Scoping** : the value of the free instance is set at call time .

Let's take exemples of static /dynamic scopes : 
Static result : 
```racket 
(define x 123) ; x =123
(define (getx) x) ; getx -> x
(define (bar1 x)(getx))

(test(getx) => ??) ; return 123

((let ([x 456]) (getx)) => ??) ; because getx don't get parameters we send the same x as before -> 123

(test (bar1  999) => ??) ; again return 123 because the getx function doesn't take parameter 

(define (foo x)
	(define (helper) (+ x 1)) helper)

(test ((foo 0)) => 1) ; return 1 because the function foo use the x passed as input and then the x is overidden in the scope of the foo function and the global x is not used .
```


Dynamic Exemple :
```racket 
(define x 123) ; x =123
(define (getx) x) ; getx -> x
(define (bar1 x)(getx))
(define (bar2 y)(getx))

(test(getx) => ??) ; return 123

((let ([x 456]) (getx)) => ??) ; -> 456 ,because the x is overriden in the scope of the let.

(test (bar1  999) => ??) ; -> 999 because the x is overriden into the scope of the bar function.


(test (bar2  999) => ??) ; -> 123 because y is defined into this function the value of x stay the same.

(define (foo x)
	(define (helper) (+ x 1)) helper)

(test ((foo 0)) => 124) ; return 124 because when the helper fucntion is called there is no x that was created in the scoper of the helper function but just the old x=123 , then he use it.
```

The problem of the dynamic scope  is that like the test with bar2 , just the name of the parameter is important even outside the scope of the function . 


We are calling **open object** an object  that have free instance into it  .
If I want to fix the problem of having free instances by replacing every instance that I meet by his value , i do the substitution model (and it doesn't help us ) .

Then we will want to return our object with his parameter and a list of the instance with them value (as an exemple for the \[(Fun name body ) we return <name, body, env> ]) this is called **closure object** ( we use an environment `env` that we are working with) . 



---
# Tirgul 13

we create new evaluation rule to use the new model of environment .

**Evaluation rules:**
eval(N,env) = N
eval({+ E1 E2},env) = eval(E1,env) + eval(E2,env)
eval({- E1 E2},env) = eval(E1,env) - eval(E2,env)
eval({* E1 E2},env) = eval(E1,env) * eval(E2,env)
eval({/ E1 E2},env) = eval(E1,env) / eval(E2,env)
eval(id,env) = lookup(id,env)
eval({with {x E1} E2},env) = eval(E2,extend(x,eval(E1,env),env))
eval({fun {x} E},env) = <{fun {x} E}, env>
eval({call E1 E2},env1)
	= eval(E1,extend(x,eval(E2,env1),env2))
			if eval(E1,env1) = <{fun {x} E1}, env2>
	= error! otherwise

we define new type for our environment  : 
```racket
(define-type ENV
	[EmptyEnv]
	[Extend Symbol VAL ENV])

(define-type VAL
	[NumV Number]
	[FunV Symbol FLANG ENV])

(: lookup : Symbol ENV -> VAL)
(define (lookup name env)
	(cases env
		[(EmptyEnv) (error 'lookup "no binding for ~s" name)]
		[(Extend id val rest-env)
			(if (eq? id name) val (lookup name rest-env))]))

(: arith-op : (Number Number -> Number) VAL VAL -> VAL)
(define (arith-op op val1 val2)
	(: NumV->number : VAL -> Number)
	(define (NumV->number v)
		(cases v
			[(NumV n) n]
			[else (error 'arith-op "expects a number, got: ~s" v)]))
		(NumV (op (NumV->number val1) (NumV->number val2))))
```

And a new eval function : 
```racket
(: eval : FLANG ENV -> VAL)
(define (eval expr env)
	(cases expr
		[(Num n) (NumV n)]
		[(Add l r) (arith-op + (eval l env) (eval r env))]
		[(Sub l r) (arith-op - (eval l env) (eval r env))]
		[(Mul l r) (arith-op * (eval l env) (eval r env))]
		[(Div l r) (arith-op / (eval l env) (eval r env))]
		[(With bound-id named-expr bound-body)
		(eval bound-body
			(Extend bound-id (eval named-expr env) env))]
		[(Id name) (lookup name env)]
		[(Fun bound-id bound-body)
			(FunV bound-id bound-body env)]
		[(Call fun-expr arg-expr)
			(let ([fval (eval fun-expr env)])
				(cases fval
					[(FunV bound-id bound-body f-env)
					(eval bound-body
						(Extend bound-id (eval arg-expr env) f-env))]
					[else (error 'eval "`call' expects a function, got: ~s" fval)]))]))
```



And the run function : 
```racket
(: run : String -> Number)
(define (run str)
	(let ([result (eval (parse str) (EmptyEnv))])
		(cases result
			[(NumV n) n]
			[else (error 'run
				"evaluation returned a non-number: ~s" result)])))
```


Exercise in the Tirgul 13 slides about the 3 type of model (substitution , substcache, environment ).
