# What's this repository for?

[Rust is my first language](https://github.com/Dhghomon/programming_at_40/blob/master/README.md), and recently I've begun delving into languages in the Rust periphery, one of which is F# (since the Rust compiler was originally written in OCaml and it incorporates a good deal of its syntax).

Almost every F# manual is written for C# developers in mind. That's fine, but I have no real familiarity with C#. And when I look at F# in practice I see Rust everywhere: Options, Results, match statements, iterators, and so on and so forth. An F# guide that's readable for Rustaceans that also serves as a Rust guide for F# users seems to be a necessity. So let's put one together.

I will be adding to this whenever an idea strikes me, but note that I am still very weak at F# and would appreciate PRs for anything I get wrong or haven't explained sufficiently.

That's enough for an intro. Let's start!

# Primitive types

Both Rust and F# have a compiler that uses type inference. They also both use `let` bindings. So if you write the following:

```
let x = 9;
```

It will make `x` an `i32` in Rust, which in F# is an `int`...which is also a 32 bit signed integer. By the way, F# doesn't use a semicolon at the end (so don't use one) but the compiler also won't yell at you if you do.

Let's look at some primitive types:

* Rust `u8`, an 8-bit unsigned integer. In F# this is a `byte`, equivalent to the .NET type `Byte`.
* Rust `u16` in F# is a `uint16` (.NET: `UInt16`)
* Rust `u32` in F# is a `uint` (.NET: `UInt32`)
* Rust `u64` in F# is a `uint64` (.NET: `UInt64`)
* Rust `i8`, an 8-bit signed integer. In F# this is an `sbyte`, equivalent to the .NET type `SByte`.
* Rust `i16` in F# is an `int16` (.NET: `Int16`)
* Rust `i32` in F# is an `int` (.NET: `Int32`)
* Rust `i64` in F# is an `int64` (.NET: `Int64`)
* Rust `char` in F# is a `char` (.NET `Char`). Rust `char` is UTF-8, while in F# they are UTF-16.
* Rust `unit` in F# is also `unit`. They both use `()` to represent it. This is one of those types that is very welcome to see for a Rustacean. (Almost) everything is an expression!
* Rust `String` in F# is `string`. The F# documentation calls it "a sequential collection of characters that's used to represent text. A String object is a sequential collection of System.Char objects that represent a string", so that makes it very different from Rust's `&str` (a string slice) and much more like String, which is defined as:

```
pub struct String {
    vec: Vec<u8>,
}
```

(For F# readers: a `Vec` is sort of like an `Array`)

So perhaps a Rust equivalent of the F# `string` would be:

```
pub struct String {
    vec: Vec<char>,
}
```

However, note that .NET strings [are immutable](https://docs.microsoft.com/en-us/dotnet/api/system.string?view=net-5.0#Immutability):

```
A String object is called immutable (read-only), because its value cannot be modified after it has been created. Methods that appear to modify a String object actually return a new String object that contains the modification.

Because strings are immutable, string manipulation routines that perform repeated additions or deletions to what appears to be a single string can exact a significant performance penalty. For example, the following code uses a random number generator to create a string with 1000 characters in the range 0x0001 to 0x052F. Although the code appears to use string concatenation to append a new character to the existing string named str, it actually creates a new String object for each concatenation operation.
```

So in that case it's very different from a `Vec<char>`, which can simply append a `char` and only needs to reallocate once it surpasses its capacity (which is done automatically).

## Casting to other types

In Rust you use `as` to cast one primitive type into another:

```
fn print_isize(number: isize) { // This will only take an isize
    println!("{}", number);
}

fn main() {
    let x: i8 = 9;
    print_isize(x as isize); // so we cast it into one with as
}
```

In F# you write the name of the type to cast into before the name of the 'variable'. Let's say we have a function that insists on taking a `byte` type:

```
let printByte (number: byte) =
    printfn "%i" number
```

(For Rust users: the `%i` means to print out something that is an integer. If you change it to `%s` (string) for example it will not compile)

Then if we have a number that we simply declare to be 9, F# will make it an `int` (an `i32` in Rust) and you will need to cast it as `byte`.

```
let printByte (number: byte) =
    printfn "%i" number

let number = 9
printByte (byte number)
```

Note the difference between the two: for F# we had to outright specify that the function will only take a `byte` for it not to compile; otherwise it will adapt the type of the function to the input it gets. Rust is stricter and requires generics to take in various types. (F# has generics too - more on the differences between the two in that section)

However, F# is not all loosey-goosey when it comes to function inputs. For example:

```
let printNumber number =
    printfn "%i" number

let number = 9
printNumber number
```

Here, the compiler will know from `%i` and the let binding on `number` that it's taking in an int, and the function signature will be `number:int -> unit`. But if you add the following:

```
let printNumber number =
    printfn "%i" number

let number = 9
printNumber number // Give an int
printNumber (byte number) // Then give a byte
```

This time it will refuse to compile, because it has determined that it needs an `int` and now we've told it to also take a `byte` type as well. The former input has set the variable input type in stone and it duly refuses:

```
This expression was expected to have type
    'byte'    
but here has type
    'int'   
```

On that note, let's compare compiler messages for the same issue. Let's make a function that takes an `int` in F# and `i32` in Rust, but receives a `byte` in F# (a `u8`) in Rust and see what it says.

F#:

```
let printNumber (number: int) =
    printfn "%i" number

let number = 9
printNumber (byte number)
```

Rust:

```
fn print_i32(number: i32) {
    println!("{}", number);
}

fn main() {
    let x: u8 = 9;
    print_i32(x);
}
```

The messages are:

F#

```
This expression was expected to have type
    'byte'    
but here has type
    'int'
```

Rust

```
7 |     print_i32(x);
  |               ^
  |               |
  |               expected `i32`, found `u8`
  |               help: you can convert a `u8` to an `i32`: `x.into()`
```

Notice that the messages are saying the same thing backwards.

Rust: expected `i32`, found `u8` = **The function is expecting an i32, but you gave me a u8**

F: This expression was expected to have type 'byte' but here has type 'int' = **Your input made me expect a byte, but the function is of type int**

(The part mentioning `.into()` in the Rust message is another way to convert between types - more on that later)

# Match

Both Rust and F# use the keyword `match`. Using `match` is pretty similar, but there are some differences. Here's the first one:

If you don't include all cases or use `_` to match all other remaining cases, Rust will fail to compiler. F# will compile, but it will give you a warning.

Rust:

```
enum Mood {
    Good,
    Okay,
    Bad
}

fn check_mood(mood: Mood) {
    match mood {
        Mood::Good => println!("Feeling good"),
        Mood::Bad => println!("Not so good")
    }
}
```

Rust will generate the following:

```
error[E0004]: non-exhaustive patterns: `Okay` not covered
```

In F# you might see something like this:

```
type Mood = | Good | Okay | Bad

let myMood = Good

let matchMood = match myMood with
    | Good -> printfn "Feeling good"
    | Bad -> printfn "Not so good"
```

It compiles, but the moment this is written you see the following below: `Incomplete pattern matches on this expression. For example, the value 'Okay' may indicate a case not covered by the pattern(s).`

Also note that syntax in F# will tend to vary depending on how you want the code to look. The following does the same:

```
type Mood =
    | Good 
    | Okay 
    | Bad

let myMood: Mood = Good

let matchMood = 
    match myMood with
        | Good -> printfn "Feeling good"
        | Bad -> printfn "Not so good"
```

Note that this time we specified the type: `let myMood: Mood = Good`. The F# compiler will also use type inference on the inside of a type to try to determine what it is; Rust will not do that. So if you write a second enum with all the same fields:

```
type Mood = | Good | Okay | Bad

type Mood2 = | Good | Okay | Bad

let myMood = Good

let matchMood = 
    match myMood with
        | Good -> printfn "Feeling good"
        | Bad -> printfn "Not so good"
```

It will make `myMood` of type `Mood2` (it is basically shadowing `Mood`). Watch out! And this happens even if a single field is the same and/or in a different order:

```
type Mood = | Good | Okay | Bad

type Mood2 = | Bad | Good

let myMood = Good

let matchMood = 
    match myMood with
        | Good -> printfn "Feeling good"
        | Bad -> printfn "Not so good"
```

Here as well `myMood` is a `Mood2`. Rustaceans will be in the habit of writing `let myMood: Mood` in any case, but keep in mind that the F# compiler will not do your work for you if you give multiple types the same field names and don't declare their types when making a let binding.

# Currying

Currying doesn't exist in Rust, while F# uses it all the time. Currying means to have a function that takes multiple parameters, but is fine with just taking in one or a few instead of all of them at the same time. For the Rustaceans, it's sort of like this...

```
fn add_three(num_one: i32, num_two: i32, num_three: i32) {
    num_one + num_two + num_three
}
```

except that the return type is not `i32`, it's whatever's left to make the function work. Give it a single `i32` (let's say we type `add_three(8)`) and now it will know the value of `num_one`, and then pass you back this:

```
fn two_left(num_two: i32, num_three: i32) {
    let num_one = 8;
    num_one + num_two + num_three
}
```

(two_left is just a sample name to show what it takes in now)

Now if you type `two_left(9, 10)` it will complete the function with output `i32` and pass back the final number: 27. But if you just give it `two_left(9)`, it will give you:

```
fn one_left(num_three: i32) -> i32 {
    let num_one = 8;
    let num_two = 9;
    num_one + num_two + num_three
}
```

In F# it looks like this:

```
let addThree a b c = a + b + c

let twoLeft = addThree 8
let oneLeft = twoLeft 9
printfn "%i" (oneLeft 10)
```

Note that the spaces between inputs isn't just F# trying to be cool and minimalistic: it's the currying syntax. If you don't want a function to be curried, put the inputs inside of a tuple:

```
let addThree (a, b, c) = a + b + c

printfn "%i" (addThree (8, 9, 10))
```

Fsharpers don't usually like to enclose things in brackets though (especially double brackets) and prefer to use the pipeline operator. This sort of thing might be preferable:

```
let addThree (a, b, c) = a + b + c

let finalNumber = (8, 9, 10) |> addThree

printfn "%i" finalNumber
```

So let's talk about the pipeline operator now. Rust has something similar too.

# Pipelining

Rust doesn't use the word pipeline, nor does it have the `|>` operator. However, sometimes you'll see syntax that reminds you of one in the other. First, let's talk about what the pipeline operator does.

Because (almost) everything is an expression in F# too, pretty much everything you get will return something, even a `()` type. With `|>` you can quickly pass the return on to something else. Here's an example:

```
let addOne x = x + 1
let timesTwo x = x * 2
let printIt x = printfn "%A" x

8 |> addOne |> timesTwo |> printIt
```

So here you see three functions, each of which does something: one adds 1, the next multiplies by two, and the last one prints it.

By the way: `%A` prints the representation of an object. This is most similar to the `Debug` trait in Rust, which uses `{:?}` (for Debug printing) instead of `{}` (Display printing). Using `%A` is a quick way to avoid having to specify `%i` (int), `%s` (string) etc. when printing.

The same in Rust (well, almost the same) would look like this:

```
fn add_one(x: i32) -> i32 {
    x + 1
}

fn times_two(x: i32) -> i32 {
    x * 2
}

fn print_it<T: std::fmt::Debug>(x: T) {
    println!("{:?}", x)
}

fn main() {
    print_it(times_two(add_one(8)));
}
```

Here we're putting them inside successive parentheses to accomplish the same thing. The `T: std::fmt::Debug` is generic and is saying "I guarantee to give you something that can be Debug printed" to the compiler.

If you wanted a more left to right syntax like in F#, you would want to create structs with methods that can do this and it probably wouldn't be worth it. However, you do see this sort of syntax a lot with iterators, which are made to be passed on from left to right. Working with iterators is probably where Rust's syntax gets most 'pipeliney'. For example, let's take ten numbers from 0 to 10, multiply each by two, keep only the even numbers, and then collect it into a `Vec`. It looks like this:

```
fn main() {
    let times_two_then_even: Vec<i32> = (0..=10)
        .map(|number| number * 2)
        .filter(|number| number % 2 == 0)
        .collect();

    println!("{:?}", times_two_then_even);
}
```

Printing that gives us `[0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20]`.

The F# version is very similar:


```
let timesTwoThenEven =
    [0..10]
    |> List.map (fun number -> number * 2)
    |> List.filter (fun number -> number % 2 = 0)

printfn "%A" timesTwoThenEven
```

Here's the output: `[0; 2; 4; 6; 8; 10; 12; 14; 16; 18; 20]`

Notice the difference between the two: both use closures/anonymous functions to perform the work, but the signature is:

Rust: |variable_name_here| variable_name_here * 2

F#: (fun variable_name_here -> variable_name_here * 2)

In Rust, `||` is for closures and `()` for regular functions, while in F# regular functions don't have a particular syntax and closures have `fun` in front of them.

In both cases you are giving a name to the input in order to tell it what to do in the next stage.
