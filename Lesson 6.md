
we can use the word `let ` as variable attribution :
```racket
(let ([x 5]) (+ x x)) // x =5 -> x+x = 10 
```

the `let` make the attribution locally where the key-word `define` make the attribution  globally .

It's important in a language to allow to attribute values to variables :
	1. To avoid code duplication
	2. Efficiency
	3. Simplicity and readability
	4. Expressiveness

In our language we will add another key-word `with` that will add a local variable

for that we will use the  older definition from [[Lesson 5]] :

The WAE grammer 

\<WAE> := \<num> 1
			| { +\<WAE> \<WAE>} 2 
			| { -\<WAE> \<WAE>} 3  
			| { \*\<WAE> \<WAE>} 4  
			| { \/\<WAE> \<WAE>} 5  
			| {with {\<id> \<WAE> } \<WAE> } 6   ;where \<id> refers to any Racket Symbol, and \<num> is any Racket  Number .
			| \<id> 7

\<WAE =6>  {with {\<id> \<WAE> } \<WAE>} 
	=> {with {x \<WAE> } \<WAE>} =2 
	=> {with {x { + \<WAE>  \<WAE>} \<WAE>} = 1,1>
	=> {with {x { + 4  2 }} \<WAE>} =4>
	=> {with {x { + 4  2 }} { \* \<WAE>  \<WAE>}}  =7,7>
	=> {with {x { + 4  2 }} { * x  x }}  

Implementation in Racket

```racket
(define-type WAE
	[Num Number]
	[Add WAE WAE]
	[Sub WAE WAE]
	[Mul WAE WAE]
	[Div WAE WAE]
	[With Symbol WAE WAE]
	[Id Symbol]
)
```
	
Now we will parse the new operation by updating our parser from [[Lesson 4]]  :

```racket
(: parse-sexpr : Sexpr -> WAE)
(define (parse-sexpr sxp)
	(match sxp
		[(number: n) (Num n)]
		[(Symbol: name) (Id name)]
		[(cons 'with more)
		(match sxp ;we check if the With input is sexpr
		(list 'with (list (symbol: name) named-expr) body )
			(With name ( parse-sexpr named-expr) (parse-sexpr body))])
		[else (error 'parse-sexpr "bad syntax in ~s" sxp)]]
		[(list '+ l r) (Add (parse-sexpr l) (parse-sexpr r))]
		[(list '- l r) (Sub (parse-sexpr l) (parse-sexpr r))]
		[(list '* l r) (Sub (parse-sexpr l) (parse-sexpr r))]
		[(list '/ l r) (Sub (parse-sexpr l) (parse-sexpr r))]
		[else (error 'parse-sexpr "bad syntax in ~s" sxp)]
))
```


---
# Tirgul 6

exercice : given two number : x, y ,  return a list with : x! + y! , x! - y!, x! * y! , x! / y! , (x! + y!)/2 , sqrt(x! * y!)

```racket
(: fact-result : Natural Natural -> Natural)
(define (fact-result x y)
    (let ([xf (fac x)] [yf (fac y)])
        (list (+ xf yf) (- xf yf) (* xf yf) (/ xf yf) (/ (+ xf yf) 2) (sqrt (* xf yf)))
        )
    )

```

Create a parser with match
```racket
(: parse-sexpr : Sexpr -> BoolE)
(define (parse-sexpr sxpr)
	(match sxpr
		['t (BOOL #t)]
		['f (BOOL #f)]
		[(list 'not x) (NOT (parse-sexpr x))]
		[(list 'or x y) (OR (parse-sexpr x) (parse-sexpr y))]
		[(list 'and x y) (AND (parse-sexpr x) (parse-sexpr y))]
		[else (error 'parse-sexpr "invalid syntax in ~s" sxpr)]
	)
)
```

Create an eval function to resolve our parsing
```racket
(: eval-BoolE : BoolE -> Boolean)
(define (eval-BoolE BE)
	(cases BE
		[(BOOL b) b]
		[(AND x y) (and (eval-BoolE x) (eval-BoolE y))]
		[(OR x y) (or (eval-BoolE x) (eval-BoolE y))]
		[(NOT x) (not (eval-BoolE x))]
	)
)

(: eval : String -> Boolean)
(define (eval str)
	(eval-BoolE (parse str)))
```