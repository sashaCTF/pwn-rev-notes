# Introduction

### What is binary exploitation?

Binary exploitation is when you manipulate a program to make it behave in unintended ways, which could include getting remote code execution \(commonly popping a shell\), bypassing login checks, etc. We do this by examining the program with reverse engineering, and search for possible attack vectors, such as a `buffer overflow` where too much data is written into a buffer, or a `printf` exploit where we control the format string argument, or a `UAF` where freed chunks can be referenced later, and so on. Point is there's quite a few methods of exploiting programs, and I'll go over many of the techniques I know of.

