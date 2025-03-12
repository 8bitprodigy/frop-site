# Hi-C

### A call for an Improved/Expanded C

(Public Domain/CC0) 2025, Chris DeBoy

![hi-c.png](hi-c.png)

## Rationale

C is great -- but it is lacking in metaprogramming features. There are preprocessor directives that enable an impressive amount of metaprogramming, but only with enough effort. Nim, on the other hand, has among the most powerful metaprogramming feature sets of any modern language. The goal is to bring them to C as part of the language proper, rather than as part of the preprocessor, but also keeping the implementation conservative and minimal. 

Also like Nim, this compiler would simply compile any Hi-C exclusive constructs to standard C code, but since it would only be a superset of C, all the regular C code would just "fall through" to the output C files, making said compiler much simpler than that of Nim's. This would also bring the benefit of being able to use any existing C library in Hi-C without use of a wrapper, though wrapping it to make it more idiomatic for Hi-C would certainly be better.

## Additions

In order to facilitate this level of metaprogramming power, a few additions will need to be made to C's keywords and semantics.

### Keywords:

- `macro` - Provides deep AST manipulation.
  Syntax:
  
  ```c
  macro IDENTIFIER(ARGS...)
  {
      /* Hi-C code here if necessary to 
         be computed before expansion 
      */
      return `
      /* Quoted Hi-C and/or standard C 
         code to be inserted in place of 
         the macro call goes here 
      */
      `;
  }
  ```

- `transform` - Allows syntax extensions by rewriting code patterns at compile time.
  
  ```c
  transform IDENTIFIER [CODE PATTERN]
  {
      /* Hi-C code here if necessary to 
         be computed before transformation 
      */
      return `
      /* Quoted Hi-C and/or standard C 
         code to be inserted in place of 
         the pattern goes here 
      */
      `;
  }
  ```

- A variation of `typeof()` that returns structured type information, including base type, function pointer detection, and AST representation.

### Semantics:

#### "Quoted" code using backticks(`) for macros and transforms

Macros and transforms are able to return chunks of Hi-C and the full set of the C language by wrapping them in single backticks. This code will be inserted where the macro is declared or transform pattern is found, and then expanded into standard C in a later pass.

#### Argument/Operator/Keyword(maybe) overloading:

- *Argument Overloading* - Allows multiple functions with the same symbol to be declared, so long as their signature and/or return type differ.
  Usage:
  
  ```c
  int 
  myFunc(int x) 
  {
      /* Function definition here */    
  }
  
  int
  myFunc(int x, float y)
  {
      /* Function definition here */
  }
  ```

- *Operator Overloading* - Allows new functionality to be added to operators to enable more ergonomic expression of operations with complex data types. To declare an overloading function using an operator, the operator must be wrapped in backticks (`` ` ``).
  Syntax:
  ``TYPE `OPERATOR`(ARGS...) {FUNCTION BODY}``
  Usage:
  
  ```c
  /* assuming the following struct was typedef'd:
  struct
  Vec2
  {
      double x;
      double y;  
  } Vec2;
  */
  
  Vec2 
  `+`(Vec2 vec2_1, Vec2 vec2_2)
  {
      return (Vec2){
          .x = vec2_1.x + vec2_2.x,
          .y = vec2_1.y + vec2_2.y
      };
  }
  
  Vec2 v1 = (Vec2){.x = 1.0f, .y = 1.0f};
  Vec2 v2 = (Vec2){.x = 2.0f, .y = 2.0f};
  
  Vec2 v3 = v1 + v2;
  ```

- *Keyword Overloading (maybe)* - Just floating this one. May be too powerful for prime time.

#### Expanded execution of compile time computations

- Maybe functions declared as `const` are computed at compile time?

- Macros and Transforms can contain a subset of C + Hi-C that is evaluated at compile time.

### Other possible additions

- More straightforward, fixed-width types as first class, i.e. `int8`, `uint16`, `float64`, etc, which would expand to their C counterparts in `stdint.h` or whichever library defines them.

- Support for runes, a.k.a. 16-bit characters.

- Indexing declarations on a per-scope basis on first pass in order to eliminate the need for forward-declarations. This could eliminate the need for writing headers with the addition of the...

- `export` specifier which makes any definition using it accessible from outside source files, making it a modular language. This would also imply the existence of `import` statements

- With this modular system, perhaps the compiler could recursively parse through the import statements(as well as `#include` when standard C libraries and/or code files are used) starting with those in the source file that defines `int main()` to check for what source files are needed to build the project and produce not only the expanded C files, but a makefile as well, which would not only eliminate the need for complex build systems like `cmake`, `m4`, `auto(conf/make)`, etc., but also enforce that `make` is used, ensuring greater compatibility and portability of programs written in Hi-C
