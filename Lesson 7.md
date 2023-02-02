we will use again the `with` keyword we saw on [Lesson 6](Lesson%206.md).

we want to define the operation substitution:
`e[v/i]` = To substitute an identifier 'i' in an expression 'e' with an expression 'v' , replace 
all identifiers in 'e' that have the same name 'i' with the expression 'v'.

exemple :
1. `{with {x 6} {* x x}} ==> i=x, e={* x x}, v=6`
e\[v/i] = {* 6 6} 

2. `{with {x 6} {* 7 8}} ==> i=x, e={* 7 8}, v=6`
e\[v/i] = {* 7 8} 

3. `{with {x 6} {+ x with{x 3} 10}} ==> i=x, e={+ x with{x 3} 10}}, v=6`
e\[v/i] = {+ 6 {with {6 3} 10}} -> **WRONG** 3 cannot be 6 .

Then we will fix our operation :
1. **Binding** Instance : is to attribute a variable to a value (ex: {x 6}).
2. Scope : domaine where the operation is done .
3. **Bound** Instance :  is in a {} instance but dont **bind** (ex: {* x x})
4. free instance : instance that are no **bound** and no **bind** (ex: in the local statement e={+ *x* with {x 3}}) our *x* is no bound because there is no `with` before and he is no bind) .

we adapt our definition
`e[v/i]` = To substitute an identifier 'i' in an expression 'e' with an expression 'v' , replace 
all identifiers in 'e' that have the same name 'i' , that are not themselves binding instances with the expression  'v'.

4. `{with {x 6} {+ x with{x 3} x}} ==> i=x, e={+ x with{x 3} x}}, v=6`
e\[v/i] = {+ 6 {with {x 3} 6}}  -> **WRONG** not we expected 

Then we change another time the definition 

`e[v/i]` = To substitute an identifier 'i' in an expression 'e' with an expression 'v' , replace 
all identifiers in 'e' that have the same name 'i' , that are not themselves binding instances with the expression   and that are not in any nested scope of 'i' with the expression  v'.

4. `{with {x 6} {+ x with{x 3} x}} ==> i=x, e={+ x with{x 3} x}}, v=6`
e\[v/i] = {+ 6 {with {x 3} x}}  

5. `{with {x 6} {+ x with{y 3} x}} ==> i=x, e={+ x with{y 3} x}}, v=6`
e\[v/i] = {+ 6 {with {y 3} x}}  

`e[v/i]` = To substitute an identifier 'i' in an expression 'e' with an expression 'v' , replace 
all instances of 'i' that are free in 'e' with an expression 'v'.


let's now write the substitution  operation :

```racket
(test (subst (Mul (Id 'x) (Id 'x)) ;e
'x ;i
(Num 6) ;v
) => (Mul (Num 6) (Num 6)))

(: subst : WAE Symbol WAE -> WAE)
(define (subst expr from to)
	(cases expr 
	[(Num n) expr ]
	[(Add l r) (Add (subst l from ro) (subst r from to))]
	[(Sub l r) (Sub (subst l from ro) (subst r from to))]
	[(Mul l r) (Mul (subst l from ro) (subst r from to))]
	[(Div l r) (Div (subst l from ro) (subst r from to))]
	[(With name named body) (With name (subst named from to )
									   (if (eq? name from) 
									   body
									   (subst body from to)))]
	[(Id  name) (if (eq? name from) to expr) ]
))

```


---

# Tirgul 7

We introduce a new BNF for the language WAE : 

\<WAE> ::= \<num>
| { + \<WAE> \<WAE> }
| { - \<WAE> \<WAE> }
| { * \<WAE> \<WAE> }
| { / \<WAE> \<WAE> }
| { with { \<id> \<WAE> } \<WAE> }
| \<id>

In racket : 
```racket
(define-type WAE
	[Num Number]
	[Add WAE WAE]
	[Sub WAE WAE]
	[Mul WAE WAE]
	[Div WAE WAE]
	[Id Symbol]
	[With Symbol WAE WAE])
```


This formal specs define substitution rules for 'subst':

N\[v/x] = N
{+ E1 E2}\[v/x] = {+ E1\[v/x] E2\[v/x]}
{- E1 E2}\[v/x] = {- E1\[v/x] E2\[v/x]}
{* E1 E2}\[v/x] = {* E1\[v/x] E2\[v/x]}
{/ E1 E2}\[v/x] = {/ E1\[v/x] E2\[v/x]}
y\[v/x] = y
x\[v/x] = v
{with {y E1} E2}\[v/x] = {with {y E1\[v/x]} E2\[v/x]}
{with {x E1} E2}\[v/x] = {with {x E1\[v/x]} E2}

the racket code : 
```racket
(: subst : WAE Symbol WAE -> WAE)
(define (subst expr from to)
	(cases expr
		[(Num n) expr]
		[(Add l r) (Add (subst l from to) (subst r from to))]
		[(Sub l r) (Sub (subst l from to) (subst r from to))]
		[(Mul l r) (Mul (subst l from to) (subst r from to))]
		[(Div l r) (Div (subst l from to) (subst r from to))]
		[(Id name) (if (eq? name from) to expr)]
		[(With bound-id named-expr bound-body)
			(With bound-id
				(subst named-expr from to)
				(if (eq? bound-id from)
					bound-body
					(subst bound-body from to)))]))


```


Eval implementation : 
```racket
(: eval : WAE -> Number)
(define (eval expr)
	(cases expr
		[(Num n) n]
		[(Add l r) (+ (eval l) (eval r))]
		[(Sub l r) (- (eval l) (eval r))]
		[(Mul l r) (* (eval l) (eval r))]
		[(Div l r) (/ (eval l) (eval r))]
		[(With name named body) (eval (subst body name (Num (eval named))))]
		[(Id name) (error 'eval "free identifier: ~s" name)] ))
```

function to find if expression WAE contain a free instance 

```racket
(: containsFreeInstance? : WAE -> Boolean)
(define (containsFreeInstance? expr)
	(cases expr
		[(Num n) #f]
		[(Add l r) (or (containsFreeInstance? l) (containsFreeInstance? r))]
		[(Sub l r) (or (containsFreeInstance? l) (containsFreeInstance? r))]
		[(Mul l r) (or (containsFreeInstance? l) (containsFreeInstance? r))]
		[(Div l r) (or (containsFreeInstance? l) (containsFreeInstance? r))]
		[(With bound-id named-expr bound-body)
			(or (containsFreeInstance? named-expr)
				(containsFreeInstance? (subst bound-body
								bound-id
								(Num 0))))]
		[(Id name) #t]))
```
