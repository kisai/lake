# Clake - Clake is a [Rake](https://github.com/ruby/rake)-like program implemented in Common Lisp.
[![Build Status](https://circleci.com/gh/Rudolph-Miller/clake.svg?style=shield)](https://circleci.com/gh/Rudolph-Miller/clake)

## API

See [Document](https://rudolph-miller.github.io/clake/overview.html).  
This HTML is generated by [Codex](https://github.com/CommonDoc/codex).

### [Function] clake

    CLAKE &key target pathname

Loads a Clakefile specified with `pathname` to execute a task of name `target` declared in the loaded Clakefile. If `target` is not given, "default" is used for the default task name. If `pathname` is not given, a file of name `Clakefile` in the current directory is searched.

## Command line interface

Clake can be also used via command line interface. Specifying a number of targets is allowd in the command line interface.

**SYNOPSIS**

    clake [-f clakefile] targets...

**OPTIONS**

    -f [FILENAME]
        (Not implemented) Uses FILENAME as the clakefile to search for.

## Current design policy

For now trusting rake's design against preceding GNU make to follow after it.

## API design exploration

Here shows some API design examples. 

    ;; Do the default task for Clakefile in the current directory.
    ;; TBD: 以下のどちらを採用するか？
    ;;      ・Unix 的に、カレントディレクトリや asdf:system-source-directory の直下の Clakefile を探す
    ;;      ・Lisp 的に、先にタスク定義を load しておく。ASDF
    ;;      -> 1) REPL からは Lisp 的に、シェルからは Unix 的に振る舞うのがよい、とする
    ;;            パケージについてどうしよう？ターゲット名であると同時に、ファイル名でもある
    ;;           ファイル名について「基準となるディレクトリ」という概念が必要。もはや Lisp 世界の話ではない
    ;;         2) Unix の世界を Lisp から間接的に操作するものなので、Unix 的に振る舞えばよい、とする
    (clake:clake)

    ;; Do a task specifying its name for Clakefile in the current directory.
    (clake:clake "hello")
    
    ;; Do a task in namespace.
    (clake:clake "hello:say")
    
    ;; Also do a file task in namespace.
    (clake:clake "hello:hello")
    
    ;; Shell command interface is provided for convenience.
    (clake:sh "pwd") 

## Clakefile design exploration

Here shows what and how Clakefile should express.

    (in-package :cl-user)
    (defpackage :hello.clake
      (:use :cl :clake))
    (in-package :hello.clake)

    ;; A task that print hello.
    (task "hello" ()
      (print "do task hello!")
      (terpri))

    ;; Tasks that build an executable with dependency.
    (defparameter cc "gcc")
    
    (file "hello" ("hello.o" "message.o")
      (sh #?"#{cc} -o hello hello.o message.o"))
    
    (file "hello.o" ("hello.c")
      (sh #?"#{cc} -c hello.c"))

    (file "message.o" ("message.c")
      (sh #?"#{cc} -c message.c"))

    (task "clean" ()
      (sh "rm -f hello hello.o message.o"))

    ;; The "all" task.
    (task "all" ("hello" "som"))
    
    (file "hello" ("hello.c")
      (sh #?"#{cc} -o hello hello.c"))
    
    (file "som" ("som.c")
      (sh #?"#{cc} -o som som.c"))

    ;; Namespaces
    (namespace "hello"
    
      ;; TBD: How should depencency in a namespace behave?
      ;;      Does this "hello" dependency refer a task only in "hello" namespace?
      (task "say" ("hello")
        (sh "./hello"))
      
      (file "hello" ("hello.c")
        (sh #?"#{cc} -o hello hello.c"))
      
      ;; A file task should depend on only file tasks, not on tasks.
      (file "hello.o" ("say")
        (sh "false")))

    ;; Can't use colons in a task name because they are reserved for namespace delimiter.
    (task "hel:lo" ())
    
    ;; Make a directory.
    (directory "some/directory")

## Task kinds

There are some kinds of "Task" representing a sequence of shell commands.

|Name|Clakefile|Description|
|---|---|---|
|Task|task|This represents a base concept processing a sequence of shell commands.|
|File task|file|This represents a task resolving file dependency with up-to-date check.|
|Directory task|directory|Make a directory.|

## Design requirements
- dynamic task definition
- expected number of tasks in a Clakefile is ~100
- up-to-date check
- duplicated task names allowed, executed respectively
- make(1)'s internal macros
- ros interface
- modularity
- parallel task execution

## Author

* Rudolph Miller (chopsticks.tk.ppfm@gmail.com)

## Copyright

Copyright (c) 2015 Rudolph Miller (chopsticks.tk.ppfm@gmail.com)

## License

Licensed under the MIT License.
