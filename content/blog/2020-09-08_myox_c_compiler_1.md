+++
title = "MYOX: C compiler: 1"
description = "Lets implement C compiler from scratch with Rust. Chapter 1"
+++

[MYOX: what does it mean?](@/blog/2020-06-29_myox.md)  
[Previous chapter](@/blog/2020-09-07_myox_c_compiler_0.md)

## What will we do?

All big things are built from smaller and more simple.
In the next few chapters we basically will build a texted calculator. It means that we will be able to write a program which consists of mathematical operations and could be compiled into assembly languge which after that can be executed.
But as we will see later the new functionality to our "calculator" will be appended in a trivial way(at least most of it) since with "calculator" we will build a foundation of our compiler.

## Natural languages, programming languages

>"int foo = 10;"  
>"My name is Konstantin."  

What do these languages have in common? A lot of.  

Every languge consists of symbols.  
Every symbol has a meaning.  
Some symbols can be combined together so that we achieve the new meaning. We can say that a group of symbols with the meaning is a word.  
Words also can be combined together, and a group of words can have the new meaning. We can say that the group of words is a sentence(in our natural language), or statement, expression(in programing languages. Don't bother we will talk about that in the future in a more detail).  
Sentences can be combined into parapraphs. Statements can be combined into functions.  
Parapraphs can be combined into chapters. Functions could be combined into modules.  
Chapters form a text. Modules form a program.

I hope that now you see that actually there are a lot of common. The boundaries of differences blur.  
I suggest to reflect on this.  

## Scanner

Now lets start to do first steps with our compiler:

```bash
# c_compiler(project root)

cargo new 01_scanner
# open Cargo.toml and append "01_scanner" as a workspace member
cd 01_scanner
```

As I said previously our language consists of symbols. Symbols can be combined together into words. So that lets treat our program(text file) as a sequence of symbols. We will read all symbols from left to right while trying to group them into words when it makes sense or left as a separate self sufficient thing or fail when there is a sequence which is not possible in our language. The abstraction which is responsible for doing such stuff we will call **Scanner**.

Lets create a new file *scan.rs* in *src/*:

```bash
touch src/scan.rs
# append "mod scan;" to src/main.rs
```

Now lets introduce a type which will describe every known symbol in our current version the language.

```rust
#[derive(PartialEq, Debug)]
enum Token {
    TPlus,
    TMinus,
    TStar,
    TSlash,
    TIntlit(usize),
}
```

I hope that you see the basic idea behind **Token** and **Scanner**. In order to clarify it more lets begin with tests. They should demonstrate more precisely what we want to achieve and also it's a good practice which should be followed when it's possible.

```rust
// scan.rs

#[cfg(test)]
mod test {
    use super::Token::*;
    use super::*;

    fn get_tokens_list(i: &str) -> Vec<Token> {
        let mut scanner = Scanner::new(&i);

        let mut ts = vec![];
        while let Some(t) = scanner.scan() {
            ts.push(t);
        }

        ts
    }

    fn compare_expected_with_scanned(expected: Vec<Token>, scanned: Vec<Token>) {
        assert_eq!(expected.len(), scanned.len());

        expected.iter().zip(scanned.iter()).for_each(|(l, r)| {
            assert_eq!(l, r);
        });
    }

    #[test]
    fn test_scan1() {
        let i = "2 + 3 * 5 - 8 / 3";

        compare_expected_with_scanned(
            vec![
                TIntlit(2),
                TPlus,
                TIntlit(3),
                TStar,
                TIntlit(5),
                TMinus,
                TIntlit(8),
                TSlash,
                TIntlit(3),
            ],
            get_tokens_list(i),
        );
    }

    #[test]
    fn test_scan2() {
        let i = "13 -6+  4*
5
       +
08 / 3";

        compare_expected_with_scanned(
            vec![
                TIntlit(13),
                TMinus,
                TIntlit(6),
                TPlus,
                TIntlit(4),
                TStar,
                TIntlit(5),
                TPlus,
                TIntlit(8),
                TSlash,
                TIntlit(3),
            ],
            get_tokens_list(i),
        );
    }

    #[test]
    fn test_scan3() {
        let i = "12 34 + -56 * / - - 8 + * 2";

        compare_expected_with_scanned(
            vec![
                TIntlit(12),
                TIntlit(34),
                TPlus,
                TMinus,
                TIntlit(56),
                TStar,
                TSlash,
                TMinus,
                TMinus,
                TIntlit(8),
                TPlus,
                TStar,
                TIntlit(2),
            ],
            get_tokens_list(i),
        );
    }

    #[test]
    fn test_scan4() {
        let i = "23 +
18 -
45.6 * 2
/ 18";

        compare_expected_with_scanned(
            vec![
                TIntlit(23),
                TPlus,
                TIntlit(18),
                TMinus,
                TIntlit(45),
                TStar,
                TIntlit(2),
                TSlash,
                TIntlit(18),
            ],
            get_tokens_list(i),
        );
    }

    #[test]
    #[should_panic]
    fn test_scan5() {
        let i = "23 * 456abcdefg";

        compare_expected_with_scanned(vec![TIntlit(23), TStar, TIntlit(456)], get_tokens_list(i));
    }
}

```

Now when we have a shape of our Scanner lets implement it.  
Algorthmic idea behind Scanner: lets read character one by one. When we know that it's self sufficient character(e.g. "+") lets create an appropriate token. When we think that the current character is a part of some more big thing lets move forward until the whole big thing isn't built(e.g. we read "1" of number "123". Now we need to read "2" after that "3" and after that say that we have a number literal token = 123). Also we should take care about useless characters such as empty lines, spaces. Sometimes we can receive the wrong sequence of symbols, the sequence which we can't recognize(e.g. "456abcdefg") in such cases we just abort compilation(we will add an error handling in the future).  
I think that based on the above discription and tests you can implement Scanner on your own and I encourage you to do that.

```rust
// scan.rs

use std::{iter::Peekable, str::Chars};

// our scanner basically refers to a read text file, its contents via stream of chars
// while advancing in reading scanner fills tokens vector, 
// basic blocks with the meaning of our program
// for advancing we will use built in rust Peekable stream.
// it offers us out of the box the ability to advance over characters, 
// see the next without advancing
struct Scanner<'a> {
    stream: Peekable<Chars<'a>>,
    tokens: Vec<Token>,
}

impl<'a> Scanner<'a> {
    // since we just read, but not modify and  
	// the read text basically should leave at least equal to scanner's lifetime 
	// we void unnecessary copying 
    pub fn new<S: AsRef<str>>(stream: &'a S) -> Self {
        Self {
            stream: stream.as_ref().chars().peekable(),
            tokens: vec![],
        }
    }

    // as it was said lets skip whitespaces and
	// pay attention only on the useful stuff
    fn next(&mut self) -> Option<char> {
        self.skip_whitespace();
        self.stream.next()
    }

    fn skip_whitespace(&mut self) {
        while let Some(' ') | Some('\t') | Some('\n') | Some('\r') = self.stream.peek() {
            self.stream.next();
        }
    }

    pub fn scan(&mut self) -> Option<Token> {
        let ch = self.next();

        if None == ch {
            return None;
        }

        match ch.unwrap() {
            '+' => Some(Token::TPlus),
            '-' => Some(Token::TMinus),
            '*' => Some(Token::TStar),
            '/' => Some(Token::TSlash),
            ch1 @ '0'..='9' => {
                let mut num = format!("{}", ch1.to_owned());
                while let Some(ch2 @ '1'..='9') = self.stream.peek() {
                    num = format!("{}{}", num, ch2);
                    self.stream.next();
                }

                // skip all after dot when float
                while let Some('1'..='9') | Some('.') = self.stream.peek() {
                    self.stream.next();
                }

                Some(Token::TIntlit(num.parse::<usize>().unwrap()))
            }
            _ => panic!(),
        }
    }
}
```

## Conclusion

In this chapter we discussed how we can treat our program, its structure, compared with natural language and saw that there are a lot of similarities.  
Next we implemented Scanner, the thing which receives a text of the program(sequence of symbols) and splits it into Tokens.  

In the next chapter we will meet with Parser. The thing which tries to understand the meaning of Tokens(= words) sequence and based on that do appropriate actions.

