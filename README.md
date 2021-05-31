# Rust for Fsharpers and F# for Rustaceans

<img src="RF.png"
     width="50%" height="50%"
     style="float: left; margin-right: 10px;" />

# What's this repository for?

[Rust is my first language](https://github.com/Dhghomon/programming_at_40/blob/master/README.md), and recently I've begun delving into languages in the Rust periphery, one of which is F# (since the Rust compiler was originally written in OCaml and it incorporates a good deal of its syntax).

Almost every F# manual is written for C# developers in mind. That's fine, but I have no real familiarity with C#. And when I look at F# in practice I see Rust everywhere: Options, Results, match statements, iterators, and so on and so forth. Reading a book on F# with references to C# everywhere for a Rustacean is kind of like a German being forced to learn Dutch through Portuguese. And for an Fsharper, reading a beginner's book on Rust that assumes the user doesn't know about Option, match statements and the rest is too much filler to be enjoyable to read. 

With so much in common, an F# guide that's readable for Rustaceans that also serves as a Rust guide for F# users seems to be a necessity. So let's put one together.

I will be adding to this whenever an idea strikes me, but note that I am still very weak at F# and would appreciate comments or PRs for anything I get wrong or haven't explained sufficiently.

That's enough for an intro. Let's start!

# Trying out the language without installing

Both languages have nice online environments to practice without having to install anything. In Rust you use the [Rust Playground](https://play.rust-lang.org/), while F# uses a site called [Try F#](https://try.fsharp.org/).

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
* Rust `usize` in F# is `unativeint` (.NET: `UIntPtr`)
* Rust `isize` in F# is `nativeint` (.NET: `IntPtr`)
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

Interestingly, F# also has a `function` keyword that is basically short for `match (name) with`.

```
type Options = 
    | Sunny
    | Rainy
    | Other

let message = function
    | Sunny -> printfn "It's sunny today"
    | Rainy -> printfn "It's rainy"
    | Other -> printfn "Not sure what the weather is."

message Sunny
```

This will print out "It's sunny today".

Over on the [F# documentation](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/match-expressions) you can see an example of both:

```
// Pattern matching with multiple alternatives on the same line.
let filter123 x =
    match x with
    | 1 | 2 | 3 -> printfn "Found 1, 2, or 3!"
    | a -> printfn "%d" a

// The same function written with the pattern matching
// function syntax.
let filterNumbers =
    function | 1 | 2 | 3 -> printfn "Found 1, 2, or 3!"
             | a -> printfn "%d" a
```



# Currying

Currying doesn't exist in Rust, while F# uses it all the time. Currying means to have a function that takes multiple parameters, but is fine with just taking in one or a few instead of all of them at the same time. For Rustaceans, it would be sort of like this...

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

Interesting fact: all functions in F# are curried, which is why they have this interesting signature:

```
val addThree : a:int -> b:int -> c:int -> int
```

When you pass in three numbers to the function, it will curry the function with one input parameter at a time, passing it on until all three have a value and then finally return an `int`. (I may have explained this oddly or wrongly - let me know if so)

In Rust, the function would look like this:

```
fn add_three(a: i32, b: i32, c: i32) -> i32
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

By the way: `%A` prints tuples, record and union types. This is most similar to the `Debug` trait in Rust, which uses `{:?}` (for Debug printing) instead of `{}` (Display printing). Using `%A` is a quick way to avoid having to specify `%i` (int), `%s` (string) etc. when printing. Another specifier is %O to print other objects (this automatically uses ToString() to do it).

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

There is a big difference between Rust and F# here: Rust has something known as "zero-cost abstractions", which essentially means that there is no performance impact for fancy code compared to simple `for` loops and such things. Let's look at `times_two_then_even` in Rust again:

```
    let times_two_then_even: Vec<i32> = (0..=10)
        .map(|number| number * 2)
        .filter(|number| number % 2 == 0)
        .collect();
```

This gives a `Vec<i32>` because of the `.collect()` method at the end. Let's see what happens when we take it out and don't assign it to a variable:

```
fn main() {
    (0..=10)
        .map(|number| number * 2)
        .filter(|number| number % 2 == 0);
}
```

It compiles, but tells us that we actually haven't done anything:

```
warning: unused `Filter` that must be used
 --> src/main.rs:2:5
  |
2 | /     (0..=10)
3 | |         .map(|number| number * 2)
4 | |         .filter(|number| number % 2 == 0);
  | |__________________________________________^
  |
  = note: `#[warn(unused_must_use)]` on by default
  = note: iterators are lazy and do nothing unless consumed
```

This is because calling these iterator methods without collecting them or assigning them to a variable just makes a big complex type; we haven't mapped or filtered anything yet. Let's see what it looks like by getting the compiler mad:

```
fn main() {
    let times_two_then_even: i32 = (0..=10) // Tell the compiler it's an i32
        .map(|number| number * 2)
        .filter(|number| number % 2 == 0);
}
```

The compiler complains:

```
expected type `i32`
           found struct `Filter<Map<RangeInclusive<{integer}>, [closure@src/main.rs:3:14: 3:33]>, [closure@src/main.rs:4:17: 4:41]>`
```

So all we've done is put together a type Filter<Map<RangeInclusive etc. etc. etc. And if we call .map a whole bunch of times it just keeps on putting this big struct together:

```
found struct `Map<Map<Map<Map<Map<Map<Map<Map<RangeInclusive<{integer}>, [closure@src/main.rs:3:14: 3:33]>, [closure@src/main.rs:4:14: 4:33]>, [closure@src/main.rs:5:14: 5:33]>, [closure@src/main.rs:6:14: 6:33]>, [closure@src/main.rs:7:14: 7:33]>, [closure@src/main.rs:8:14: 8:33]>, [closure@src/main.rs:9:14: 9:33]>, [closure@src/main.rs:10:14: 10:33]>`
```

This struct is all ready to go, and gets run *once* when we actually decide to do something with it (like collect it into a `Vec`). The pipeline operator in F#, however, gets run every time you use it: if you do something like this:

```
let addOne x = x + 1;

let number  = 
    addOne 9
    |> addOne
    |> addOne
    |> addOne
    |> addOne
    |> addOne
    |> addOne
```

It's going to call `addOne` every time, and same for F#'s iterator methods. Apparently too much usage of the pipeline operator can slow down performance, though F# is performant enough. (Somewhat less that C#, miles faster than something like Python) Rust is naturally right up there with C and C++ in terms of performance.

F# also has something called the "forward composition operator" which looks like `>>`, which is a fancy way of saying that it smushes functions together into one. It's basically the same as a pipeline except all the operations are turned into a single function. Here's a simple example:

```
let addOne x = x + 1
let timesTwo x = x * 2
let printOut x = printfn "%i" x

let addMultiplyPrint = addOne >> timesTwo >> printOut

addMultiplyPrint 9
```

That prints out `20`. You can imagine it being useful from time to time, though the pipeline operator alone is clear enough and makes it easy to make small changes (like if you wanted to change `let addOne x = x + 1` to `let addNum x y = x + y`).

F# also has a right to left pipeline operator: `<|`. Users of F# caution against using it too much, and say it should only be used sparingly when it makes code readable. Using both results in some pretty wacky syntax:

```
let printTwo x y = printfn "%i and %i" x y

8 |> printTwo <| 9
```

The compiler is perfectly happy with this, printing out `8 and 9`. But there's no reason to write it this way over the simpler `printTwo 8 9`.

Although even that syntax could be interesting once in a while. Here's one example:

```
type Diplomat = {
    name: string
    message: string
}

let diplomat1 = {name = "Quintus Aurelius"; message = "We demand concessions!"}
let diplomat2 = {name = "Argentyx"; message = "We would rather die!"}

let meeting person1 person2 =
    printfn "%s says: %s" person1.name person1.message
    printfn "%s responds: %s" person2.name person2.message

diplomat1 |> meeting <| diplomat2
```

Kind of looks nice to see our two diplomats approaching each other in the middle with `diplomat1 |> meeting <| diplomat2` rather than a boring old `meeting diplomat1 diplomat2`. The output is of course:

```
Quintus Aurelius says: We demand concessions!
Argentyx responds: We would rather die!
```

On that note, what is that `type Diplomat` doing there? Let's look at that now.

# Rust structs, F# record types and structs

The main custom data type in Rust is a struct, which looks very similar to the type we just declared above. Let's look at a Rust `Diplomat` struct and the F# `Diplomat` record again:

```
struct Diplomat {
    name: String,
    message: String
}
```

```
type Diplomat = { 
    name: string
    message: string 
    }
```

Note that the fields in Rust are separated with commas, and a new line in F#. In F# you can also use semicolons if you want the fields on the same line:

```
type Diplomat = { name: string; message: string }
```

Rust users will also have noticed that we didn't even say we were creating a `Diplomat` when we instantiated them. All we wrote was this:

```
let diplomat1 = {name = "Quintus Aurelius"; message = "We demand concessions!"}
```

This is once again an example of the F# compiler looking at the fields of a type to infer the type. The inference doesn't go *that* deep though. For example, if we make a separate Diplomat2 type:

```
type Diplomat = { name: string; message: string }
type Diplomat2 = { name: string; message: string }
```

Then the `meeting` function will assume that it's taking a `Diplomat2` because it shadows the previous one:

```
let meeting person1 person2 =
    printfn "%s says: %s" person1.name person1.message
    printfn "%s responds: %s" person2.name person2.message
```

But even declaring this type will cause it to err!

```
type Diplomat = { name: string; message: string }
type MessagelessDiplomat = { name: string }
```

Here too the compiler assumes that the function will take a `MessagelessDiplomat`, and because it doesn't have a `message` field it will give an error. A Rustacean in any case will certainly feel more comfortable declaring function with this sort of syntax:

```
let meeting (person1: Diplomat) (person2: Diplomat) =
    printfn "%s says: %s" person1.name person1.message
    printfn "%s responds: %s" person2.name person2.message
```

You can also declare anonymous record types in F# by adding `||`:

```
let diplomat1 = {|
    name = "Marcus Aurelius"
    message = "The Emperor demands tribute."
    |}

printfn "%s says: %s" diplomat1.name diplomat1.message
```

If you want to add methods to a record, you use `with` and then the `member`keyword, followed by `this` and the method you want to write. This is fairly different from Rust, but once the boilerplate is done it is quite similar.

```
type Diplomat = {
    name: string
    message: string
}
with
    member this.Talk() = 
        printfn "%s says: %s" this.name this.message
```

Now our diplomatic summit turns into this:

```
type Diplomat = {
    name: string
    message: string
}
with
    member this.Talk() = 
        printfn "%s says: %s" this.name this.message


let meeting (person1: Diplomat) (person2: Diplomat) =
    person1.Talk()
    person2.Talk()

let diplomat1 = {name = "Quintus Aurelius"; message = "We demand concessions!"}
let diplomat2 = {name = "Argentyx"; message = "We would rather die!"}

diplomat1 |> meeting <| diplomat2
```


Now let's look at Rust structs in comparison. Let's look at the Diplomat struct again:

```
struct Diplomat {
    name: String,
    message: String
}
```

To declare one, you need to specify that you are creating a `Diplomat` - the Rust compiler doesn't root through the fields to try to determine the type for you. Also don't forget the semicolon. If you do this it won't work:

```
struct Diplomat {
    name: String,
    message: String
}

fn main() {
    let diplomat1 = Diplomat{
    name: "Quintus Aurelius".to_string(), message: "We demand concessions!".to_string()
        
    }
}
```

Fortunately, the Rust compiler generally knows what you are trying to do. Here's the message:

```
error: expected `;`, found `}`
  --> src/main.rs:10:6
   |
10 |     }
   |      ^ help: add `;` here
11 | }
   | - unexpected token
```

That was nice of it. Also note that the formatting above is a bit lazy. Let's type `cargo fmt` (or hit the Rustfmt button on the Playground) to make it nice.

```
struct Diplomat {
    name: String,
    message: String,
}

fn main() {
    let diplomat1 = Diplomat {
        name: "Quintus Aurelius".to_string(),
        message: "We demand concessions!".to_string(),
    };
}
```

Much better! One note here: the semicolon is necessary because Rust is an expression-based language too. Rust also treats the final line of an expression as the return value, but we are not looking to pass on a `Diplomat` to something else so we need a semicolon at the end to make it return a unit type instead.

The F# user is now going to be wondering what this `.to_string()` method is doing and why we need it in Rust. This is because Rust has more than one `String` type, and this is the simplest one to understand in a struct: it's similar to the F# string type mentioned above in that it is an owned collection, though in this case it's a collection of `u8` bytes. (We saw the signature above already) Without the `.to_string()` method, we are instead dealing with a "&str", which is a borrowed reference - the type does not own it. It is essentially an immutable view into a string. And because it doesn't own it, the compiler will complain at this sort of signature:

```
struct Diplomat {
    name: &str,
    message: &str,
}

fn main() {
    let diplomat1 = Diplomat {
        name: "Quintus Aurelius",
        message: "We demand concessions!",
    };
}
```

Because here `Diplomat` doesn't own the data, there's a possibility that you might pass in data that starts out owned by another object, which later on is dropped and deallocated, and now you have a reference to data that doesn't exist. The compiler will tell you to give it a hint about how long the data will live:

```
error[E0106]: missing lifetime specifier
 --> src/main.rs:2:11
  |
2 |     name: &str,
  |           ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 | struct Diplomat<'a> {
2 |     name: &'a str,
  |
```
 
As it turns out, the hint it gives you is exactly what you need to get the code to compile. It says "there is a lifetime that we'll call <'a> that the Diplomat struct lives for, and I'll only pass in a &str that lives for at least that long."

One note here for users of both languages: both languages use <> angle brackets to specify generics, but slightly differently:

`<'a>`: in Rust, this is a lifetime specifier: "this object will live for at least the lifetime that we'll call 'a." 

`<'a>`: in F#, this is a generic specifier.

`<T>`: in Rust, this is a generic specifier.

In short, stick with `String` in the beginning with Rust until you get used to lifetimes. A `&str` is more performant than a `String`, but is less flexible.

So back to the struct. How does Rust add methods? It uses what is known as an `impl` block. These are separate from the struct declaration, and you can use as many of them as you like. Inside the `impl` block you add your methods. You have four choices with methods:

- Methods that take `Self` (the equivalent of F# `this`): these take ownership of the struct's data! The struct will then only live until the end of the function. These methods are usually used with the builder pattern, by bringing in the object, making one change and then returning `Self` back to the struct. This is a bit F-sharpy since it allows you to chain one method to another. Let's take a look at this in detail. Here's a regular struct called `City`:

```
struct City {
    name: String,
    population: i32
}
```

Now we'll give it some methods. Start an `impl` block: 

```
impl City {

}
```

Then add some methods. First a new method. Note: `new` isn't a keyword in Rust - we could call this `neu` or `nouveau` or anything else.

```
fn new() -> Self { // Can also write City
    Self {
        name: "".to_string(),
        population: 0
    }
}
```

Because this method doesn't take `self` (or `&self` or `&mut self`), it's called an associated method and is called like this: `City::new()`. That's because there is no instance of `City` to use the dot operator on.

So that will make a new city with no name and a population of 0. Now let's add two methods that take self and return it after making a change:

```
fn population(mut self, population: i32) -> Self {
    self.population = population;
    self
}

fn name(mut self, name: &str) -> Self {
    self.name = name.to_string();
    self
}
```

(Note: no problem taking a `&str` here because we don't care about using it after the function is over. Nobody cares about the lifetime in this case, and it is turned into a `String` which is owned by the struct)

And with all that, we can now set up a city using this F#-like pattern:

```
fn main() {
    let city = City::new().population(100).name("My city");
}
```

Usually a `new` method will look more like this though:

```
fn new(population: i32, name: &str) -> Self {
    Self {
        population,
        name: name.to_string()
    }
}
```

Why didn't we write `population: population`? We could have, but if the two names are the same, we can just write `population`.

Now that the Fsharpiest part is out of the way, here is what is more commonly passed into methods: `&self` and `&mut self`. `&self` is a reference to self (you can read but not modify it) while `&mut self` is a mutable reference to self (you can modify it). As you can imagine, there are rules regarding references to avoid unexpected behaviour.

1) You can have as many references as you like if they are not mutable references.
2) You can have up to one mutable reference.
3) If you have a mutable reference, everything else is essentially frozen: nobody else can have a mutable or immutable reference as long as the mutable reference is still around.

