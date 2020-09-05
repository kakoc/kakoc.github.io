+++
title = "MYOX: C compiler: 2"
description = "Lets implement C compiler from scratch with Rust. Chapter 2"
+++

[MYOX: what does it mean?](@/blog/2020-06-29_myox.md)  
[Previous chapter](@/blog/2020-09-08_myox_c_compiler_1.md)

## What we've done before

In the previous chapter we firstly looked at what actually a programming language is(from the high level) and implemented a compiler part which is called Scanner. Scanner scans initial text of the program and splits it into tokens aka lexemes.  

## What will we do?

In this chapter we are going to implement Parser. The idea behind it is to traverse tokens and try to treat a group of them as a something meaningful and if so do appropriate actions.

## Parser

Lets create a new version of our compiler:

```bash
# c_compiler root

cargo new 02_parser
# add 02_parser to Cargo.toml as a workspace member

# copy all code from the previous chapter
cp -r 01_scanner/src/* 02_parser/src/

cd 02_parser/src
touch parser.rs
# add "mod parser" to main.rs
```

Now lets talk a bit about what we are going to do.  
Look, the idea behind the parser is to treat a sequence of tokens as a something meaningful. But how actually we know that that combination makes sense but another doesn't? And the answer is - we have a language and our language has rules. Lets specify them:  

```
expression: number
            | expression '*' expression
            | expression '/' expression
            | expression '+' expression
            | expression '-' expression
            ;

number:     T_INTLIT
            ;
```

After first look it could be strange, but don't worry lets walk through it. Firstly that scheme notation has a particular name: [BNF](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form). And the idea behind it is to show generically our language grammar via a recursion and substitution.  
Suppose we have a program "2 + 2 * 2". Lets try to figure out whether our language can understand that sequence. After Scanner we have something like:  

"[TI(2), TS(+), TI(2), TS(\*), TI(2)]".  
With our grammar: firstly we read "2" but it's not the last symbol, so we can't to stop. If "2" is not a number it can be probably an expression(| stands for or). So we see that the expression could be also a number. So that we can use it for our "2". But we don't have a single expression rule inside root expression(it's always used with "| (+ | - | * | / |) expression"). So we need to read the next symbol. The next symbol is "+". And we have so rule(expression '+' expression). We see that there are no other rules which can be used so we need to try to expand that one. We've read "2", "+", after that we read 2. To can be number or expression, but we haven't finished so lets try find something other. Behind "2" there is "*" and we see that there is "expression '*' expression" rule. Lets try to expand it. We've read "2 + 2 *" no we see "2". It's the last symbol and we have a rule that it could be the number, and number can be the expression. So that it fits to our last "expression '*' expression" rule.

I hope that that explanation can clarify the basic idea. In future chapters via such notation we will expand our language with new constructs and also solve other tasks(such as precedence of "*" over "+").

Now, after the explanation of the core idea I want to begin with tests we can show more info about what do we want to achieve.

```rust
// parser.rs

#[test]
fn test_parser_eval() {
    let i = "2 + 3 * 5 - 8 / 3";
    let expected = 11;

    let res = interpret_ast(Parser::new(get_tokens_list(i)).parse());

    assert_eq!(expected, res);

    let i = "13 -6+  4*
5
       +
08 / 3";
    let expected = -21;

    let res = interpret_ast(Parser::new(get_tokens_list(i)).parse());

    assert_eq!(expected, res);
}

#[test]
#[should_panic]
fn test_parser_invalid_program() {
    let i = "12 34 + -56 * / - - 8 + * 2";
    let res = interpret_ast(Parser::new(get_tokens_list(i)).parse());
}
```

An attentive reader may notice that currently our evaluation doesn't take care about the precedence. As I said earlier we will do it in the next chapters.

Now lets talk about what actual output from Parser is. The output from Parser is a famous Abstract Syntax Tree term. When we hear "tree" we always can think as a recursive nature about it. And now it will be very useful. As I demonstrated earlier when parsing the sequence of tokens we work with a recursive nature. Lets slowly begin: 

