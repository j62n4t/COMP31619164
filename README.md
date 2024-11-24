java c
COMP3161/9164 24T3 Assignment 2 
Type Inference for Polymorphic MinHS 
Version 1.4.2 
Marks :        17.5% of the overall mark
Due date: Monday 18th November 2024, 12:00 PM Sydney time 
Overview In this assignment you will implement type inference for MinHS. The language used in this assignment differs from the language of Assignment  1 in two respects:  it has a polymorphic type system, and it has aggregate data structures.
The assignment requires you to:
•  (100% marks) Implement the type inference  algorithm discussed in the lectures for polymorphic MinHS with sum and product data types;
•  (5% bonus) adjust the type inference pass to include various simple syntax extensions from Assign- ment 1;
•  (5% bonus) adjust the type inference pass to allow optional type annotations provided by the user;
Each of these parts is explained in detail below.
The MinHS parser and evaluator are provided for you.  You do not have to change anything in any module other than TyInfer .hs (even for the bonus parts).
Your type inference pass should return the inferred type scheme of the top-level binding, main.
Your assignment will only be tested on correct programs, and will be judged correct if it produces a correct type for main up to α-renaming of type variables, and reordering of quantifiers.
Submission 
Submit your (modified) TyInfer .hs using the CSE give system, by typing the command
give  cs3161  TyInfer  TyInfer .hs
or by using the CSE give web interface.
1 Task 1 
Task 1 is worth 100% of the marks of this assignment. You are to implement type inference for MinHS with aggregate data structures. The following cases must be handled:
• the MinHS language of the first task of assignment 1 (without n-ary functions, or lists);
• product types: the 0-tuple (aka the Unit type) and 2-tuples;
• sum types;
•  polymorphic functions 
These cases are explained in detail below. The abstract syntax defining these syntactic entities is in Syntax .hs. You should not need to modify the abstract syntax definition in any way.Your implementation is to follow the definition of inference rules provided with this assignment.  In particular, you must implement a variables-in-contexts version of Hindley-Milner type inference  [1] by tracking definitions (in addition to declarations) for type variables in the context.Additional material can be found in the lecture notes on polymorphism, and reference materials [2,3]. The variables-in-contexts approach you will need to implement is outlined in this assignment specification, along with inference rules in Figures 1,2 and 3. 
2 Bonus Tasks These tasks are all optional, and are worth a total of an additional 10%.  Marks above 100% are converted to exam marks, at an exchange rate of 1 to 0.15—for example, a mark of 105% yields 0.75 bonus marks for the exam.
2.1 Bonus Task 1: Simple Syntax Extensions 
This bonus task is worth an additional 5%.  In this task, you should implement type inference for multiple bindings in the one let expression, with the same semantics as the extension task for Assignment 1.
You will need to develop the requisite extensions to the type inference algorithm yourself, but the exten- sions are very similar to the existing rules.
2.2 Bonus Task 2: User-provided type signatures This bonus task is worth an additional 5%.  In this task you are to extend the type inference pass to accept programs containing some type information. You need to combine this with the results of your type inference pass to produce the final type for each declaration.  That is, you need to be able to infer correct types for programs like:
main  =  let  f  ::   (Int  ->  Int)
=  recfun  g  x  =  x;
in  f  2;
You must ensure that the type signatures provided are not overly general.  For example, the following pro- gram should be a type error, as the user-provided signature is too general:
main  ::   (forall  ’a .  ’a)  =  3;
You may assume for simplicity that the user-provided types have distinct type variable names for all bound / free type variables. Your solution should, for example, support programs such as:
main  ::  forall  ’a .  ’a  ->  ’a  +  Int  =  recfun  m  y  =
let  g  ::  forall  ’b .  ’b  ->  ’a  +  ’b  =  recfun  f  x  =  Inl  y;
in  g  1
where the type variable  ’a is in scope for the expression bound to main and appears in the user-provided type annotating g. All occurrences of ’a within the bound expression reference the same type variable.
3 Algebraic Data Types This section covers the extensions to the language of the first assignment. In all other respects (except lists) the language remains the same, so you can consult the reference material from the first assignment for further details on the language.
3.1 Product Types 
We only have 2-tuples in MinHS, and the Unit type, which could be viewed as a 0-tuple.

3.2 Sum Types 
Sum types in MinHS follow the presentation in the lectures.

