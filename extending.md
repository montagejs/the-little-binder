
# Building on the FRB expression language

The FRB expression language has many faces.  These are the things that
FRB does with the expression language internally.

1. Parsing / Stringifying
1. Evaluation / Assignment
1. Observation / Binding
1. Algebra Simplification

In addition, we could:

1. Convert the expression into another language
1. Infer the type of an expression

Each of these operations boils down to a recursive function that
operates on nodes of the FRB syntax tree.

The form of the syntax tree is exhaustively described in the FRB README,
but in general, nodes are objects with a `type` property, and most have
an `args` property, the exceptions being terminal nodes like literals.
So, for example, the expression `2 + 2` has the form:

```javascript
var syntax = {type: "add", args: [
    {type: "literal", value: 2},
    {type: "literal", value: 2}
]}
```

From this syntax tree, we can create an evaluator, an observer, a
binder, or many other things.

1. `stringify` produces the normal-form of an expression in the FRB
   language.  At each node of the syntax tree, it combines the syntax of
   the arguments and its own syntax to create the aggregate expression.
   To do this, it sometimes needs to know the syntax of the parent node,
   to know whether it needs to parenthesize a subexpression because of
   precedence, and sometimes it needs to know the scope that the string
   will be evaluated in so that it can label "components".

2. `evaluate` accepts a scope and returns the corresponding value.  One
   way to implement `evaluate` would be to visit each node of the syntax
   tree, evaluate each of the arguments, and perform some operation to
   combine them.  In the case of `2 + 2`, the "literal" evaluator would
   return the `value` property of the syntax node and the "add"
   evaluator would combine `evaluate(args[0], scope) + evaluate(args[1],
   scope)`.

   However, in the interest of performance if there are frequently
   evaluated expressions, `evaluate` is implemented in terms of
   `compileEvaluator` which produces `evaluate` functions for each node
   of the syntax tree.

Suppose that you were going to implement a function that translates a
subset of FRB expressions into SQL queries.  For example, given some
table like `People`, `filter{name.startsWith("A")}.sorted{name}` would
be translated to `SELECT * FROM People WHERE name LIKE "A%" SORTED BY
name`.

Such a method, `toSql`, would need to know the syntax and the database
schema, particularly the table on which the expression will be
evaluated.

`toSql(syntax, table)`

The syntax tree for the FRB expression would be:

```javascript
var syntax = {type: "sortedBlock", args: [
    {type: "filterBlock", args: [
        {type: "value"},
        {type: "startsWith", args: [
            {type: "property", args: [
                {type: "value"},
                {type: "literal", value: "name"}
            ]},
            {type: "literal", value: "A"}
        ]}
    ]},
    {type: "property", args: [
        {type: "value"},
        {type: "literal", value: "name"}
    ]}
]}
```

One approach would be to translate this into a syntax tree for the SQL
and then stringify that to SQL.

```javascript
var syntax = {
    type: "select",
    where: {type: "like", args: [
        {type: "property", args: [
            {type: "value"},
            {type: "literal", value: "name"}
        ]},
        {type: "literal", value: "A%"}
    ]},
    sortedBy: [
        {type: "property", args: [
            {type: "value"},
            {type: "literal", value: "name"}
        ]}
    ]
};
```

It will probably be handy in the course of doing this to have a SQL
syntax type with methods like `where` and `sortedBy` that would create a
copy of the current syntax and extend it.

```javascript
function SqlSyntax(type) {
    this.type = type || "select";
    this.where = null;
    this.sortedBy = null;
}

SqlSyntax.prototype.clone = function () {
    var clone = new SqlSyntax();
    clone.type = this.type;
    clone.where = this.where;
    clone.sortedBy = this.sortedBy;
    return clone;
};

SqlSyntax.prototype.where = function (syntax) {
};

SqlSyntax.prototype.sortedBy = function (syntax) {
    
};

```

To be continued…

