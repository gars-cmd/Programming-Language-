
Like we saw in [[Lesson 3]]  , we will define a new operation for \<AE>

\<AE> :: = 1.\<num>
					2.	 | \<AE> + \<AE>
					3.	 | \<AE> * \<AE>

or :

\<AE> :: = 1.\<num>
					2.	| \<AE> + \<AE>
					3.	| \<mul>

let's define \<mul> :

\<mul>: : = \<num>
					| \<num> * \<mul>

but we cannot have two \<num> then : 

we will define 


\<AE> :: =
					1.	| \<AE> + \<AE>
					2.	| \<mul>
					3. | \<mul> = 4. \<num>
					5. | \<num> * \<num>

exemple:

![[Pasted image 20221108153306.png]]

Now let's program our language :
```racket
;definition of the language and the operations that can be done on it

(define-type AE
	[Num Number]
	[Add AE AE]
	[Sub AE AE]
)

; define the parser for operation in our language (Add and Sub)
(: parse-sexpr : Sexpr -> AE)
(define (parse-sexpr sxp)
	(cond [(number? sxp) (Num sxp)]
		[(and) (list? sxp)
				(= (length sx) 3)
				(eq? (first sxp) '+)) (Add (parse-sexpr (second sxp)
											(parse-sexpr (third sxp))] 
											
		[(and) (list? sxp)
				(= (length sx) 3)
				(eq? (first sxp) '-)) (Sub (parse-sexpr (second sxp)
											(parse-sexpr (third sxp))] 
	[else (error 'parse-sexpr "bad syntax in ~s" sxp)]
))




;Other way 

(: parse-sexpr : Sexpr -> AE)
(define (parse-sexpr sxp)
	(match sxp
		[(number: n) (Num n)]
		[(list '+ l r) (Add (parse-sexpr l) (parse-sexpr r))]
		[(list '- l r) (Sub (parse-sexpr l) (parse-sexpr r))]
		[else (error 'parse-sexpr "bad syntax in ~s" sxp)]
))




;test for our parser 
(define (parse code)
	(parse-sexpr (string -> sexpr code))
)
(test (parse "4") => (Num 4))
(test (parse "{+ 3 4") => (Add (Num 3) (Num 4)))
(test (parse "{+ 3 {-5 4}}") => (Add (Num 3)
									(Sub (Num 5)
										 (Num 4))))

(test (parse "{+ 2 3 4 5}") =error> "bad syntax") 



```


---
# Tirgul 4 

 When we create a new programming language one of the first thing to do is to Specify the language , for this we use BNF ( Backus-Naur form).

ex : 
 \<AE> :: = \<num>
	 | \<AE> + \<AE> 
	 | \<AE> - \<AE> 

#### Terminal and non Terminal Symbols
(From Chat GPT )
> **terminal symbols** are the basic building blocks of a language and cannot be further reduced or broken down. These are the actual characters or strings that are used in a language, such as keywords, variables, literals, operators, etc.

Examples of terminal symbols in programming languages:
- Keywords (e.g. "if", "while", "for" in C-like languages)
- Literals (e.g. "3.14", "hello world" in C-like languages)
- Identifiers (e.g. variable names such as "x", "y", "z" in C-like languages)
- Operators (e.g. "+", "-", "\*", "/"  in C-like languages)
- Punctuation (e.g. ";", ",", "(" in C-like languages)

> **non-terminal symbols** represent the structure of the language, and are used to define the relationships between terminal symbols. They represent the abstract concepts in a language, such as expressions, statements, and rules, and are usually defined in terms of other non-terminal symbols or terminal symbols. Non-terminal symbols are used to build a grammar for a language.

- Expressions (e.g. "x + y" in C-like languages)
- Statements (e.g. "x = x + y;" in C-like languages)
- Rules (e.g. "if-else" statement rule in C-like languages)
- Programs (e.g. a complete program written in a certain programming language)

**Exemple** : Define a BNF  for email : 
\<string> ::= \<char>
    | \<char> \<string> 

\<email> ::= \<string> @ \<string> . \<string>

**Exemple** : Define a BNF for Boolean (that support "and" , "or" , ):

\<Boolean> := true 
            | false
            | { and \<Boolean> \<Boolean> }
            | { or \<Boolean> \<Boolean> }
            | { not \<Boolean> }

*The result in racket :*
```racket
(define-type BoolE
[BOOL Boolean ]
[AND BoolE BoolE]
[OR BoolE BoolE]
[NOT BoolE])
```


The  parser for the boolean type :
```racket
(: parse-sexpr : Sexpr -> BoolE)
(define (parse-sexpr sxpr)
	(cond
		[(equal? sxpr 'f) (BOOL #f)]
		[(equal? sxpr 't) (BOOL #t)]
		[(and (list? sxpr)
			(= (length sxpr) 2)
			(equal? (first sxpr) 'not))
		(NOT (parse-sexpr (second sxpr)))]
	[(and (list? sxpr)
		(= (length sxpr) 3)
		(equal? (first sxpr) 'and))
	(AND (parse-sexpr (second sxpr)) (parse-sexpr (third sxpr)))]
	[(and (list? sxpr)
		(= (length sxpr) 3)
		(equal? (first sxpr) 'or))
	(OR (parse-sexpr (second sxpr)) (parse-sexpr (third sxpr)))]
	[else (error 'parse-sexpr "bad syntax in ~s" sxpr)]
))

(: parse : String -> BoolE)
(define (parse str)
	(parse-sexpr (string->sexpr str)))
```