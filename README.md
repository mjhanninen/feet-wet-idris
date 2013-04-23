Getting your feet wet with Idris
================================

Let's start with what this article is not. This is not a tutorial on Idris the
Language. Please see [the official tutorial][brady13] for that.

This is more of a personal travel report as I get to know Idris and try to
apply it into small practical problems; which implies couple things. Firstly,
this is a work-in-progress that I will push slowly forward as I drill deeper
into the language myself. Secondly, I emphasize the practical side of the
things paying little attention to the nifty theoretical stuff. There are other
sources on the interwebs that cover that stuff.

Finally as a word of warning I am not versed in Idris and claim no expertise
on the language.


Preliminaries
-------------

*In which we obtain the tools as well as the source code to use as a
 reference.*

I assume that you have Haskell and Cabal installed. If you don't then google
up the instructions and install them first. Once you these dependencies
installed obtain Idris and check that it works:

    $ cabal update
    $ cabal install idris
    $ idris
         ____    __     _
        /  _/___/ /____(_)____
        / // __  / ___/ / ___/     Version 0.9.7
      _/ // /_/ / /  / (__  )      http://www.idris-lang.org/
     /___/\__,_/_/  /_/____/       Type :? for help

    Idris> :quit
    Bye bye

Next you should obtain the sources for Idris. You may ask why and the answer
would be that Idris being a young language there is no nice reference books
available for it yet. Or even a decent tutorial. So, read the source, Luke!

    $ cd your/3rd-party/stuff
    $ mkdir idris
    $ cd idris
    $ git clone git://github.com/edwinb/Idris-dev.git

Editor support is not really required but is always nice. In case you are
using Emacs and would like to have some support from your editor there is an
[Idris mode for Emacs][startling] available. To install it:

    $ cd your/3rd-party/stuff
    $ git clone https://github.com/startling/idris-mode.git

And then add your variation of the following to `.emacs` file:

    (add-to-list load-path "your/3rd-party/stuff/idris-mode")
    (require 'idris-mode)

Now you should be good to go.


### Exercises ###

1. Go into the directory `path/to/idris/Idris-dev/tutorial/examples` and read
   through all the files you find from the directory. Probably the easiest way
   to do so is `less *.idr` and then move back and forth between the file with
   `:n` and `:p`.

2. Now do the same with by going into the directory `path/to/idris/Idris-dev`
   and running the command `git ls-files examples*.idr | xargs less`. Try also
   `git ls-files samples*.idr | xargs less`.

3. Find out where the function `fread` is mentioned in the code base. Try `git
   grep fread -- *.idr`. To narrow the query down to definition only try `git
   grep '^fread :' -- *.idr`.

4. Find all functions that deal with `File`. Try for example `git grep
   ':.*File' -- *.idr`.

5. Read `lib/Prelude.idr`.


Getting minimal program compiled and run
----------------------------------------

*In which we learn to compile and run a simple program and get to know the
 REPL a little bit better*

Type your first program into the file `hello.idr`:

    module Main

    main : IO ()
    main = putStrLn "Hello, world!"

Compile and run it:

    $ idris hello.idr -o hello
    $ ./hello
    Hello, world!

Now do the same using the console:

    $ idris
         ____    __     _
        /  _/___/ /____(_)____
        / // __  / ___/ / ___/     Version 0.9.7
      _/ // /_/ / /  / (__  )      http://www.idris-lang.org/
     /___/\__,_/_/  /_/____/       Type :? for help

    Idris> :load hello
    Skipping ./hello.idr
    *hello> :exec
    Hello, world!
    *hello> :quit
    Bye bye

Note that Idris is telling that it skips `hello.idr`. What happens is that the
compiler realizes that nothing has changed since last compilation and it can
skip the compilation phase. If you peek what the directory contains you should
see the following:

    $ ls
    hello*  hello.ibc  hello.idr

Accompanying the binary `hello` there is a `hello.ibc` file that is compiled
byte code version of the `hello.idr` that gets loaded with `:load` command. To
prove this to yourself try the following:

    $ rm hello.ibc
    $ idris
         ____    __     _
        /  _/___/ /____(_)____
        / // __  / ___/ / ___/     Version 0.9.7
      _/ // /_/ / /  / (__  )      http://www.idris-lang.org/
     /___/\__,_/_/  /_/____/       Type :? for help

    Idris> :load hello
    Type checking ./hello.idr
    *hello> :exec
    Hello, world!
    *hello> :quit
    Bye bye

This time Idris reports type checking the file when it loads.


### Exercises ###

1. Try the command `:?` that lists the available interactive commands and go
   over the list carefully.

2. With `hello.idr` loaded check the type of `main` by running the command `:t
   main`.

3. With `hello.idr` loaded try applying every command that takes `<name>` as
   an argument on the function `main` (for example, `:info main`). Most of the
   combinations don't make sense but ponder on the resulting error messages.

4. Try both short and long versions of the commands. For example the short
   form `:m` works okay but the long form `:metavars` results in error. This
   is most probably a bug.


Doing basic I/O
---------------

*In which we develop skills to interact with the user, file system, and
 command line*

Open the `hello.idr` file you created in the previous section and modify
it to contain the following:

    module Main

    main : IO ()
    main =
      do putStrLn "What is your name?"
         n <- getLine
         putStr $ "Hello, " ++ n

and try the modified version on the interpreter:

    *hello> :reload
    Type checking ./hello.idr
    *hello> :exec
    What is your name?
    Jack
    Hello, Jack

Now type the following program into `readme.idr`:

    module Main

    echoLine : File -> IO ()
    echoLine file =
      do l <- fread file
         putStr l

    overLines : (File -> IO ()) -> File -> IO ()
    overLines fn file =
      do x <- feof file
         if not x
           then do fn file
                   overLines fn file
           else return ()

    main : IO ()
    main =
      do file <- openFile "readme.idr" Read
         overLines echoLine file
         closeFile file

Upon executing the program you should see the program printing out its program
listing. Next let extend that into naive implementation of the Unix `cat`
utility. So `cat.idr` goes:

    module Main

    import System

    echoLine : File -> IO ()
    echoLine file =
      do l <- fread file
         putStr l

    overLines : (File -> IO ()) -> File -> IO ()
    overLines fn file =
      do x <- feof file
         if not x
           then do fn file
                   overLines fn file
           else return ()

    echoFile : String -> IO ()
    echoFile n =
      do file <- openFile n Read
         overLines echoLine file
         closeFile file

    echoFiles : List String -> IO ()
    echoFiles [] = return ()
    echoFiles (n :: ns) =
      do echoFile n
         echoFiles ns
         exit 0

    printUsage : String -> IO ()
    printUsage progName =
      do putStrLn $ "usage: " ++ progName ++ " file [file ..]"
         exit 1

    main : IO ()
    main =
      do args <- getArgs
         case args of
           [progName] => printUsage progName
           (_ :: fileNames) => echoFiles fileNames


[brady13]: http://www.cs.st-andrews.ac.uk/~eb/writings/idris-tutorial.pdf
     "Programming in Idris: a tutorial"

[startling]: https://github.com/startling/idris-mode
     "Idris mode for Emacs"
