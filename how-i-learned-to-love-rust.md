---
title: How I learned to love Rust
date: 2019-01-07
---

# How I learned to love Rust

A journey from hating Rust to loving it more than any other language.

**Note:** In this post, I won't be talking about Rust itself, but rather about
my relationship with it, and how I came to love it. Don’t expect to learn much
about Rust here, but if you haven’t been able to convince yourself to try it out
yet, maybe this post will help!

**TL;DR:** Even though many features appealed to me, I initially hated Rust
because of how much I struggled with the borrow-checker and how hard things were
for a beginner like me. And now, after hundreds (if not thousands) of hours with
it, I love it and I want to use it everywhere.

---

I discovered Rust a few years ago, back when I was mostly trying to
[hack into the C# compiler](https://github.com/71/Cometary). Back then, I hadn't
had much experience with languages that weren't garbage collected, but somehow
Rust’s (amazing) memory management model wasn't the first thing that caught my
eye; what really piqued my interest at first was the language itself, and more
precisely the fact that (almost) everything was an expression. This is something
very common in functional programming, but I had not yet traveled down those
roads, so I couldn’t help but be amazed.

Another feature of Rust that I hadn't seen before was the concept of traits, or
in other languages, type classes / protocols. Thus far, I had only worked with
classes and interfaces (as found in C++, C# and Java), and didn't really get
what traits brought to the table. However, I very quickly realized that the
modularity they brought would make me like them much more than regular ol'
classes and interfaces.

The last feature that sold Rust to me was, surprisingly again, macros.
Meta-programming was another one of these features I loved, and Rust's hygienic
macros were a great plus for me. Since then, meta-programming in Rust has
evolved greatly, and now it _does_ feel like a complete part of the language.

---

With all those features in mind, I was sold on Rust. I _had_ to try it out. And
I did. And I hated it. The Rust compiler refused to compile my code, with at
times several hundred compile-time errors. The simplest thing took me hours,
because the borrow-checker would always find something wrong with my code.

You know the way it goes, though: "It's not you; it's me."

It wasn't Rust's fault for refusing to go through my bug-ridden code; it was
mine. And I knew that. So for months, I kept working (or rather struggling) with
Rust, one step at a time. Thankfully,
[the Rust documentation is insanely good](https://doc.rust-lang.org/), which
helped me through most of my troubles.

Finally, I felt like I was ready to tackle my first big project with Rust: a
fully-fledged programming language, with lexer, parser, and compiler. I had had
many unsuccessful tries designing a language that would fully satisfy me in the
past, coding dozens of iterations of the same language, always starting over
before it got anywhere. But something was different in Rust: **since memory
management was now a concern to me, I was forced to think more thoroughly about
my program, and incidentally, its entire logic**. I was much slower to
accomplish anything in Rust than I ever was in C#; yet, I ended up doing **much,
much more** than I had ever done before.

After several months of work and hundreds of hours of Rust, things started
working: my programming language's JIT (kinda) worked, powering both a
[Jupyter Kernel](https://jupyter.readthedocs.io/en/latest/projects/kernels.html)
and a REPL. Even though Rust has
[a great LLVM crate](https://github.com/TheDan64/inkwell),
[as well as its own low-level compiler](https://github.com/CraneStation/cranelift),
I had made everything from scratch, from lexing to x86 code generation. A sort
of challenge, if you will.

---

However, not everything was all fun and games. Back then,
[non-lexical lifetimes](https://rust-lang-nursery.github.io/edition-guide/rust-2018/ownership-and-lifetimes/non-lexical-lifetimes.html)
were only a dream, and
[the borrow-checker would often complain about perfectly valid code](https://github.com/nikomatsakis/nll-rfc/blob/87b2a0a6648a7d77105b35d0d253468175191d24/0000-nonlexical-lifetimes.md#problem-case-1-references-assigned-into-a-variable),
which meant that one of two things had to be done: use unsafe blocks, or use
slightly less efficient (and often uglier) code. I always went with the former,
which also meant that I went through many unexpected things and segfaults.

Another thing that still isn't great to this day is editor support. Rust has a
[Language Server Protocol implementation](https://github.com/rust-lang/rls), but
it leaves much to be desired, whatever the platform. I would often see the Rust
extension not working for days at a time, before somehow starting to work again.
Not only this, but to this day, Rust is the only language whose tools broke both
Emacs, Vim and Visual Studio Code for me. I can throw any project in any
language to any of these editors, and asynchronous completion will make sure it
never freezes, but with (intermediate-sized) projects in Rust, every single one
of these editors freeze during most of their execution time, which forced me to
stop using the Rust editor tools altogether when working on these projects. As a
comparison, I can open [Roslyn](https://github.com/dotnet/roslyn) on my computer
in a few seconds, even though that project is several dozen times bigger than
the one I was working on in Rust. It's not all bad though: when it does work,
the editor support is pretty good, showing auto-completion, documentation
comments and inline tests (in Visual Studio Code).

Finally, after having worked that much with Rust, I started using other
languages again, but... I always wanted to come back to Rust. Doing anything in
a garbage collected language would make me slightly uncomfortable, because of
all the allocations I had to make (even though I _knew_ the garbage collector
would make this extremely fast anyway). Lower-level languages like C and C++
didn't cut it either, because of their lack of traits, safety or ecosystem.
Plus, I didn't like C++'s approach to implicitly copying most things, like
vectors; or its very verbose use of `std::move`.