Here is a quick example of code that doesn't compile:

```
fn main() {
    let mut my_string = "I am a string".to_string();
    let x = &my_string;
    let y = &mut my_string;
    
    println!("{}", x);
}
```

Once you declare x as a reference to `my_string`, it is expecting a `String` that says "I am a string". However, in the next line comes `y` which is able to modify the data that `x` has a reference to, and is going to try to print out two lines later. This is not okay, and the compiler tells us as much:

```
error[E0502]: cannot borrow `my_string` as mutable because it is also borrowed as immutable
 --> src/main.rs:4:13
  |
3 |     let x = &my_string;
  |             ---------- immutable borrow occurs here
4 |     let y = &mut my_string;
  |             ^^^^^^^^^^^^^^ mutable borrow occurs here
5 |     
6 |     println!("{}", x);
  |                    - immutable borrow later used here
```

Note the part that says "immutable borrow later used here", because the compiler is smart enough to track not just the creation of references, but whether they are being used or not. If we were to comment out line 6 it would compile, because even though we created references that both live until the end of the scope, the compiler can see that they never get used and it will not make an issue out of it. (Some years back the compiler wasn't capable of this and would always generate errors on the *existence* of such references instead of being able to track their actual use)

# Mutability

Both Rust and F# use let bindings for variables that are immutable by default. To make something mutable in Rust, add the `mut` keyword. F# requires a longer keyword: `mutable`. This fits with the F# philosophy of trying to avoid mutability as much as possible. In Rust immutability is just a common sense default - why make something mutable unless you need to change it?

Changing a mutable variable in Rust involves just `=` and the new value. Note that we're not using the `let` keyword here! Using `let` again would create a new variable that shadows the previous one. Using `=` naturally does not let you change the type of whatever it is we are modifying.

```
let mut x = 7;
x = 8;
```

In F#, changing the value of a mutable variable is done with the `<-` operator.

```
let mutable x = 7
x <- 8
```

# Order of code

In Rust, the order in which you write your code matters inside a single scope, while code outside of this does not. So a typical simple program will have something like this:

```
struct SomeStruct {
    field: i32
}

enum SomeChoices {
    Good,
    Bad
}

fn do_thing() {
    // Do a thing
}

fn main() {
    // Do stuff
}
```

But you can just as easily put the struct, enum and function below `main` and it would have no effect. The one exception to this is macro definitions. (Macros are function-like things that expand into source code before the compiler starts its work and are written with a `!` - more on them later) So this will work:

```
macro_rules! my_macro {
    () => {
        println!("You didn't give me anything");
    };
    ($a:expr) => {
    {
        println!("Here's your expression back");
        $a
    }
    };
}

fn main() {
    let x = my_macro!();
    let z = my_macro!(9);
}
```

But it won't recognize this `my_macro()!` macro if you move it down below main.

As for F#, order always matters. So if you have a `Customer` record like the one following:

```
type Customer = {
    name: string
    accountBalance: float
}

let billy = {
    name = "Billy Brown"
    accountBalance = 100.00
}
```

It will generate an error if you move it down below our `let billy` declaration. And not only this: the order of files in an F# program matters too! If you have multiple files, be sure to order them accordingly. This is in accordance with the F# "data in data out" sort of principle that likes to see everything done in order.

Besides named structs, Rust also has tuple structs. They are generally used for two things:

- Simple structs that don't need named fields.

```
struct RGB(u8, u8, u8);

fn main() {
    let my_colour = RGB(80, 65, 0);
}
```

- New types. While type aliases can be declared with the `type` keyword:

```
type VecOfInts = Vec<i32>;

fn main() {
    let some_vec: VecOfInts = vec![8, 9, 10];
}
```
     
this type is 100% equivalent to `Vec<i32>` and the compiler reads it as such. But a struct containing another type works as a new type, and while it does have access to the type's methods by calling on it (using `.0`), it is effectively a new type and that lets you implement other traits on it.
     
So this will no longer work:

```
struct VecOfInts(Vec<i32>);

fn main() {
    let some_vec: VecOfInts = vec![8, 9, 10];
}
```
     
But this will:
     
```
struct VecOfInts(Vec<i32>);

fn main() {
    let some_vec = VecOfInts(vec![8, 9, 10]);
}
```

And since `VecOfInts` is its own type, this won't work yet:
     
```
println!("{:?}", some_vec);
```
     
But this will:

```
println!("{:?}", some_vec.0);
```
     
The ears of Fsharpers will have perked up at this already, because F# uses these sorts of new types **a lot** and loves declaring them on the fly for type safety and readability. Here's an example:

```
type Temperature = 
    | Celsius of float
    | Fahrenheit of float
    | Kelvin of float
    | Réaumur of float

let getTemperature temperature = 
    match temperature with
        | Celsius number -> printfn "%.1f Celsius" number
        | Fahrenheit number -> printfn "%.1f Fahrenheit" number
        | Kelvin number -> printfn "%.1f Kelvin" number
        | Réaumur number -> printfn "%.1f Réaumur" number

getTemperature (Réaumur 9.0)

let temperature = Réaumur 9.0
```

You can see that `Temperature` does work like an enum, because you can match against it. The variants of `Temperature` also can contain values, though they are somewhat different than in Rust. Here is how you would do more or less the same thing:

```
enum Temperature {
    Celsius(f64),
    Fahrenheit(f64),
    Kelvin(f64),
    Reaumur(f64)
}

impl Temperature {
    fn get_temperature(&self) {
    use Temperature::*;
        match self {
            Celsius(number) => println!("{} Celsius", number),
            Fahrenheit(number) => println!("{} Fahrenheit", number),
            Kelvin(number) => println!("{} Kelvin", number),
            Reaumur(number) => println!("{} Réaumur", number),
        }
    }
}

fn main() {
    let temperature = Temperature::Reaumur(9.0);
    temperature.get_temperature();
}
```
     
So let's compare the two a little. First the Temperature type/enum:

```
type Temperature = 
    | Celsius of float
    | Fahrenheit of float
    | Kelvin of float
    | Réaumur of float
     
enum Temperature {
    Celsius(f64),
    Fahrenheit(f64),
    Kelvin(f64),
    Reaumur(f64)
}
```
     
The Rust version creates an enum called `Temperature` with four arms that all contain `f64`s. The F# version does something similar. However, in F# these are new types (like Rust tuple structs) organized together under a type called `Temperature`, whereas in Rust they are dependent arms of the enum. You can't write `let x = Reaumur(9.0);` in Rust, only `let x = Temperature::Reaumur(9.0);`.
     
The match statement is pretty much identical, though in F# it's an independent function compared with Rust which has it as a method. In both cases you could do the opposite (writing a method in F# or an independent function in Rust) but methods are generally preferred in Rust (because they are [convenient when it comes to working with references](https://doc.rust-lang.org/nomicon/dot-operator.html) for one) while I get the impression that F# likes freely roaming functions instead of methods attached to types.

```
let getTemperature temperature = 
    match temperature with
        | Celsius number -> printfn "%.1f Celsius" number
        | Fahrenheit number -> printfn "%.1f Fahrenheit" number
        | Kelvin number -> printfn "%.1f Kelvin" number
        | Réaumur number -> printfn "%.1f Réaumur" number
     
impl Temperature {
    fn get_temperature(&self) {
    use Temperature::*;
        match self {
            Celsius(number) => println!("{} Celsius", number),
            Fahrenheit(number) => println!("{} Fahrenheit", number),
            Kelvin(number) => println!("{} Kelvin", number),
            Reaumur(number) => println!("{} Réaumur", number),
        }
    }
}
```
     
Note that: 

- `get_temperature` is taking a reference to `self` because we only want to read the data, not destroy it. 
- We have `use Temperature::*;` to import all the arms of the enum (same as F# `open`) to avoid typing `Temperature::Celsius(number), Temperature::Fahreinheit(number)` etc. You don't have to do this but it saves keystrokes.
- The identifier Réaumur here is written Reaumur. Rust is working on [non-ASCII identifiers](https://doc.rust-lang.org/stable/unstable-book/language-features/non-ascii-idents.html) and this is already available as an unstable feature. It'll probably end up as a stable feature sooner rather than later.
     
# Collection types

Here is a quick overview of the collection types in both languages:

**MAIN TYPES**

Rust: Vec, array, tuples

F#: list, array, sequence, tuples


