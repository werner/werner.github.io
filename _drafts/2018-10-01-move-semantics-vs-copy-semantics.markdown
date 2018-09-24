---
layout: post
comments: true
title: "Move Semantics vs Copy Semantics"
date: 2018-10-01 13:47:00 -0430
categories: language ownership type-system
---

This is a post I wanted to write because I feel the moment I clearly understand move semantics I was able to
use Rust in a basic level. Obviously Rust has a steep learning curve, specially for people like me, who comes from Ruby,
so I thought this could help Rust beginners like myself to learn how to use the language.

In a few situations you want copy semantics, which means the whole value is copy from one point of memory to another.
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
$ gcc test.c -o test
$ ./test   
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

So, what happened?, the pointer address book1 was copied into book2, making book1 invalid, which means that now book2 owns the data.
 What can we do in this case?, depending on the situation, we can borrow the value like this: `let book2 = &book1;` or we can implement the Clone trait like this:

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

## **iter method**

In another situation, you want move semantics, like this one:

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

    let books = vec![book1, book2];
    let mut new_books: Vec<Books> = vec![];
    for book in books.iter() {
        if book.book_id > 12 {
            new_books.push(book);
        }
    }
} 
{% endhighlight %}

{% highlight bash %}
error[E0308]: mismatched types
  --> src/main.rs:29:28
   |
29 |             new_books.push(book);
   |                            ^^^^ expected struct `Books`, found &Books
   |
   = note: expected type `Books`
              found type `&Books
{% endhighlight %}