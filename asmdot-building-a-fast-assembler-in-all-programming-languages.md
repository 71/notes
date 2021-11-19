---
title: "ASM.: Building a fast assembler in all programming languages"
date: 2018-06-27
---

# [ASM.](https://github.com/71/asmdot): Building a fast assembler in all programming languages

ASM. originally aimed to create a fast, minimalist and unopinionated assembler
in C that could live in a single header file, and support multiple
architectures.

For this purpose, a Python library was built to transform various instructions
from different architectures into a simple, common AST that supports bitwise and
logical expressions, basic flow control and variables into C code.

Basically, I wanted to transform these "descriptions":

```
40  inc              r16-32  
48  dec              r16-32
```

Into this C code:

```c
void inc_r16(void** buf, Reg16 operand) {  
    *(uint8_t*)(*buf) = 102 + get_prefix(operand);  
    *(byte*)buf += 1;  
    *(uint8_t*)(*buf) = 64 + operand;  
    *(byte*)buf += 1;  
}

void inc_r32(void** buf, Reg32 operand) {  
    if (operand > 7)  
    {  
        *(uint8_t*)(*buf) = 65;  
        *(byte*)buf += 1;  
    }  
    *(uint8_t*)(*buf) = 64 + operand;  
    *(byte*)buf += 1;  
}

void dec_r16(void** buf, Reg16 operand) {  
    *(uint8_t*)(*buf) = 102 + get_prefix(operand);  
    *(byte*)buf += 1;  
    *(uint8_t*)(*buf) = 72 + operand;  
    *(byte*)buf += 1;  
}

void dec_r32(void** buf, Reg32 operand) {  
    if (operand > 7)  
    {  
        *(uint8_t*)(*buf) = 65;  
        *(byte*)buf += 1;  
    }  
    *(uint8_t*)(*buf) = 72 + operand;  
    *(byte*)buf += 1;  
}
```

Furthermore, since code would be generated automatically, language-specific
naming conventions and types could be easily modified when generating the
sources. Thus, instead of having functions named `inc_r16` and `inc_r32`, we
could prefix them with their architecture, because C has no concept of module
and cannot import multiple symbols with the same name.

---

After implementing the translation process, I soon realized that it could be
easily extended to not only support C, but also other programming languages with
different syntax, types and conventions.

Therefore, I set out on another goal: produce an AST for each instruction, and
then translate it to C code. Using the abstract signature of each function, it
would be easy to create bindings to the C API.

However, when testing the Python bindings, I came to the realization that using
FFI over native code either introduces performance _losses_, or negligible gains
in **an interpreted language**. If Python is faster than C when taking the FFI
overhead into consideration, then surely other compiled languages such as Nim or
Rust will actually be _faster_ if FFI is dropped in favor of native code.

And thus, another choice was made: bindings would no longer be generated;
instead, all languages were given the ability to generate native sources.

---

The generated AST is still optimized for C generation (its assembly output is
analyzed in both GCC and Clang to find the best possible AST), but performances
should still be very good across all languages.

This change also allows to better adapt to each language. For example, using
bindings in C# would mean passing `ref IntPtr buffer` to each function, which is
**not** .NET-friendly. With the new changes, C# sources now use the standard
`Stream` class instead of pointers, making it much more easy to integrate ASM.
into existing projects.

Finally, after hand-writing enumerations and distinct types in C, C#, Haskell,
Nim, Python and Rust individually, I decided to integrate them to the AST and
generate these things automatically, making maintenance and addition of new
languages or declarations much easier. Later on, the same choice was made with
the test suites.

---

After all these changes, ASM. has reached a rather stable state.

The ASM. package is tasked with returning functions, enumerations, distinct
types and test suites, which may then be output by any Python script.

This allows anyone to create assemblers for their own programming language
without contributing to the project directly -- it is enough to create a script
`lang.py`.

---

At the time of writing, [ASM.](https://github.com/71/asmdot) supports the Arm,
Mips and x86 architectures partially, and translates them to C, C#, Haskell,
Nim, Python and Rust. It also generates test suites in all these languages to
verify that the implementation is correct everywhere.
