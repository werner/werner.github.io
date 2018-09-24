---
layout: post
comments: true
title: "Move Semantics vs Copy Semantics"
date: 2018-10-01 13:47:00 -0430
categories: language ownership type-system
---

This is a post I wanted to write because I feel the moment I clearly understand move semantics I was able to
use Rust in a basic level. Obviously Rust has a steep learning curve, but this could help Rust beginners like myself to learn
how to use the language.

In a few situations you expect copy semantics, which means they copy the whole value from one point of memory to another.
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

    struct Books *struct_pointer;

    strcpy( book1.title, "C Programming");

    struct Books book2;

    book2 = book1;

    strcpy( book2.title, "Copy Semantics");

    printf( "%s\n", book1.title );

    printf( "%p\n", &book1 );
    printf( "%p\n", &book2 );
}
{% endhighlight %}

{% highlight bash %}
gcc test.c -o test
➜  Desktop ./test   
C Programming
0x7ffca2811e30
0x7ffca2811f00
{% endhighlight %}

As you can see, the variables point to different memory addresses, now if you want to do the same thing in Rust:

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