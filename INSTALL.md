
Try Dafny Online
================

The easiest way to get started with Dafny is to use [rise4fun](http://rise4fun.com/dafny), where you can write and verify Dafny programs without having install anything. On rise4fun, you will also find the [online Dafny tutorial](http://rise4fun.com/Dafny/tutorial/guide).

Get Started with Visual Studio Code
===================================
1. Install [Visual Studio Code](https://code.visualstudio.com/)
2. If you are on a Mac or Linux, [install Mono](https://www.mono-project.com/download/stable/).  
  We recommend to get the newest version directly from the Mono Project.
3. In Visual Studio Code, press `Ctrl+P` (or `⌘P` on a Mac), and enter `ext install correctnessLab.dafny-vscode`.
4. If you open a `.dfy` file, Dafny VSCode offers to download and install the latest Dafny: 
  ![Dafny Installation](https://raw.githubusercontent.com/DafnyVSCode/Dafny-VSCode/develop/installation.gif)



Install in another an IDE
==============

Dafny offers several IDE integrations for interactive verification.

- **[Visual Studio Code](#get-started-with-visual-studio-code)** (cross-platform)
- **Visual Studio** (Windows only): see [Windows installation instructions below](#windows)
- **[Emacs](#emacs)** (cross-platform)

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

## Windows 

To install Dafny on your own machine, download Dafny.zip from the [releases page](https://github.com/Microsoft/dafny/releases) and **save** it to your disk. Then, before you open or unzip it, right-click on it and select Properties; at the bottom of the dialog, click the Unblock button and then the OK button. Now, open Dafny.zip and copy its contents into a directory on your machine. (You can now delete the Dafny.zip file.)

Then:

-   To run Dafny from the command line, simply run Dafny.exe.
-   To install Dafny for use inside Visual Studio 2012, double-click on `DafnyLanguageService.vsix` to run the installer. You may first need to uninstall the old version of the extension from within Visual Studio (Tools ==\> Extensions and Updates). Then, whenever you open a file with the extension .dfy in Visual Studio, the Dafny mode will kick in. (If you don't intend to run Dafny from the command line, you can delete the directory into which you copied the contents of Dafny.zip.)

## Linux and Mac

Make sure you have [Mono](https://www.mono-project.com/download/stable/). Then save the contents of the Dafny.zip for the appropriate version of your platform. You can now run Dafny from the command line by invoking the script file `dafny`.


Install the source code
=======================

## Windows
First, install the following external dependencies:

-   Visual Studio 2012 Ultimate
-   [Visual Studio 2012 sdk extension](https://visualstudiogallery.msdn.microsoft.com/b2fa5b3b-25eb-4a2f-80fd-59224778ea98)
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
-   `dafny\Source\DafnyExtension.sln`

Last, follow the conventions:

-   Visual Studio
    -   Set "General:Tab" to "2 2"
    -   For `"C#:Formatting:NewLines` Turn everything off except the first option.

## Mac OS X and Linux

Follow the instructions in the [INSTALL.md](https://github.com/Microsoft/dafny/blob/master/INSTALL.md) file in the root directory of the repository.

