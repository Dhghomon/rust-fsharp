# What's this repository for?

[Rust is my first language](https://github.com/Dhghomon/programming_at_40/blob/master/README.md), and recently I've begun delving into languages in the Rust periphery, one of which is F# (since the Rust compiler was originally written in OCaml and it incorporates a good deal of its syntax).

Almost every F# manual is written for C# developers in mind. That's fine, but I have no real familiarity with C#. And when I look at F# in practice I see Rust everywhere: Options, Results, match statements, iterators, and so on and so forth. An F# guide that's readable for Rustaceans that also serves as a Rust guide for F# users seems to be a necessity. So let's put one together.

I will be adding to this whenever an idea strikes me, but note that I am still very weak at F# and would appreciate PRs for anything I get wrong or haven't explained sufficiently.

That's enough for an intro. Let's start!

# Primitive types in both languages

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
* Rust `unit` in F# is also `unit`. They both use `()` to represent it.
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

