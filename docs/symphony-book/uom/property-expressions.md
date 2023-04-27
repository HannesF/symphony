# Property Expressions

Symphony Solution [```Components```](./solution.md#componentspec) can use property expressions in its property values. Property expressions are evaluated at Solution [Vendor](../vendors/overview.md) level, which means they are immediately evaluated once they arrive at the Symphony API surface. Hence, none of the [Managers](../managers/overview.md) or [Providers](../providers/overview.md) need to worry about (or be aware of) the expression rules. When authoring Symphony artifacts, a user can use these expressions in their Solution documents.

In general, Symphony attempts to pare the property values as close as strings as possible. Only when Symphony detects clear and valid arithmetical expressions and function calls, it will try to evaluate them first before the string evaluation. This means Symphony is mostly tolerable to syntax errors in expressions and will treat those as string literals, unless there are clear errors such trying to divide by zero. For example, ```"10/"``` is allowed as Symphony assumes ```/``` is used as a forward-slash. However, ```"10/0"``` is disallowed as it leads to division by zero. ```"'10/0'"``` is allowed as it’s a quoted string.

## Constants

* Integer constants are evaluated as numbers when possible.
* Float numbers are always treated as strings. However, if a float is the result of evaluation (such as ```5/2```), it’s treated as number till no longer possible. Hence ```5/2+1``` is ```3.5``` while ```5/2+a``` is ```2.5a```.
* Negative integers are allowed.
* String constants can't have spaces between words. Surrounding spaces are trimmed. Words after the first space are omitted. To use spaces between words, surround the string literal with single quote(```'```).
* For single-quoted strings, single quotes are removed. For double-quoted strings, double quotes are kept as they are.


The following table summarizes how constants are evaluated:

| Expression | Value | Comment |
|--------|--------|--------|
| ```"1"``` | ```"1"``` | Intergers are treated as strings, when no further evalutaion is possible |
| ```"3.14"``` | ```"3.14"``` | Floats are always treated as strings
| ```" "``` | ```""``` | Spaces are trimmed |
| ```"  abc   "``` | ```"abc"```| Spaces are trimmed |
| ```"abc def"```| ```"abc"```| Words after first space are omitted|
| ```'abc def'```|```"abc def"``` | Single-quoted string|
| ```'abc def```|```"'abc def"``` | Single-quote not closed, returns as it is|
| ```'abc def'hij```| ```"abc def"```| Single-quoted part is parsed, the rest is omitted|

## Operators
* Plus(```+```) is treated as numeric addition, if both sides of ```+``` evaluate to numbers. Otherwise, it’s treated as string concatenation. 
* Minus (```-```) is treated as numeric addition, if both sides of ```-``` evaluate to numbers. Otherwise, it’s treated as a dash (```-```). 
* Multiplication (```*```) is treated as numeric multiply, if both sides of ```-``` evaluate to numbers. If the left of * is a string and the right of * is a number, the string is repeated the number of times. Otherwise, it’s treated as a star (```*```). 
* Divide (```-```) is treated as numeric addition, if both sides of ```-``` evaluate to numbers (divide by zero is not allowed). Otherwise, it’s treated as a forward-slash (```/```). 
* Use parenthesis (```()```) to change calculation precedence. Expressions in parenthesis are evaluated first.

The following table summarizes how operators work:
| Expression | Value | Comment |
|--------|--------|--------|
| ```"+2"```| ```"2"```| Unary plus|
| ```"-3"```| ```"-2"```| Unary minus|
| ```"-abc"```| ```"abc"```| Dash ABC |
| ```"abc-"```| ```"abc-"```| ABC dash |
| ```"+a"``` | ```"a"```| (empty) plus a |
| ```"a+"``` | ```"a"```| a plus (empty) |
| ```"1+2"```| ```"3"```| Addition (integers are treated as numbers when possible)|
| ```"1.0+2.0"```|```"1.02.0"```| Concatenation (floats are always treated as strings)|
| ```"1-2"```|```"-1"```| Subtraction |
|  ```"3*-4""```|```"-12"```| Multiplication|
| ```"abc*3"```|```"abcabcabc"```| Repeat abc 3 times |
| ```"abc*-3"```|```"abc*-3"```| Repeating -3 times is impossible, return as a string|
| ```"abc*0"```| ```""```| Repeat 0 times |
| ```"abc*(5/2)"```| ```"abcabc"```| Repeat floor(5/2) times |
|```"3-(1+2)/(2+1)"```|```"2"```| Parentheses are evaluted first|


> **NOTE:** **Why we treat integers as numbers when possible, and floats always as strings?** We allow integer calculations for cases like calculating an offset and other simple number manipulations. However, we don’t aim to offer a full-scale scripting language, and floats in our contexts are often version numbers like “```1.2```” and “```1.2.3```”. Hence we always treat floats as strings. Of course, you can do things like “```1.(3+4)```”, which evaluates to “```1.7```” because at ```(3+4)``` numerical evaluation of integer expression is still possible.
## Functions
Symphony supports a few functions for artifact content overrides, as summarized by the following table.

When these functions are used, a valid ```EvaluationContext``` is required, which injects a secret provider, a configuration provider, and a Symphony deployment object as evaluation context. Failing to provide required context causes an evaluation error.

| Function | Behavior|
|--------|--------|
|```$config(<config object>, <config key>)``` | Reads a configuration from a config provider |
|```$secret(<secret object>, <secret key>)```| Reads a secret from a secret store provider |

## Using Operators Alone

We try to parse properties as closely as strings as possible with limited calculations and functions calls allowed. When operators are used out of the context of an expression, they are evaluated differently. Although the following are unlikely scenarios, we present how they are evaluated following the above evaluation rules.

* When used alone, a period (```.```) are returned as it is, such as ```.``` and ```...``` are returned as they are.
* When used alone, a plus(```+```) is treated as a unary operator, which means a single, or a consecutive ```+``` signs are evaluated to empty strings, as “plus nothing” is still “nothing”.
* When used alone, a minus(```-```) is treated as a unary operator, which means a single ```-``` is evaluated to empty string, as “minus nothing” is “nothing”. However, when you use two minus signs, the second minus is treated as “dash”, and a negative “dash” is still “dash”. Hence ```--``` evalutes to ```-```. 
* When used alone, a forward-slash (```/```) are returned as it is, such as ```/``` and ```///``` are returned as they are.