```rust
use super::scan::{Scanner, Token as ScannerToken};
use std::iter::Peekable;

// every meaningful token from Scaner has a corresponding tree node
struct AstNode {
    token: Token,
    left: Option<Box<AstNode>>,
    right: Option<Box<AstNode>>,
}

// just keeping pointer on spanning elements advance through them
struct Parser {
    scanned: Peekable<Box<std::vec::IntoIter<ScannerToken>>>,
}

impl Parser {
	pub fn new(tokens: Vec<ScannerToken>) -> Self {
		Parser {
            scanned: Box::new(tokens.into_iter()).peekable(),
        }
    }
}
```

Now we are going to begin to implement [Recursive Decent Technique](https://en.wikipedia.org/wiki/Recursive_descent_parser). The basic idea is that we somehow try to go from top to down over the rules(from most complex to the most simple) and from the bottom begin to build our actual ast nodes.

```rust
+enum Token {
+    AAdd,
+    ASubtract,
+    AMultiply,
+    ADivide,
+    AIntlit(i64),
+}

// new methouds inside Parser

// as i said we begin with most complex to the most simple
pub fn parse(&mut self) -> AstNode {
	self.bin_expr()
}

fn bin_expr(&mut self) -> AstNode {
	let left = self.primary();

	if let None = self.scanned.peek() {
		return left;
	}

	let middle = arithop(&self.scanned.next().unwrap());

	let right = self.bin_expr();

	AstNode {
		token: middle,
		left: Some(Box::new(left)),
		right: Some(Box::new(right)),
	}
}

fn primary(&mut self) -> AstNode {
	match self.scanned.next() {
		Some(ScannerToken::TIntlit(v)) => make_ast_leaf(Token::AIntlit(v)),
		_ => panic!("invalid token"),
	}
}

// helpers to make life easier
fn make_ast_leaf(token: Token) -> AstNode {
    AstNode {
        token,
        left: None,
        right: None,
    }
}

fn make_ast_unary(token: Token, left: Box<AstNode>) -> AstNode {
    AstNode {
        token,
        left: Some(left),
        right: None,
    }
}

fn arithop(token: &ScannerToken) -> Token {
    use super::scan::Token as ScannerToken;

    match *token {
        ScannerToken::TPlus => Token::AAdd,
        ScannerToken::TMinus => Token::ASubtract,
        ScannerToken::TStar => Token::AMultiply,
        ScannerToken::TSlash => Token::ADivide,
        _ => panic!("unknown token in arithop()"), // fprintf(stderr, "unknown token in arithop() on line %d\n", Line);
    }
}

fn get_tokens_list(i: &str) -> Vec<ScannerToken> {
    let mut scanner = Scanner::new(&i);

    let mut ts = vec![];
    while let Some(t) = scanner.scan() {
        ts.push(t);
    }

    ts
}
```

Finally we need a way how we can actually intrepret the ast.

```rust
fn interpret_ast(root: AstNode) -> i64 {
    if let Token::AIntlit(v) = root.token {
        return v;
    } else {
        let left = interpret_ast(*root.left.unwrap());
        let right = interpret_ast(*root.right.unwrap());

        match root.token {
            Token::AAdd => left + right,
            Token::ASubtract => left - right,
            Token::AMultiply => left * right,
            Token::ADivide => left / right,
            _ => panic!("invalid operations tree"),
        }
    }
}
```

## Conclusion

In this chapter we explored what actually parsing process is, how can do it as well as how we can describe our language in a more formal, strict way.  
I'm sure that there are a lot of questions about recursive decent parsing. And based on my experience you won't understand it without playing with it. So I suggest to as an exercise implement a pretty printer - the tool which traverses tokens from scanner and print the ast in the appropriate way. 

## Next steps

In the next chapters we will fix the precedence problem and finally begin to generate an assembly code.
