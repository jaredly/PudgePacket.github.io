---
layout: post
title:  "First taste of Rust"
date:   2014-07-28 22:40
categories: rust
---

<i>This post serves as a basic introduction to Rust by creating the first steps of a [Wavefront .obj](http://en.wikipedia.org/wiki/Wavefront_.obj_file) parser. More will be added to the parser in subsequent posts. It will also be a comparison between Rust and current major langauges.</i>

Recently I've been writing a lot of game logic in C# and even though an outstanding language in its own right it has left me yearning for something closer to the metal. I experimented with Go by writing a [small webserver](https://github.com/PudgePacket/GoAppengineTesting) that runs on a [Google Appengine](https://cloud.google.com/products/app-engine/) instance with an Android app frontend but found that while many features of the language resonated with me, it didn't fill any particular needs and the target domain was not something I worked with often.

I'd seen [Rust](http://www.rust-lang.org/) mentioned on [HN](https://news.ycombinator.com) and was curious if it was possible to incorporate elements from Haskell, C++, and Lisp into the same language elegantly. I was pleasantly surprised by the combination of [algebraic data types](https://en.wikipedia.org/wiki/Algebraic_data_type), and [pattern matching](https://en.wikipedia.org/wiki/Pattern_matching) with [deterministic resource management](https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization), zero cost abstractions, and 'C style' syntax.

I decided to give it a shot. After a few small hiccups getting it running on windows and finding the [Sublime](https://www.sublimetext.com/) plugin I dove straight in.

The function syntax impressed me with a distinct lack of redundant keywords as well as being able to return multiple values via tuple.

{% highlight rust %}
// Doubled values of a and b
fn doubled(a: int, b: int) -> (int, int) {
    (a + a, b + b)
}

fn main() {
    let (a,b) = doubled(2,3);
    // a == 4, b == 6
}
{% endhighlight %}

I enjoy writing functional code when I can but it's not always possible in my line of work. Rusts 'immutability by default' with `let` felt like a sound design decision however the ability to use mutable variables without hassle was the clincher here.

{% highlight rust %}
let x = 50i; // Immutable
let mut y = 50i; // Mutable

x = 100i; // Error
y = 100i; // Okay
{% endhighlight %}

The first task I attempted was printing the contents of a file. A relatively simple task but also one that would show the import process as well as how collections and iteration would work. A quick serch found me the official [IO module documentation](http://doc.rust-lang.org/std/io/) which had good examples for most tasks.

{% highlight rust %}
use std::io::File;
use std::io::BufferedReader;

fn main() {
    let path = Path::new("message.txt");
    let mut file = BufferedReader::new(File::open(&path));
    for line in file.lines().map(|x| x.unwrap()) {
        parse_line(line.as_slice());
    }
}
{% endhighlight %}

Coming from C++ this section of code was quite straight forward, set up a path to the file, open the file using a buffered reader and read it line by line into a vector. Perhaps the only unusual looking part is the [lamba syntax](http://doc.rust-lang.org/rust.html#lambda-expressions) inside of map `|x| x.unwrap()` though even that is not difficult to translate into `|arguments| expression`. Also, since String is not something we can modify directly we have to get it in a workable type in the form of &str using `.as_slice()`.

Next up I wanted to see the pattern matching in action. I've been working with lots of 3D models lately and I wanted to emulate a little parser. I used the [OBJ](http://en.wikipedia.org/wiki/Wavefront_.obj_file) format because it is text based and quite wide spread. The format is defined that the first character of each line determines the type of what follows, quite a typical idea and one that lends itself well to pattern matching.

{% highlight rust %}
match first_character {
    '#' => println!("Comment"),
    'v' => println!("Vertex"),
    'f' => println!("Face"),
    _   => println!("Unidentified {}", first_character);
}
{% endhighlight %}

The behaviour is very straight forward, match `first_character` on the pattern (Left side) and if the match succeeds evaluate the expression (Right side). Pattern matching in Rust must be exhaustive. This can be achieved by creating patterns to satisfy every possibility. This process is made easier with a wildcard like `_` to catch all value, similar to default in other langauges.

Match being an expression rather than a statement means we can do things like this

{% highlight rust %}
fn phonetic_table_expander(letter: char) -> &'static str {
    match letter {
        'a' | 'A' => "Alpha", 
        'b' | 'B' => "Bravo",
        'c' | 'C' => "Charlie",
        ...
        _         => "Unknown"
    }
}
{% endhighlight %}

For a 'C style' language there are a few things that look like they're missing, namely any kind of return statement. This is because Rusts return is implicit. Match works the same way, returning the matching expression. `return` still exists as a keyword but it is used for returning early from loops and the like.

Next we have to combine the two concepts to get our parser going, matching on the lines from the file. We already know the first letter of each line is what we need to parse so what's left is to make a function that takes a line of text and tells us what type of line it is.


{% highlight rust %}
fn parse_line(line: &str) {}
{% endhighlight %}

We need to get an iterator to grab any of the characters, an iterator is made by calling `.chars()`. Finally we'll call next on the iterator to grab what will be the first character off of the iterator, `.next()`. All up this leaves us with our potential first character.

{% highlight rust %}
let first_character = line.chars().next();
{% endhighlight %}

`first_character` is now an [Option](http://doc.rust-lang.org/std/option/)\<char\> that might contain a character. From a C++ perspective we can think of Option as an enum that holds different types instead of different values. Option can be Some\<char\> or None. In this instance we're pattern matching against type as well as value, the type of Some\<char\> or None as well as the individual char value. What we're left with is:

{% highlight rust %}
match first_character {
    Some('#') => println!("Comment"),
    Some('v') => println!("Vertex"),
    Some('s') => println!("Shading"),
    Some('f') => println!("Face"),
    Some(x)   => println!("Unknown {}" , x),
    None      => ()
}
{% endhighlight %}

We check for the existence of a letter from a subset of the known OBJ types and let stdout know we've found something. For the catch-all we return `()` which is essentially the equivalent of null. The catch for unknown characters `Some(x)` uses destructuring, bringing x into the scope of it's expression so that we can print the unknown value.

I'll be continuing my exploration of rust by getting my hands dirty with this parser so check back soon for the next part.

And that's pretty much it!

[A github link to the code is here, feel free to fork if you like.](https://github.com/PudgePacket/Rusticle/tree/5081a02ca41f75da99daa25ae0927b55cc13605f)

Full source code:

{% highlight rust %}
use std::io::File;
use std::io::BufferedReader;

fn parse_line(line: &str) {
    let first_character = line.chars().next();
    match first_character {
        Some('#') => println!("Comment"),
        Some('v') => println!("Vertex"),
        Some('s') => println!("Shading"),
        Some('f') => println!("Face"),
        Some(x)   => println!("Unknown {}" , x),
        None      => ()
    }
}

fn main() {
    let path = Path::new("cube.obj");
    let mut file = BufferedReader::new(File::open(&path));
    for line in file.lines().map(|x| x.unwrap()) {
        parse_line(line.as_slice());
    }
}
{% endhighlight %}
