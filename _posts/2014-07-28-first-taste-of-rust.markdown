---
layout: post
title:  "First taste of Rust"
date:   2014-07-28 22:40
categories: rust
---

<i>This post serves as a basic introduction to some core concepts in Rust that I discovered on my path to make a [Wavefront .obj](http://en.wikipedia.org/wiki/Wavefront_.obj_file) parser. It will expand on the differences between languages I have used previously and Rust.</i>

Recently I've been writing a lot of game logic in C# and even though an outstanding language in its own right it has left me yearning for something closer to the metal. I experimented with Go by writing a [small webserver](https://github.com/PudgePacket/GoAppengineTesting) that runs on a [Google Appengine](https://cloud.google.com/products/app-engine/) instance with an Android app frontend but found that while many features of the language resonated with me, it didn't fill any particular needs and the target domain was not something I worked with often.

I'd seen [Rust](http://www.rust-lang.org/) mentioned on [HN](https://news.ycombinator.com) and was curious if it was possible to incorporate elements from Haskell, C++, and Lisp into the same language elegantly. I was pleasantly surprised by the combination of [pure functions](https://en.wikipedia.org/wiki/Pure_function), [algebraic data types](https://en.wikipedia.org/wiki/Algebraic_data_type), and [pattern matching](https://en.wikipedia.org/wiki/Pattern_matching) with [deterministic resource management](https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization), zero cost abstractions, and C style syntax.

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

I enjoy writing functional code when I can but it's not always possible in my line of work. Rusts immutability by default with `let` felt like a sound design decision however the ability to use mutable variables without hassle was the clincher here.

{% highlight rust %}
let     x = 50i; // Immutable
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
    let lines: Vec<String> = file.lines().map(|x| x.unwrap()).collect();
    for line in lines.iter() {
        println!("{}",line);
    }
}
{% endhighlight %}

Coming from C++ this section of code was quite straight forward, set up a path to the file, open the file using a buffered reader and read it line by line into a vector. Perhaps the only unusual looking part is the [lamba syntax](http://doc.rust-lang.org/rust.html#lambda-expressions) inside of map `|x| x.unwrap()` though even that is not difficult to translate into `|arguments| expression`.

Next up I wanted to see the pattern matching in action. I've been working with lots of 3D models lately and I wanted to emulate a little parser. I used the [OBJ](http://en.wikipedia.org/wiki/Wavefront_.obj_file) format because it is text based and quite wide spread. The format is defined that the first character of each line determines the type of what follows, quite a typical idea and one that lends itself well to pattern matching.

{% highlight rust %}
match char_value {
    '#' => println!("Comment"),
    'v' => println!("Vertex"),
    'f' => println!("Face"),
    _   => println!("Unidentified {}", char_value);
}
{% endhighlight %}

The behaviour is very straight forward, match `char_value` on the pattern (Left side) and if the match succeeds evaluate the expression (Right side). Currently it looks like a regular switch statement but I'll get to that soon. Matching on `_` is the catch-all, usually referred to as the default. Unlike C++ the match must cover all possible values of the type it's matching on. This is because match is an <i>expression</i> not a <i>statement</i> and must be able to return something.

Being an expression means we can do things like this

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

Coming from C++ there are a few things that look like they're missing, namely any kind of return statement. This is because Rust returns by having the block produce an expression, and match returns the expression that was matched. `return` still exists but it is used for returning early from loops and the like.

Next we have to combine the two concepts to get our parser going, matching on the lines from the file. We already know the first letter of each line is what we need to parse so what's left is to make a function that takes a line of text and tells us what type of line it is.


{% highlight rust %}
fn parse_line(line: &String) {}
{% endhighlight %}

Since String is a wrapper class around the data we need to get a slice of it, `.as_slice()`. Slices are more or less equivalent to an array of characters. Next we need to get an iterator to grab any of the characters, `.chars()`. Finally we'll call next on the iterator to grab what will be the first character off of the iterator, `.next()`. All up this leaves us with our potential first character.

{% highlight rust %}
let first_letter = line.as_slice().chars().next();
{% endhighlight %}

`first_letter` is now an [Option](http://doc.rust-lang.org/std/option/)\<char\> that might contain a character. Rusts pattern matching lets us check for the existence of the character as well as which character it is at the same time. What we're left with is:

{% highlight rust %}
match first_letter {
    Some('#') => println!("Comment"),
    Some('v') => println!("Vertex"),
    Some('s') => println!("Shading"),
    Some('f') => println!("Face"),
    Some(x)   => println!("Unknown {}" , x),
    None      => ()
}
{% endhighlight %}

We check for the existence of a letter from a subset of the known OBJ types and let stdout know we've found something. For the catch-all we return `()` which is essentially the equivalent of null. The catch for unknown characters `Some(x)` uses destructuring, bringing x into the scope of it's expression so that we can print the unknown value.

And that's pretty much it!

[A github link to the code is here](https://github.com/PudgePacket/Rusticle/tree/f851941d3853d08391fa6193af7e8db540367f71)

{% highlight rust %}
use std::io::File;
use std::io::BufferedReader;

fn parse_line(line: &String) {
	let first_letter = line.as_slice().chars().next();
	match first_letter {
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
	let lines: Vec<String> = file.lines().map(|x|x.unwrap()).collect();
	for line in lines.iter() {
		parse_line(line);
	}
}
{% endhighlight %}