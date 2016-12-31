Built-in Types JavaScript defines seven built-in types: • null • undefined • boolean • number • string • object • symbol—added in ES6

Many developers will assume “undefined” and “undeclared” are roughly the same thing, but in JavaScript, they’re quite different. undefined is a value that a declared variable can hold. “Undeclared” means a variable has never been declared

JavaScript strings are immutable, while arrays are quite mutable

Converting a value from one type to another is often called “type casting,” when done explicitly, and “coercion” when done implicitly (forced by the rules of how a value is used

var a = 3 * 6; var b = a; b; In this snippet, 3 * 6 is an expression (evaluates to the value 18). But a on the second line is also an expression, as is b on the third line. The a and b expressions both evaluate to the values stored in those variables at that moment, which also happens to be 18. Moreover, each of the three lines is a statement containing expressions. var a = 3 * 6 and var b = a are called “declaration statements” because they each declare a variable (and optionally assign a value to it). The a = 3 * 6 and b = a assignments (minus the vars) are called assignment expressions

The third line contains just the expression b, but it’s also a statement all by itself (though not a terribly interesting one!). As such, this is generally referred to as an “expression statement.