3.3 Polymorphism The extensions to allow polymorphism are relatively minor.  Three new type forms have been introduced: the FlexVar  t form, the RigidVar  t form, and the Forall  t  e form.   FlexVar  t represents a unification variable introduced during type inference.   RigidVar  t represents  a fixed type variable introduced by a forall-quantifier.  Consult Section 4 for more details on the notational conventions used in this specification and how they relate to the Haskell code.  We distinguish between type schemes and other types:

Type inference should return a correctly typed top-level binding for main.  For example, consider the fol- lowing code fragment before and after type inference:
main  =
let  f  =  recfun  g  x  =  x;
in  if  f  True
then  f   (Inl  1)
else  f   (Inr   ());
main   ::   (Int  +  1)  =
let  f  =  recfun  g  x  =  x;
in  if  f  True
then  f   (Inl  1)
else  f   (Inr   ());
4 Notational Conventions 
In this document, we will use a number of conventions and conveniences to streamline the presentation detailed in Table 1.

Table 1: Notations and Conventions in this Specification vs. Haskell code for Assignment
Type variable names are ranged over by lowercase greek letters.  We distinguish between flexible and rigid type variables by superscripting such names with F and R, respectively.
Declarations and definitions for flexible type variables appear in typing contexts, see Section 5.1 for the full grammar of contexts.Substitution for type variables occurring in types is explained in Section 6. Since there are two kinds of type variables:  flexible ones introduced during type inference, and rigid ones bound by ∀-quantifiers, we have two kinds of substitution operation depending on which type variables we wish to replace.  These are called substFlex and substRigid in the Subst .hs Haskell module, substituting for flexible and rigid type variables, respectively.
5 Type Inference Rules 
The type inference are rules presented in Figure 3using the judgement form.

where blue components on the left of  can be viewed as inputs to the algorithm, while red components on the right of  can be viewed as outputs. The type inference rules directly encode an algorithm with explicit inputs and outputs. Inputs to the conclusion are inputs to the first premise (by convention the left-most premise of a rule). The outputs of a premise can be used as inputs to subsequent premises. The left-hand typing environment and expression are inputs to any rule. The right-hand typing environment and the type are outputs. The judgement encodes a substitution from type variables in the input typing environment to types in the output typing environment. 
Such an algorithm has type signature:
inferExp   ::  Gamma  ->  Exp  ->  TC   (Type,  Gamma)The goal is to infer the type of the top-level binding by inferring the types for all its sub-expressions, and propagating this information upwards through the program source tree. You should replace the unannotated top-level binding (i.e. main) with an explicitly typed equivalent.
5.1 Contexts for Polymorphism, Contexts for Type Inference To support polymorphism and enable type inference, the grammar for typing contexts Γ is extended to track unification or flexible type variables, and their instantiation with a concrete type, if any. They are “flexible” in the sense that they are introduced by the type inference algorithm in order to establish constraints between  types.A context tracks both declarations and definitions of flexible type variables.  A declaration is denoted by α in this document, or α :=HOLE in Haskell and declares α to be the name of a flexible type variable which has not yet been instantiated with a concrete type. On the other hand, a definition is denoted by α := τ for some type variable name α and type τ . The full grammar for contexts looks like this:
Γ     ::=   ϵ | Γ · x : σ  | Γ · α  | Γ · α := τ  | Γ · • 
We distinguish a context from a suffix ∆ which is a list containing only type variable declarations or defini- tions:
∆    ::=    [] | α :: ∆ | α := τ :: ∆
5.2 Generalisation Generalisation is the process of introducing zero or more 丫 quantified type variables on a type to form. a type scheme. This process must be minimal in the sense that it generalises only those type variables relevant for the current type inference problem. The judgement form. is:
where from input context Γ 1  and expression e the algorithm infers the type scheme σ and output context Γ2 . There is only one rule for generalisation given by:

where rev(∆) reverses the elements in the suffix ∆ .The algorithm keeps track of relevant type variables by using markers, denoted by •, which split the context up into localities.  A type variable occurring to the right of a marker can be safely generalised in a type scheme because the surrounding context does not depend on it.  In the GEN rule, we can generalise τ over ∆ (which contains only type variable declarations and definitions) to form. the type scheme σ:
Type variables can gain more global relevance by moving their declarations to the left of markers. Such a movement is irreversible, once a type variable has moved beyond a marker it can never be moved back “to the right” .To keep type inference tractable, generalisation is performed only on let-bindings and the top-level main binding. Recursive functions are not generalised.  Pay close attention to the premises of the LET  and PRO - GRAM typing rules in Figure 3to see where generalisation should be used.
5.代 写COMP3161/9164 24T3 Assignment 2 Type Inference for Polymorphic MinHSPython
代做程序编程语言3 Unification Solving type inference problems depends on solving unification problems between types. Just like the type inference rules, unification can be expressed algorithmically as a transformation on the typing context. Some of these rules were discussed in lectures and are repeated in Figure 1 (except a few structural and symmetric rules — fill in the details!) and take the following form.
where, again, inputs are blue and outputs are red.

