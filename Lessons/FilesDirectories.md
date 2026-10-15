# 📜 Day 4: Files & Directories

### ⏱ Agenda

1. [[**05m**] 🏆 Objectives](#05m--objectives)
2. [[**60m**] 💻 Work on SSG MVP](#60m--work-on-ssg-mvp)
3. [[**30m**] ✓ Review](#30m--review)
4. [[**15m**] 🌴 BREAK](#15m--break)
5. [[**30m**] 📚 Overview: Files & Directories](#30m--overview-files--directories)
6. [[**30m**] 💻 Work on SSG v1.1](#30m--work-on-ssg-v11)
7. [📚 Resources & Credits](#-resources--credits)

## [**05m**] 🏆 Objectives

TODO

## [**60m**] 💻 Work on SSG MVP

Use this time to make progress on your MVP.

## [**30m**] ✓ Review

Ask class to close laptops. Demonstrate breaking down requirements into function stubs.

Call on individuals to live code their solutions to the first three requirements. Invite the class to analyze each solution and determine why it works.

## [**15m**] 🌴 BREAK

## [**30m**] 📚 Overview: Files & Directories

### Introduction

The `os` package we used last class period to read and write files can also list directories.

Its `ReadDir` function is an approachable way to list all the files in a directory. Check out the method signature below:

```golang
func ReadDir(name string) ([]DirEntry, error)
```

`ReadDir` reads the directory named by `name`, and returns a list of directory entries sorted by filename.

### Code Snippet: Traversing a Directory Tree

Create a new directory named `traversing`, with a file inside named `main.go`.

Paste the code below, then build and run it. Examine the output and syntax.

```golang
package main

import (
    "fmt"
    "log"
    "os"
)

func main() {
    directory := "."
    files, err := os.ReadDir(directory)
    if err != nil {
        log.Fatal(err)
    }

    for _, file := range files {
        fmt.Println(file.Name())
    }
}
```

### Ignoring Return Values

To safely ignore returned values from a function, use the `_` in place of any variable name.

### DirEntry Interface

You might have noticed that the method signature for `ReadDir` returns a slice of [`DirEntry`](https://pkg.go.dev/os#DirEntry) values.

Change the value of the `directory` variable, and set it to point to your home directory. For example, `/Users/YOUR_MAC_USERNAME`.

Click the link in the last sentence, then modify the example code to print out the name of the path and a label indicating if the path is a directory or not (HINT: use `IsDir()`).

## [**30m**] 💻 Work on SSG v1.1

1. ⭐️ **IMPORTANT**: Finish the [requirements](https://github.com/Tech-at-DU/makesite#mvp) for the MVP before beginning v1.1.
2. Begin working on the [requirements](https://github.com/Tech-at-DU/makesite#v11) for SSG v1.1. **Be sure to copy them into your project's README to keep track of your progress**!

## 📚 Resources & Credits

- [**List the files in a folder with Go**](https://flaviocopes.com/go-list-files/): How to get a list of files inside a folder on the filesystem using Go.
