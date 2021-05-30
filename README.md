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

```type Diplomat = { name: string; message: string }```

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

<'a>: in Rust, this is a lifetime specifier: "this object will live for at least the lifetime that we'll call 'a." 

<'a>: in F#, this is a generic specifier.

<T>: in Rust, this is a generic specifier.