Figure 1: Unification Rules
5.4 Instantiation The unification judgement depends on instantiation, that is, solving an equation involving a flexible type variable (unification variable) and any type which is not a flexible type variable. This judgement (Figure 2) takes the following form.
The algorithm simplifies the problem by moving through the context, similar to the strategy employed during unification for the case of unifying two flexible type variables. Some points worth noting:
• FTV (τ) computes the set of names of flexible type variables occurring in τ ;
•  For DEFN, we must check that the no flexible type variables occurring in τ have the name α .  This condition is known as the occurs check and prevents cyclic dependencies which are unsound;
•  Instantiation accumulates a list ∆ recording the type variable dependencies of τ which must be given more global relevance in order to solve for α ;
•  For DEFN, the dependencies of τ are placed to the right of the definition of α because by setting α :=τ , it also depends on type variables in ∆;
6 Substitution 
Substitutions are implemented as an abstract data type, defined in Subst .hs. Subst is an instance of the Monoid type class, which is defined in the standard library as follows:
class  Monoid  a  where
mappend  ::  a  ->  a  ->  a     --  also  written  as  the  infix  operator  <> 
mempty    ::  a

Figure 2: Algorithmic instantiation rulesFor the  Subst instance, mempty corresponds to the empty substitution, and mappend is substitution composition. That is, applying the substitution a  <>  b is the same as applying a and b simultaneously. It should be reasonably clear that this instance obeys themonoid laws:
mempty  <>  x  ==  x                                  --  left  identity
x  <>  mempty  ==  x                                  --  right  identity
x  <>   (y  <>  z)  ==   (x  <>  y)  <>  z  --  associativityIt is also commutative (x  <>  y  ==  y  <>  x) assuming that the substitutions are disjoint (i.e that dom(x)∩ dom(y) = ∅). In the type inference algorithm, your substitutions are all applied in order and thus should be  disjoint, therefore this property should hold.
You can use this <> operator to combine multiple substitutions into a single substitution; however, there should be little need to use anything more complicated than single substitutions for this algorithm.
You can construct a singleton substitution, which replaces one variable, with the =: operator, so the sub- stitution  ("a"  =:    FlexVar  "b")  <>   ("b"  =:    FlexVar  "c") using substFlex is a sub- stitution which renames all flexible type variables with name a or b with the name c.
The Subst module also includes a variety of functions for running substitutions on types and expres- sions for both rigid and flexible type variable replacement. In particular, the module provides two functions for expressions and types respectively, for substitution of type variables:
--  |   Perform. substitution  on  rigid  type  variables .
substRigid   ::  Subst  ->  Type  ->  Type
--  |   Perform. substitution  on  flexible  type  variables .
substFlex    ::  Subst  ->  Type  ->  Type
and similar for Exp. You can use these functions for implementing some of the algorithimic rules.
7 Errors and Fresh Names 
Thus far, the following type signature would be sufficient for implementing our type inference function:
inferExp   ::  Gamma  ->  Exp  ->   (Type,  Gamma)
Unification is a partial function, however, so we want a principled way to handle the error cases, rather than just bail out with error calls.
To achieve this, we’ll adjust the basic, pure signature for type inference to include the possibility of a TypeError:

