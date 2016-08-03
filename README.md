# Tokenize

**Note:** This package does not currently work well with transpose and conjugate transpose operators like in `A' * b` and will try to tokenize this as the beginning of a `CHAR` literal.

`Tokenize` is a Julia package that serves a similar purpose and API as the [tokenize module](https://docs.python.org/3/library/tokenize.html) in Python but for Julia. This is to take a string or buffer containing Julia code, perform lexical analysis and return a stream of tokens.

The goals of this package is to be

* Fast, it currently lexes all of Julia source files in less than a second (1.6 million Tokens)
* Round trippable, that is, from a stream of tokens the original string should be recoverable exactly.
* Non error throwing. Instead of throwing errors a certain error token is returned.

### API

#### Tokenization

The function `tokenize` is the main entrypoint for generating `Token`s.
It takes a string or a buffer and creates an iterator that will sequentially return the next `Token` until the end of string or buffer.

```jl
julia> collect(tokenize("function f(x) end"))
9-element Array{Tokenize.Tokens.Token,1}:
 1,1-1,9:   KEYWORD "function"
 1,9-1,10:   WHITESPACE " "
 1,10-1,11:   IDENTIFIER    "f"
 1,11-1,12:   LPAREN    "("
 1,12-1,13:   IDENTIFIER    "x"
 1,13-1,14:   RPAREN    ")"
 1,14-1,15:   WHITESPACE    " "
 1,15-1,18:   KEYWORD   "end"
 1,18-1,18:   ENDMARKER ""
```

#### `Token`s

Each `Token` is represented by where it starts and ends, what string it contains and what type it is.

The API for a `Token` (non exported from the `Tokenize.Tokens` module) is.

```julia
startpos(t)::Tuple{Int, Int} # row and column where the token start
endpos(t)::Tuple{Int, Int} # row and column where the token ends
startbyte(T)::Int64 # byte offset where the token start
endbyte(t)::Int64 # byte offset where the token ends
untokenize(t)::String # the string representation of the token
kind(t)::Token.Kind # The type of the token
exactkind(t)::Token.Kind The exact type of the token
```

The difference between `kind` and `exactkind` is that `kind` returns `OP` for all operators while `exactkind` returns a unique type for all different operators, ex;

```jl
julia> tok = collect(tokenize("⇒"))[1];

julia> Tokenize.Tokens.kind(tok)
OP::Tokenize.Tokens.Kind = 60

julia> Tokenize.Tokens.exactkind(tok)
RIGHTWARDS_DOUBLE_ARROW::Tokenize.Tokens.Kind = 129
```
