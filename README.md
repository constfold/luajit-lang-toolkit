LuaJIT Language Toolkit
===

The LuaJIT Language Toolkit is a Lua implementation of the Lua programming language itself.
It generates LuaJIT's bytecode complete with debug informations.
The generated bytecode, in turn can be run by the LuaJIT's virtual machine.

On itself this tookit does not do anything useful since LuaJIT is able to generate and run the bytecode for any Lua program.
The purpose of the language toolkit is to provide a starting point to implement a programming language that target the LuaJIT virtual machine.

With the LuaJIT Language Toolkit is easy to create a new language or extend the Lua language because the parser is cleanly separated from the bytecode generator and the virtual machine run time environment.

The toolkit implement actually a complete pipeline to parse a Lua program, generate an AST tree and generate the bytecode.

Lexer
---

Its role is to recognize lexical elements from the program text.
It does take the text of the program as input and does produce a flow of "tokens".

Using the language toolkit you can run the lexer only to examinate the flow of tokens:

```
luajit run-lexer.lua tests/test-1.lua
```

to obtain a list of the tokens:

    TK_local
    TK_name	x
    =
    {
    }
    TK_for
    TK_name	k
    =
    TK_number	1
    ,
    TK_number	10
    TK_do
    TK_name	x
    [
    TK_name	k
    ]
    =
    TK_name	k
    *
    TK_name	k
    +
    TK_number	1
    TK_end

Each line represent a token where the first element is the kind of token and the second element is its value, if any.

The Lexer's code is an almost literal translation of the LuaJIT's lexer.

Parser
---

The parser takes the flow of tokens as given by the lexer and form the statements and expressions according to the language's grammar.
The parser takes a list of user supplied rules that are invoked each time a parsing rule is completed.
The user's module can return a result that will be passed to the other rules's invocation.

For example, the grammar rule for the "return" statement is:

```
explist ::= {exp ','} exp

return_stmt ::= return [explist]
```

In this case the toolkit parser rule will parse the optional expression list by calling the function `expr_list`.
Then, once the expressions are parsed the user's rule `ast:return_stmt(exps, line)` will be invoked by passing the expressions list obtained before.

```lua
local function parse_return(ast, ls, line)
    ls:next() -- Skip 'return'.
    ls.fs.has_return = true
    local exps
    if EndOfBlock[ls.token] or ls.token == ';' then -- Base return.
        exps = { }
    else -- Return with one or more values.
        exps = expr_list(ast, ls)
    end
    return ast:return_stmt(exps, line)
end
```

As you cas see the user's parsing rules are invoked using the `ast` object.

With the LuaJIT Language Toolkit a set of rules are defined in "lua-ast.lua" to build the AST of the program.

In addition the parser provides additional informations about:

* the function prototype
* the syntactic scope

The first is used to keep trace of some informations about the current function parsed.

The syntactic scope rules tell to the user's rule when a new syntactic block begins or end.
Currently this is not really used by the AST builder but it can be useful for other implementations.

The Abstract Syntax Tree (AST)
---

The abstract syntax tree represent the whole Lua program with all the informations.
If you implement a new programming language you can implement some transformations of the AST tree if you need.
Currently the language toolkit does not perform any transformation and just pass the AST tree to the bytecode generator module.

Bytecode Generator
---

Once the AST tree is generated it can be feeded to the bytecode generator module that will generate the corresponding LuaJIT bytecode.

The bytecode generator is based on the original work of Richard Hundt for the Nyanga programming language.
It was greatly modified by myself to produce optimized code similar to what LuaJIT generate itself.

Running the Application
---

The application can be run with the following command:

```
luajit run.lua <filename>
```

The "run.lua" script will just invoke the complete pipeline of the lexer, parser and bytecode generator and it will pass the bytecode to luajit with "loadstring".

Since this is a didactic application the bytecode is also shown on the screen but this can be easily disabled if needed.