Figure 3: Algorithmic Type Inference Rules for MinHS expressions and programs
inferExp   ::  Gamma  ->  Exp  ->  Either  TypeError   (Type,  Gamma)
Even this, though, is not sufficient, as we cannot generate fresh, unique type variable names for use as flexible/unification variables:
freshId  ::  Id  --  it  is  impossible  for  fresh  to  return  a  different
freshId  =  ?       --  value  each  time!
To solve this problem, we could pass an (infinite) list of unique names around our program, and fresh could simply take a name from the top of the list, and return a new list with the name removed:
fresh  ::  [Id]  ->   ([Id],Id)
fresh   (x:xs)  =   (xs,x)
This is quite awkward though, as now we have to manually thread our list of identifiers throughout the entire inference algorithm:inferExp  ::  Gamma  ->  Exp  ->   [Id]  ->  Either  TypeError   ([Id],   (Type,  Gamma))
To resolve this, we bundle both the [Id] state transformer and the Either  TypeError  x error handling into one abstract type, called TC (defined in TCMonad.hs)
newtype  TC  a  =  TC   ([Id]  ->  Either  TypeError   ([Id],  a))
One can think of TC  a abstractly as referring to a stateful action that will, if executed, return a value of type a, or throw an exception.
As the name of the module implies, TC is a Monad, meaning that it exposes two functions (return and >>=) as part of its interface.
return   ::  a  ->  TC  a
return  =   . . .
(>>)     ::  TC  a  ->  TC  b  ->  TC  b
a  >>  b  =  a  >>=  const  b
(>>=)     ::  TC  a  ->   (a  ->  TC  b)  ->  TC  b
a  >>=  b  =   . . .The function return is, despite its name, just an ordinary function which lifts pure values into a TC action that returns that value. The function  (>>) (read then), is a kind of composition operator, which produces a TC action which runs two TC actions in sequence, one after the other, returning the value of the last one to be executed. Lastly, the function  (>>=), more general than  (>>), allows the second executed action to be determined by the return value of the first.
The TCMonad .hs module also includes a few built-in actions:
typeError  ::  TypeError  ->  TC  a  --  throw  an  error
freshId       ::  TC  Id                             --  return  a  fresh  type  variable  name
Haskell includes special syntactic sugar for monads, which allow for programming in a somewhat imperative style. Called do notation, it is simple sugar for  (>>) and  (>>=).
do  e                                      --                e
do  e;  v                               --                e  >>  do  v
do  p  <- e; v -- e >>=  \p  ->  do  v
do  let  x  =  y;  v             --                let  x  =  y  in  do  vThis lets us write unification and type inference quite naturally.  A simple example of the use of the TC monad is already provided to you, the specialise function, which takes a type with some number of quantifiers, and replaces all quantifiers with fresh variables (very useful in the type inference cases for variables, constructors, and primops):
specialise   ::  Scheme  ->  TC   (Type,  Suffix)
specialise   (Forall  xs  t)  =
do  ids  <- freshForall xs
return   (S .substRigid   (S .fromList   (map   (second  FlexVar)  ids))  t
, map   (flip   (,)  HOLE   .  snd)  ids)
To run a TC action in your top level infer function, the runTC function can be used:
runTC   ::  TC  a  ->  Either  TypeError  a
Please note: This function runs the TC action with the same source of fresh names each time! Using it more than once in your program is not likely to give correct results.
8 Program structure 
A program in MinHS may evaluate to any non-function type, including an aggregate type.  This is a valid MinHS program:
main  =   (1,(InL  True,  False));
which can be elaborated to the following type:
main  ::  forall  ’t .  (Int  *   ((Bool  +  ’t)  *  Bool))
=   (1,(InL  True,False));
8.1 Type information The most significant change to the language of assignment 1 is that the parser now accepts programs without any type information.  Type declarations are not compulsory!  Unless you are attempting the bonus parts of the assignment, you can assume that no type information will be provided in the program.
You can view the type information after your pass using --dump  type-infer.
9 Implementing Type Inference You are required to implement the function infer. Some stub code has been provided for you, along with some type declarations, and the type signatures of useful functions you may wish to implement. You may change any part of TyInfer .hs you wish, as long as it still provides the function infer, of the correct type. The stub code is provided only as a hint, you are free to ignore it.
10 Testing Your assignments will be autotested rigorously.  You are encouraged to autotest yourself.  MinHS comes with a tester script, and you can add your own tests to this. Your assignment will be tested by comparing the output Program of your infer function (not the output of tyinf  --dump  type-infer) against the expected output Program. Your solution must be α-equivalent to the expected solution.
Much like the previous assignment, you are given a suite of tests for Task  1. Unlike the previous assignment, the tests do not specify the expected results—note the lack of  .out files. Beware: unless you add .out specifying the expected result yourself, the tests will report success no matter what you return.
It is up to you to write your own tests for the extension tasks.  Any  .out files and additional tests you write are for your own benefit – you will not submit them for marking.
In this assignment we make no use of the later phases of the compiler.
11 Building MinHS 
Building MinHS is exactly the same as in Assignment 1.
To run the type inference pass and inspect its results, for cabal users type:
$  cabal  run  tyinf  --  --dump  type-infer  foo .mhs
Users of stack should type:
$  stack  exec  tyinf  --  --dump  type-infer  foo .mhsYou may wish to experiment with some of the debugging options to see, for example, how your program is parsed, and what abstract syntax is generated.  Many --dump flags are provided, which let you see the abstract syntax at various stages in the compiler.





         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
