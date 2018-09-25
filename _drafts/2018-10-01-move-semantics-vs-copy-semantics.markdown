---
layout: post
comments: true
title: "Move Semantics vs Copy Semantics"
date: 2018-10-01 13:47:00 -0430
categories: language ownership type-system
---

This is a post I wanted to write because I feel the moment I clearly understand Move Semantics I was able to
use Rust in a basic level. Obviously Rust has a steep learning curve, specially for people like me, who comes from Ruby,
so I thought this could help Rust beginners like myself to learn how to use the language.

## **Copy**

In a few situations you want copy semantics, which means the value saved in a
variable will be copied from one point of memory to another.
For example in C you can do this:

{% highlight c %}
#include <stdio.h>

int main() {
    struct Books {
        char  title[50];
        char  author[50];
        char  subject[100];
        int   book_id;
    } book1;

    strcpy( book1.title, "C Programming");

    struct Books book2 = book1;

    strcpy( book2.title, "Copy Semantics");

    printf( "book 1 title is %s\n", book1.title );
    printf( "book 2 title is %s\n", book2.title );
}
{% endhighlight %}

{% highlight bash %}
$ gcc test.c -o test
$ ./test
book 1 title is C Programming
book 2 title is Copy Semantics
{% endhighlight %}

As you can see, the equal operator in C/C++ performs a copy of the struct from book1 to book2, so, you can freely
modify book2 without altering the values in book1, now if you want to do the same thing in Rust:

{% highlight rust %}
struct Books {
    title: String,
    author: String,
    subject: String,
    book_id: i32
}

fn main() {
    let book1 = Books { 
        title: "Rust Programming".to_string(),
        author: "Jhon Doe".to_string(),
        subject: "Programming".to_string(),
        book_id: 12
    };
    let book2 = book1;
    
    println!("{}", book1.title);
}
{% endhighlight %}

{% highlight bash %}
error[E0382]: use of moved value: `book1.title`
  --> src/main.rs:18:20
   |
16 |     let book2 = book1;
   |         ----- value moved here
17 |     
18 |     println!("{}", book1.title);
   |                    ^^^^^^^^^^^ value used here after move
   |
{% endhighlight %}

So, what happened?, the pointer address book1 was copied into book2, making book1 invalid,
transfering the ownership to book2. Other languages like Ruby does a similar thing, however book1 and book2
remains valid (without the owning part), but if you think about it, it is confusing, if you modify book2, book1
is modified as well, but that's not the intention (at least most of the time),
because you want to copy the value, not being referenced by two variables,
at least Rust has a sane way to deal with this.

Now returning to the Rust's example, what can we do in this case?, depending on the situation,
we can borrow the value, taking a [reference](https://en.wikipedia.org/wiki/Reference_(computer_science)) of book1,
like this: `let book2 = &book1;` or we can implement the Clone trait like this:

{% highlight rust %}
#[derive(Clone)]
struct Books {
    title: String,
    author: String,
    subject: String,
    book_id: i32
}

fn main() {
    let book1 = Books { 
        title: "Rust Programming".to_string(),
        author: "Jhon Doe".to_string(),
        subject: "Programming".to_string(),
        book_id: 12
    };
    
    let book2 = book1.clone();
    
    println!("{}", book1.title);
}
{% endhighlight %}

## **Move**

In another case, you want move semantics, like this one:

{% highlight rust %}
struct Books {
    title: String,
    author: String,
    subject: String,
    book_id: i32
}

fn main() {
    let book1 = Books { 
        title: "Rust Programming".to_string(),
        author: "Jhon Doe".to_string(),
        subject: "Programming".to_string(),
        book_id: 12
    };

    let book2 = Books { 
        title: "C Programming".to_string(),
        author: "Pete Jhones".to_string(),
        subject: "Programming".to_string(),
        book_id: 13
    };

    let books = vec![book1, book2];
    let mut new_books: Vec<Books> = vec![];
    for book in books.iter() {
        let mut book_1 = book;
        //do something with book_1
        new_books.push(book_1);
    }
} 
{% endhighlight %}

{% highlight bash %}
error[E0308]: mismatched types
  --> src/main.rs:28:24
   |
28 |         new_books.push(book_1);
   |                        ^^^^^^ expected struct `Books`, found &Books
   |
   = note: expected type `Books`
              found type `&Books`
{% endhighlight %}

This error means that new_books expect a struct, not a reference,
but why is there a reference?, where does it come from?,
if you take a look at 
[iter](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter) method,
you can see:

![Iter Method]({{ site.url }}/assets/iter_method.png)

The iter method takes a reference of self, in this case books, making every book item
a reference as well. What can we do? we have one option,
we can [dereference](https://en.wikipedia.org/wiki/Dereference_operator) book_1
changing line 28 to `new_books.push(*book_1);`, but now we have:

{% highlight bash %}
error[E0507]: cannot move out of borrowed content
  --> src/main.rs:28:24
   |
28 |         new_books.push(*book_1);
   |                        ^^^^^^^ cannot move out of borrowed content
{% endhighlight %}

The variable book_1 was borrowed by the iter method, so we can't move it out, but,
why does it need to move?, well,
let's take a look at [push](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.push) method:

![Push Method]({{ site.url }}/assets/push_method.png)

The value parameter needs to consume T, in this case book_1,
it doesn't take a reference. What other option do we have?,
we can use [into_iter](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html#tymethod.into_iter),
let's take a look at it:

![IntoIter Method]({{ site.url }}/assets/into_iter_method.png)

It does consume self, moving books into the iterator,
the bad news is that we can't use books anymore, so,
you need to take that into account before using the method.

{% highlight rust %}
#[derive(Debug)]
struct Books {
    title: String,
    author: String,
    subject: String,
    book_id: i32
}

fn main() {
    let book1 = Books { 
        title: "Rust Programming".to_string(),
        author: "Jhon Doe".to_string(),
        subject: "Programming".to_string(),
        book_id: 12
    };

    let book2 = Books { 
        title: "C Programming".to_string(),
        author: "Pete Jhones".to_string(),
        subject: "Programming".to_string(),
        book_id: 13
    };

    let mut books = vec![book1, book2];
    let mut new_books: Vec<Books> = vec![];
    for book in books.into_iter() {
        let mut book_1 = book;
        //do something with book_1
        new_books.push(book_1);
    }
    println!("{:?}", books);
} 
{% endhighlight %}

{% highlight bash %}
error[E0382]: use of moved value: `books`
  --> src/main.rs:31:22
   |
26 |     for book in books.into_iter() {
   |                 ----- value moved here
...
31 |     println!("{:?}", books);
   |                      ^^^^^ value used here after move
   |
   = note: move occurs because `books` has type `std::vec::Vec<Books>`, which does not implement the `Copy` trait
{% endhighlight %}

I know Rust is supposed to be a C/C++ alternative, not a Ruby/Python one, however, 
I feel that the lack of a garbage collector is a feature, because you are now free
of memory leaks,
besides the performance issues that Rust resolve and safeness makes it very attractive.
