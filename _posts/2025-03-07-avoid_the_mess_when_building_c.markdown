---
layout: post
title:  "Building C: Avoid the mess"
date:   2025-03-07 00:00:00 -0400
---

## Initially avoid Make, CMake, etc.
My advice is to keep it simple. A bash script `build.sh` that calls `gcc` or `clang` is all you need. No wildcards, nothing dynamic -- just a straight-up declarative build command. Where this strategy gets overlooked is due to wanting to over organize the project too early. The more `*.h` and `*.c` files you creating, the more bothered you may think it is needing to add the file to the build script. It's not so bad. Honestly I generally like to have as few file as possible when growing a C program. In the event a new file needs to be extracted, The build fails and it's usually fairly clear what is missing.

One of the frustrations I often see with people attempting to write C programs is how complicated build systems can be. Something as "simple" as a Makefile can quickly get out of control with all it's fancy wildcards and syntax. There were several points in my journey of recreational C programming where time would be wasted crafting the perfect Makefile to accept anything thrown at it. It was a complete distraction from writing the program. Once I even try to replace `Make` with `Rake` just to have hopefully a more friendly and maintainable build system. It was still too complicated.

## A single translation unit
An optimization that is often introduced too soon is multiple translation units. When you want to compile `*.c` files into `*.o` to prevent recompiling files that don't change -- that introduces much of the complexity. This is an optimization that can be introduced later and is totally unnecessary for an infant project. "Wastefully" recompile the whole project until you have a reason not to. This can evolve from having a single `main.c` contain your entire program to then having `main.c` include other `*.c` files to add a bit of organization. Once you need to access the same function in different files, only then you extract `*.h` and `*.c` pairs and include `*.h` files instead of `*.c` files.

## A build tool that I can get behind
If I were to reach for a build tool, I would go with a lesser known one called `nob.h`. The idea is that you only need C compiler to build C code. This makes it incredibly portable and has no dependency on something like `Make`. There's no context switching to writing your build, it's just C. This library has come in handy for situations where I'm moving between operating systems.

Using it is simple. Write a main function for your build and append compiler commands.
```
// nob.c
#define NOB_IMPLEMENTATION
#include "nob.h"

int main(int argc, char **argv)
{
    NOB_GO_REBUILD_URSELF(argc, argv);
    Nob_Cmd cmd = {0};
    nob_cmd_append(&cmd, "cc", "-Wall", "-Wextra", "-o", "main", "main.c");
    if (!nob_cmd_run_sync(cmd)) return 1;
    return 0;
}
```

Build it once -- now the program `./nob` will rebuild itself and your project. Neat!
```
$ cc -o nob nob.c
$ ./nob
```

Check out [nob.h on GitHub](https://github.com/tsoding/nob.h)
