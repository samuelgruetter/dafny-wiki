
Try Dafny Online
================

The easiest way to get started with Dafny is to use [rise4fun](http://rise4fun.com/dafny), where you can write and verify Dafny programs without having install anything. On rise4fun, you will also find the [online Dafny tutorial](http://rise4fun.com/Dafny/tutorial/guide).

You'll get a much better Dafny experience from the Dafny IDEs, since they continuously run the verifier in
the background. For most users, we recommend using Dafny with VS Code, which has an easy installation, as
explained next:

Get Started with Visual Studio Code
===================================
0. Install [Visual Studio Code](https://code.visualstudio.com/)
1. If you are on a Mac or Linux, [install Mono](https://www.mono-project.com/download/stable/).  
  We recommend to get the newest version directly from the Mono Project.
2. In Visual Studio Code, press `Ctrl+P` (or `⌘P` on a Mac), and enter `ext install correctnessLab.dafny-vscode`.
3. If you open a `.dfy` file, Dafny VSCode offers to download and install the latest Dafny: 
  ![Dafny Installation](https://raw.githubusercontent.com/DafnyVSCode/Dafny-VSCode/develop/installation.gif)



Install in another an IDE
==============

Dafny offers several IDE integrations for interactive verification.

- **[Visual Studio Code](#get-started-with-visual-studio-code)** (cross-platform)
- **[Emacs](#emacs)** (cross-platform)
- **Visual Studio** (Windows only): see [Windows installation instructions below](#windows)

## Emacs

The README at https://github.com/boogie-org/boogie-friends has plenty of
information on how to set-up Emacs to work with Dafny. In short, it boils down
to running `[M-x package-install RET boogie-friends RET]` and adding the following
to your `.emacs`:
```
(setq flycheck-dafny-executable "BASE-DIRECTORY/dafny/Binaries/dafny")
```

Do look at the README, though! It's full of useful tips.



Install the binaries
====================

Note, if you install Dafny via VS Code using the steps above, you're done. The following instructions
tell you how to install Dafny so that you can run it from the command line or, on Windows, Visual Studio.

## Windows 

To install Dafny on your own machine, download Dafny.zip from the [releases page](https://github.com/Microsoft/dafny/releases) and **save** it to your disk. Then, **before you open or unzip it**, right-click on it and select Properties; at the bottom of the dialog, click the **Unblock** button and then the OK button. Now, open Dafny.zip and copy its contents into a directory on your machine. (You can now delete the Dafny.zip file.)

Then:

-   To run Dafny from the command line, simply run Dafny.exe.
-   To install Dafny for use inside Visual Studio, double-click on `DafnyLanguageService.vsix` to run the installer. You may first need to uninstall the old version of the extension from within Visual Studio (Tools ==\> Extensions and Updates). Then, whenever you open a file with the extension .dfy in Visual Studio, the Dafny mode will kick in. (If you don't intend to run Dafny from the command line, you can delete the directory into which you copied the contents of Dafny.zip.)

## Linux and Mac

Make sure you have [Mono](https://www.mono-project.com/download/stable/). Then save the contents of the Dafny.zip for the appropriate version of your platform. You can now run Dafny from the command line by invoking the script file `dafny`.


Install the source code
=======================

If you want access to the very latest Dafny developments or you want to extend Dafny yourself, get the source code, as explained here.

## Windows

First, install the following external dependencies:

-   Visual Studio 2017 or newer
-   [Visual Studio SDK extension](https://visualstudiogallery.msdn.microsoft.com/b2fa5b3b-25eb-4a2f-80fd-59224778ea98)
-   [Code contract extension](https://visualstudiogallery.msdn.microsoft.com/1ec7db13-3363-46c9-851f-1ce455f66970)
-   [NUnit test adapter](https://visualstudiogallery.msdn.microsoft.com/6ab922d0-21c0-4f06-ab5f-4ecd1fe7175d)
-   To install lit (for test run):
    -   first, install [python](https://www.python.org/downloads/)
    -   second, install [pip](http://pip.readthedocs.org/en/stable/installing/)
    -   last, run "pip install lit" and "pip install OutputCheck"

Second, clone source code.

-   [Dafny](https://github.com/Microsoft/dafny)
-   [Boogie](https://github.com/boogie-org/boogie)
-   [BoogiePartners](https://github.com/boogie-org/boogie-partners)
-   copy [Coco.exe](http://www.ssw.uni-linz.ac.at/Research/Projects/Coco/) to `\boogiepartners\CocoRdownload`

Third, build the following projects in the following order:
-   `boogie\Source\Boogie.sln`
-   `dafny\Source\Dafny.sln`
-   `dafny\Source\DafnyExtension.sln` (if you want to run Dafny in the Visual Studio IDE; you don't need this step if you'll only be using Dafny in VS Code, Emacs, or the command line)

Last, follow the conventions:

-   Visual Studio
    -   Set "General:Tab" to "2 2"
    -   For `"C#:Formatting:NewLines` Turn everything off except the first option.

## Mac OS X and Linux

Follow the instructions in the [INSTALL.md](https://github.com/Microsoft/dafny/blob/master/INSTALL.md) file in the root directory of the repository.

## Compiling to JavaScript and Go, and running the test suite

By default, Dafny compiles to .NET. If you want to compile to JavaScript, make sure you
have node.js installed (see below) and then use the `/compileTarget:js` flag. If you want
to compile to Go, make sure you have Go installed (see below) and then use the
`/compileTarget:go` flag.

Note, the tests in the `Test/comp` folder run the Dafny compiler to produce code for .NET,
JavaScript, and Go. If you don't have node.js and Go installed and on your path, these tests
will be disabled and you will get a "unsupported" message from `lit`.

NOTE: (6 May 2019) Under Windows, the test suite does not properly capture the output from
the Go programs in `Test/comp`. More precisely, under Windows, when `lit` invokes Dafny
with `/compile:3 /compileTarget:go`, the output does not show up (but running Dafny directly
with those options is fine, as is running `go run` on the program that Dafny generates).
If you know how to fix this problem, please provide a Pull Request or contact the Dafny
developers.

To set up Dafny to compile to JavaScript (node.js):
* Install node.js from https://nodejs.org/en/download
* Make sure `node` and `npm` are on your path
* Install the bignumber.js package (see https://github.com/MikeMcl/bignumber.js) by running:
  `npm install bignumber.js`

To set up Dafny to compile to Go:
* Install Go from https://golang.org/dl
* Make sure `go` in on your path

Dafny's command-line option `/compile:3` verifies, compiles, and runs your program. It works
with any compilation target. If you want to do some of these steps manually (say, for a
verifying program named `MyProgram.dfy` that has a `Main` method), here's how:
* To separately compile and run your program for .NET:
  - `dafny MyProgram.dfy`
  - `MyProgram.exe`
* Dafny can also output your .NET-compiled program as a C# program, which you can then
  compile yourself:
  - `dafny /spillTargetCode:1 MyProgram.dfy`
  - See the generated `MyProgram.cs` file for how to invoke `csc` on `MyProgram.cs`
* To separately compile and run your program for JavaScript:
  - `dafny /compileTarget:js /spillTargetCode:1 MyProgram.dfy`
  - `node MyProgram.js`
* To separately compile and run your program for Go:
  - `dafny /compileTarget:go MyProgram.dfy`
  - This created a folder `MyProgram-go` with the target files. You must set the `GOPATH`
    to point to this folder, and Go insists that the path you give is an absolute path
    (including the `C:` on Windows).
  - After setting `GOPATH`, do `go run MyProgram-go/src/MyProgram.go`

