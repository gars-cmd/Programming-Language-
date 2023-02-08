From ChatGPT : 

"First-order", "higher-order", and "first-class" are terms that describe the treatment of functions in a programming language.

- **First-order** functions are functions that take only values as input and produce values as output. They don't accept functions as arguments or return functions as output.

- **Higher-order** functions are functions that can either take functions as arguments or return functions as output or both.
''
- **First-class** functions are functions that can be used in the same way as values. They can be assigned to variables, passed as arguments to functions, and returned as values from functions.

A programming language that supports first-class functions is considered to be a higher-order programming language.

Static typing and dynamic typing are two ways of defining the type of a variable in a programming language.

**Static typing** means that the type of a variable is defined at compile time and cannot change during the execution of a program. This means that if a variable is declared as an int, for example, it will always be an int and cannot be changed to another type like a string. Languages such as C, Java, and Haskell use static typing.

**Dynamic typing**, on the other hand, means that the type of a variable is determined at runtime and can change during the execution of a program. This means that if a variable is assigned a value of one type, it can later be reassigned a value of a different type. Languages such as Python, Ruby, and JavaScript use dynamic typing.

**Static typing** can help catch type errors at compile time, while **dynamic typing** can be more flexible and allow for quicker development. The choice of static vs dynamic

Static and Dynamic Scoping is in [Lesson 13](Lesson%213) 

When i want to do a call to parse-sexpr ? I want to call to parse-sexpr  when in my define-type i received my type as argument.
exemple : 
```
(define-type WAE
	\[Num Number] ; here there is no need to make a parse 
	\[Add WAE WAE] ;because we have here a WAE then we will use parse on it 
	\[Sub WAE WAE] ;because we have here a WAE then we will use parse on it 
	...)

```
in  function when we want to check if a type that we created is something (check if number check if start with something) like what we usually do in `eval` then we will use the `cases` keyword , this is the difference between `match` and `cases`. There is also no else in `cases` the we will need to wrap all the cases into our checker.

- Id 'x or Id 'y when x or y are part of a body 

 - In the exercice resN (res1, res2 ,res3 ) are the result of the previous eval

- The keyword `parameterize` give the same result as `let` in static environment 

- There is no modification into the parse when moving from dynamic to static environment (the inverse is also true) because in parse we do not interact with defining values , that is the essence of static / dynamic  environment 

- In  the environment model we want to return from eval Function and also number but in the  previous eval from the substCache model we return  Flang and now because we want to keep an eye on the ENV we create the Val type that can  be'' number and Fun with his ENV , then eval will return Val .

- In general we are doing `eval(Fun ..` on function into `eval(Call` because we want to be sure that we are doing `call` to a function and not to something else  and so that the fun-expr return a function .

- **Compositionality** said that when we want to evaluate an expression , there is no importance at any moment in the depth of the tree (it refer to the eval part) .
"if there is a need in the eval state to refer to the height of the tree to decide how to evaluate then there is not compositionality ".

- There is no difference  between the dynamic model and the static  model in the `with` constructor in the `eval` part because the `with` is the same as `call` & `fun` where `fun` define the value of a variable and `call` used it then in the same time we have a definition and a call , with do the same as both of them then no matter if we are in dynamic or static  model definition and call occur in the same environment .


**Ambiguity** say that for an expression  in a language there is more than one tree of derivation , then it can lead to multiple results -> not good 


- In **environment model** we are using the environment of the function on his creation on call to the function 

- In **SubstitutionCache** **model** we are using a cache , but when we expand the cache , if we return backward in the recursion call we loose our expansion from the call to the end of the call 

**PL is a FIRST-CLASS** language  he can also pass function as parameter and return them , we also said that  the function have a **closure** that mean that they capture into them the variable and can access them even after the context they was defined it provide persistence for the values of the variable , it is also possible to define function in runtime .  

**The PL  language is Static Typing** , this means that that the type of every expression and variable is known at compile time , we also this that by defining on every function what it should receive .

**The PL language is Static scope**    This means that the type of each expression and variable is known at compile time, before the program is executed. The type checker verifies that expressions have the correct type and generates an error if a type mismatch is detected.

**Racket is a FIRST-CLASS** language it can handle function like object (return them , pass them as arg , define them at runtime ).

**Racket is a Dynamic Typing** Language , this mean  that the type of a value is determined at runtime, rather than being specified in advance . In Racket, values have types but variables do not have an associated type

**Racket is Dynamic Scope** language , This means that the scope of a variable is determined dynamically at runtime based on the call stack and the location of the variable's definition.



**FLANG  is a FIRST-CLASS** language because we consider function as object , we can pass them as parameter for function and also return them and define them in runtime . 

**FLANG is Dynamic Typing**  language , this mean that the type of a value is determined at runtime, rather than being specified in advance . we have type in Flang but according to which constructor we passed it we don't know which type is returned we only know the type at runtime and not at compile time like static typing.

**FLANG IN SUBSTCACHE model is Dynamic Scope language** , because we allocated every variable to his value when the function is called , then at runtime .

**FLANG IN SUBSTITUTION model is StaticScope** , because we define every value for every variable at the moment they are defined . 

**FLANG IN ENVIRONMENT model is Static scope** , because we saved the value of every variable we defined ,  until the function call ,and we apply them according to those who have been defined before the function definition .


What we call  **the bug of the Substitution model** is that in some specific cases we define as an exemple a function that take as parameter a parameter that was not define in the definition of the function , after we define the parameter , but because we are in static scope we cannot apply a parameter of a function that was already defined , then we need to return an error , but in place in substitution case we apply the value of the parameter => wrong , Then we decided to implement the environment model that save the variable that were defined into the function environment  and apply them when the function is called , if a variable was defined after the definition of the function the variable will not be into the environment of the function and an error will be returned . 
exemple : 
```racket
{with {foo {fun {z} {+  x x}}}
	{with {x 5}
		{call foo x}}}
```

here we define fun(x)  = x + x  => then the environment of the function is { }
when we do foo( x ) we look into the env of the function to check if there is a x but there is no x then we have an error here.
