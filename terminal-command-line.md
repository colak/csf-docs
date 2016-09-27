# Learning Terminal

**_An instructional essay for Mac owners who have never used Terminal before. Because… why not‽_**

***

Maybe you have a Mac because you’ve been sucked in by the Apple-fandom tractor beam. Or because you’re gratified by the chic, sleak lines of a Mac laptop’s aluminum chassy and space-faring tone. Or because you like the fact Mac computers mimic the BSD  UNIX operating system. Or maybe — as in my case — you just appreciate that using a Mac is a happier and more productive experience. Those are all influential reasons, we can agree, and there are more. But I have to ask… _Are you using Terminal?_ 

As a Mac owner, you have an obligation to use Terminal, almost as much as Linux users. An _obligation_! It’s your responsibility to exercise Terminal, and discover what it can do to make your days more productive. Maybe you didn’t know that, but now you do. 

Not using Terminal is like having a magic lamp and never calling out the genie. The lamp looks pretty sitting on the coffee shop table but acts derelict and dumb. That’s not the lamp’s fault, nor the genie’s; it’s your fault for not living up to your lamp-master duties. You need to buff that aluminum vessel once in a while, if not on a daily basis, and evoke the Terminal mist of power. Only a [macpos.er](https://macpos.er) keeps the genie locked away and forgotten. Lucky for you that site doesn’t exist, or your name would be flashing in the scrolling marquee.

If you didn’t know the genie was there, waiting to weild the white arts and to conjure at your command, The Creator in Cupertino forgives you, as long as you buy more of The Creator’s products and change the error of your ways. But if you knowingly forsake the sacred power of Terminal, you should give up your Mac-owning priviledge immediately. Send your MBP to me and buy yourself a Lenovo instead, a proper Microsoft Office tool. Dare not display your Mac in public without Terminal at least running in the background, or you’ll earn the unflinching glares of the learned and pay the price of productivity loss. 

Are you ready to learn Terminal, take back your life, save the world, and justify owning a Mac?

## _What the hell are you talking about and why should I care?_

Terminal — more precisely, Terminal.app — is Apple’s command-line client. You can find it via Finder in **Applications** > **Utilities**. Drag Terminal’s cool, black icon to the center of your Dock so it stares back at you like the eye of Sauron, beckoning to be opened and used every day.

Terminal can do a lot, most of which is out of scope for one introductory document. But the first thing to understand is that everything centers around working with files. And I _know_ you work with files because that’s where the content is, and content is _everything_! It doesn’t matter whether those files are on your hard drive, an external drive connected via a USB port, another computer in your house, or your host’s web server on the other side of the world. Content within files within directories within file trees within disk drives anywhere. And that’s why you should care. Terminal is your one app to rule them all with ease, if you so desire. And you should desire it. The genie awaits! 

But let’s not get ahead of ourselves with visions of hacking Evil Corp. The scope here is the file tree on your own machine. Accessing the drives of other locations is homework for another day. 

There are two main types of actions involved with using a command-line client like Terminal or any other: navigating file trees, and manipulating files in them.

By manipulation I mean tasks like logging in and out of secure files; uploading and downloading files to/from remote locations; packing and unpacking file sets; executing files; adding, moving, deleting, renaming, and edit files; and so on. Basically, everything you do on a daily basis but typically through graphical user interaces (GUIs). 

There’s nothing wrong with GUIs. Many applications, notably those dealing with images, would be impossible, or at least painful, to use without GUIs on macOS. But generally speaking, we rely on GUIs too much. There are many, many interstial actions we make at the computer every day that could be done more quickly and efficiently via the command-line than by click, click, clicking around in a dozen GUI ducks. (That was a joke, geoducks, get it?)   

Everything you do on the computer is a series of unique actions. Many of those actions can be accomplished more efficiently by running a specific command in Termainal; commands you can type faster than by clicking through a series of menus. The processes execute faster too, so less waiting around as something starts up, downloads, refreshes or whatever. 

The commands you will use on a regular basis are extremely simple, while more sophisticated commands are rarely necessary. But as you’re familiarity with the command-line grows, so will your thirst for applying more exciting tricks, and your ability to do more advanced things.

In that light, this document focuses on the first thing you need to learn about using the command-line — the working context. We’ll begin with important terminology, explore the concept of working context, and introduce two of the most common programs you’ll use when running commands. I’ll wrap it up with some insights to fast-track further learning, and you’ll be on your own from there. 

Once you’re familiar with command-line anatomy, working context, navigating your file tree, and expanding your knowledge of programs to use, there’s no holding you back.

## A) Terminology

You must know how to talk about this stuff so you can follow the instructions of others, and others will understand you when you have questions.

### Command-line interface: 

When you open Terminal, you’re looking at the command-line interface (CLI), GUI’s smarter cousin. It’s a simple Mac application window that you can resize to your liking, change the color of (via **Terminal** > **Preferences…** > **Profile**), and open multiple tabs in. 

Tabs are wonderful! Want to work on that essay, play inside a database shell, and tunnel into your web server to update your production content all at the same time? Use tabs to keep those commands isolated so console history is easier to follow in each case.

### Console 

The working area of the CLI (or of each tab, if you have multiple tabs open). It’s here where you run commands and read the output you generate.

### Command-line

A special line in the console where you add and run commands. The command-line is easily identified by the leading command-line prompt. You type or paste commands immediately after the prompt and hit Enter to run them. The active command-line will always be at the bottom of console output, waiting for your next orders.

### Prompt

The opening string on the command-line that ends with a `$`. The prompt serves to show where to add your commands and what context you’re in when working. The prompts of commands you already ran will appear with a leading straight bracket on them.

### Command

Commands are the orders you give to your genie, the command-line intepreter (i.e. the [bash shell](https://en.wikipedia.org/wiki/Bash_(Unix_shell), a UNIX application that accepts and executes commands). 

Commands are written on the command-line immediately after the prompt in a consistently structured way. The structure is composed of three parts from left to right: 

`program [option(s)] [argument(s)]`

A command does not need all three parts to be executed, which is what the straight brackets in that model structure mean; it just depends on the program used and what you want to do with it. But you must declare a program, at least. After that it may include option(s) only, argument(s) only, or both. We’ll have more examples of that later.

### Programs

Programs — also called “utilities”, and frequently called “commands” when run without options or arguments — are command-line application files. You invoke the program files to do an action by naming them in the first part of your command, using their abbreviations, which makes them easy to type. 

For example, the ‘list directory contents’ program is used in a command as `ls`, and the ‘change directory’ program is used as `cd`. As the `ls` program shows, the abbreviation isn’t always a clear match with the name, but most programs have obvious abbreviations like `cd` suggests so you commit them to memory quickly.

Programs are classified in three ways: 

* **Internal** (a.k.a. “builtin”) program files are those recognized and processed by the bash shell. These commands are generally represented by the core BSD UNIX command set for file handling. Common examples include `ls` and `cd` mentioned previously plus copy file(s) (`cp`), make directories (`mkdir`), move files (`mv`), open files and/or directories (`open`), remove directory entries (`rm`) and so on.
* **Included** programs are separate, executable program files, but still included and maintained in macOS (thanks, Apple) and can be run from Bash. Examples include [cURL](https://curl.haxx.se/) (`curl`) and [secure shell](http://www.openssh.com/) (`ssh`), among others. 
* **External** command-line programs are files that aren’t already installed on macOS. People who need these, such as for local website development, will install them directly (e.g. [Homebrew](http://brew.sh/) (`brew`), [Composer](https://getcomposer.org/) (`composer`), [Git](https://git-scm.com/) (`git`) and the like), or indirectly in relation to other installed software packages, like database servers.

Knowing the differene between _internal_ and _included_ programs is not particularly useful. Just think of both types as one group that’s installed on macOS and immediately available for use. That makes for a more useful comparison with _external_ programs, because you’ll need to install those before you can use them. In that case, cURL (again, already installed on macOS) is your friend, and software vendors (or other instructions you might follow) often provide the necessary `curl` commands to run if/when you want to install external programs.

### Options

A type of command parameter that enhances or alter a program’s default function in some way. General command options are single alphanumeric (upper- and lowercase) and special characters preceded by a hyphen (e.g. `-a`, `-l`, `-R`, `-@` and so on), each of which has its own influence on a command. Sometimes multiple options are used in a single command, in which case they’re collapsed together behind the hyphen (e.g. `-al`, or, equivalently, `-la`). 

The options available vary with the program you’re running. Some programs have many possible options, while others only have a few. You can learn what options exist for each command by using the _General Commands Manual_ (see section E.2.1).

Besides the options used with basic BSD commands, you’ll come across other types of options too, typically in relation to external/vendor command-line programs. For example the `mysql` command-line shell employs both general BSD command options and its own specific options for working with the MySQL server that are recognizable by leading double-hyphens.

**Arguments:** The other parameter type that comes at the end of a command. They are often folder/file paths relative to your working context and/or target location in the file tree. 

With respect to argument paths, here are two path notations that are useful to know:

* `~` (Equivalent to _/User/username_, your user directory.)
* `/` (Equivalent to the macOS disk root, which is useful to use when targeting a directory in a different branch of the file tree than the one you’re in.)

Working context and directory paths (argument parameters) will be demonstrated more in sections C and D.

## B) Working context (the key to success)

When you open Terminal’s console in a fresh window or tab, you should see something like below, with the date, time, Mac name, and username reflecting your situation:

```
Last login: Wed Sep 21 18:19:36 on ttys000
MacName:~ username$ 
```

Our focus is on the second line — ☆ **the command-line** ☆ — recognizable by the prompt, which is the entire string from `MacName` to `$`. The machine name will be whatever is provided in **System Preferences** > **Sharing**. The username will be your (administrator) account name created at the time you first setup your computer after buying it. 

The beginning (`MacName:`) and ending (`username$`) parts of the prompt are constants that never change, unless you go out of your way to change them.[^1] 

The middle part of the prompt, however, is a useful variable to be aware of; it reflects the directory your currently in — the working directory context — which in turn is in context of the file tree branch that directory is in.

Note that Terminal put you a directory notated by `~` by default. This is your user directory located at _/Users/username_ (i.e. `~` = `/Users/username`), a path relative to the macOS disk root. 

But you can’t always tell by the directory context alone where you are in the file tree because there are often many system directories with the same name both within and outside of a given file tree branch; names like _bin_, _etc_, _var_ and so forth. You may even create directories of the same name in different locations yourself. 

That’s where the ‘ping working directory’ (`pwd`) utility comes in. Run `pwd` on the command-line, and your path location will output relative to the disk root. If you do this when in your user directory, for example, you verify that `~` indeed matches as described above:

```
MacName:~ username$ pwd
/Users/username
```

The user’s directory notation is a nice convenience for a couple of reasons. For one thing it’s a single character that’s easy to see and use. It also eliminates what would otherwise be an unsightly redundancy of the username in the prompt. So whenever you see `~` in the prompt, it means the context is at your user directory.

Now you understand:

* how to identity the command-line and prompt
* how to read the prompt and use `pwd` to assess working context
* what `~` signifies (i.e. your user directory location), and 
* the structure of a command `program [option(s)] [argument(s)]`. 

Let’s take that newfound knowledge and round it out a little more by demonstrating working context in relation to file tree navigation using two basic commands: `ls` and `cd`.

**Note:** In the command-line example above and those that follow, I show the full prompt in addition to the commands. I do this to focus on the directory context, as recognized in the prompt, and what it looks like when it changes. But normally command-line instructions, like these and others you’ll come across, don’t show the prompt at all (ideally), or only the last character (`$`). The assumption is you can recognize the commands by their pattern and thus distinguish command-lines from the other console output. And you should be able to do that while remaining aware of what you’re working context is in you’re own Terminal console. The prompt and `pwd` utility help you as you go.

***

[1] Change your machine name might be reasonable if the name was really long and you wanted something shorter in your prompt. Likewise you can [change your username](https://support.apple.com/en-us/HT201548), if you really needed to, but this is not recommended unless you understand the ramifications it could have on any existing paths and links using it.)  

## C) List directory contents, `ls`

Say you wanted to see what was in your user directory, which is the working context of the default prompt discussed above. We do this easily by using the `ls` program. No options or arguments are necessary yet:

```
MachineName:~ username$ ls
```

That simple program-only command will output the names of all non-hidden files and folders inside your user directory. It does so because that’s the directory context you’re in (symbolized by `~`), and you’ve not indicated a different directory to execute the command on by any argument path. 

Now let’s say you want to display any hidden files in the user directory too. (The names of hidden files typically begin with a dot; e.g. _.DS_Store_, _.htaccess_, _.git_, _.gitignore_…) You could do that by adding the  `-a` option to the command:

```
MachineName:~ username$ ls -a
```

Finally, you could execute either of the previous two commands on a different directory than what your context is (i.e. without actually changing context to the target directory first). You do this by adding an argument to your command to say what the target directory is.

One directory down would just be the name of a child directory in your working directory:

```
MachineName:~ username$ ls -a Documents
```

But you could just as easily indicate a deeper level under the same working directory; for example, a directory called _abcGum_ in _Documents_:

```
MachineName:~ username$ ls -a Downloads/abcGum
```

Or you could use a path relative to the macOS disk (i.e. leading “/“) to indicate a folder in a different branch altogether:

```
MachineName:~ username$ ls -a /usr/local/etc
```

In all the above situations, your command-line context has not changed; you’re still working from your user directory, as indicated by the `~` in the command-line prompt. The only things changing are whether or not to show hidden files (via the `-a` _option_), and the directory you’re executing the command on (via the folder/path _argument_).

But you won’t always work from your user directory. Changing your working context from time to time can be productive, especially when file path arguements target more than a couple levels in the file tree. It’s impossible to remember where everything is in your file tree, after all. That’s where the `cd` command comes in.

## D) Change directory, `cd` 

The ‘change directory’ command is a general “builtin” command that does exactly what it implies. More specifically, it changes your working context from the directory your currently in to a new location you indicate via a path argument. There are no options to use with the `cd` command.

In a previous example, we ran a list command to display the contents of a child folder without changing the working context of the user directory. Here it is again:

```
MachineName:~ username$ ls -a Downloads/abcGum
```

As an alternative to targeting the _abcGum_ directory remotely as we did, we could first change our working context to that directory, then run the list command after we get there. This would require running two commands in series:

```
MachineName:~ username$ cd Downloads/abcGum
MachineName:abcGum username$ ls
```

Note how the directory context changed in the prompt after running the `cd` command. That’s visually letting you know that your working context is now in the _abcGum_ directory. The second command is simply `ls`, which outputs the child content of _abcGum_ (hidden files exluded).

You can just as easily change your working context back to the user directory:

```
MachineName:abcGum username$ cd ~
MachineName:~ username$
```

Or to jump from anywhere in the file tree to a deep level under the user’s directory:

```
MachineName:abcGum username$ cd ~/Documents/essays/2016
MachineName:2016 username$
```

Note how the path notation for the user directory, `~` (equivalent to _/Users/username_), can be used as the folder path argument or as part of the path.

Here are a couple more change directory tricks you’ll find useful at some point… This one takes you from wherever you are in the file tree (no matter how deep) and puts you directly at the disk root:

```
MachineName:~ username$ cd /
MachineName:/ username$
```

And this one moves you up one level to the respective parent directory:

```
MachineName:~ username$ cd ../
MachineName:Users username$
```

***

The takeaway from all the examples shown so far for, if you’ve not recognized it yet, is that your working context doesn’t have to be inside a given directory to run commands on it, a demonstrated with the `ls` commands in the previous section. As long as you know where you are in the file tree (and you do thanks to the context given in the prompt), you can use relative paths to get things done “remotely”, without moving around first. This can reduce the number of commands you have to run in a series, thus can be a time-saver.

On the other hand, you can easily change your working context at any time, as demonstrated with the `cd` commands. This is very useful if you’re more than a couple levels away from your target, or in a different branch of the file tree entirely. 

Ultimately, it makes no difference how you choose to use the command-line and navigate around. Do what makes the most sense. The two-step process of changing into the target location first then executing the process commands is probably wise for new users of Terminal, as it helps you get familiar with moving around your file tree and commit basic commands to memory. 

As you get more comfortable, you might start working more from a single directory (e.g. the default user directory) and launching commands remotely. But there’s no right or wrong way to do it, and some command-line “browsing” is expected no matter what skill level you attain. As long as you understand the context your working from, it’s all good.

## E) Tips for command-line productivity

Some fast-track tips as you head off into Terminal land.

### Errors: harmless typos versus actual “uh-ohs”

It’s very easy to make typographical and syntactical mistakes when working on the command-line; especially when you’re getting used to the working context and writing command parameters like long file paths. For the most part, the command-line is pretty forgiving. If you run a command with a mistake in the program or options part, the bash shell will simply return an error and a new prompt to start over again. 

For example, here’s a typo in the `ls` program part of the command:

```
MachineName:~ username$ lls -a
-bash: lls: command not found
MachineName:~ username$ 
```

As you can see from the helpful error line, the bash shell doesn’t recognize the indicated program, then gives you a new prompt to try again.

Here’s another faulty command, this time with the option, which doesn’t exist for the `ls` program:

```
MachineName:~ username$ ls -j
ls: illegal option -- j
usage: ls [-ABCFGHLOPRSTUWabcdefghiklmnopqrstuwx1] [file ...]
MachineName:~ username$
```
 
This time the shell not only shows you the error, but all the options that are permissable with the `ls` program. Nice!

There are many different types of error messages you might face when running commands, and external programs will have their own set of error messages too. Error messages are your friends. They help you when you make mistakes and get back on your feet.

However, the bash shell can’t read your mind. If you type a program or parameter that is not funcionally wrong but it’s not the right parameter you meant, the command will still run successfuly. This could be harmless or detrimental depending on what command it was. Some commands, like `mv` (which moves or renames a file), can be corrected by running another command to reverse the action (e.g. move the file back). But other commands, like `rm` (which removes/deletes a file), are permanent once they’ve been run (there’s no _Trash_ folder in the command-line).

Many system files (i.e. those needed to make your macOS operate correctly) and vendor configuration files (e.g. what might be needed for Apache web server or MySQL database server) are something to be careful of too. In this case, you have to be the machine’s root user (the administrator account) to make changes on such files. Those commands will require using `sudo` before the rest of the command, plus you’ll have to supply the root user password before the command will run. But that’s getting ahead of things for purposes here. You may read [Apple’s Command-line Administration Manual](https://www.apple.com/server/docs/Command_Line.pdf) about using `sudo`, and for all other advanced topics.    

The point here is, don’t be afraid of breaking anything. You won’t blow up your computer if you accidently run a faulty command. You just have to be careful about running syntactically correct commands that do what you didn’t intend. These kinds of mistakes generally happen when writing file paths, as already mentioned.  

At the same time, you don’t want to be making mistakes unnecessarily, they are annoying and cut into productivity. The whole point of the command-line, after all, is to do things faster that you would by click, click, clicking around in a graphical user interface. To counter attack your occasional typos and get back in the captain’s chair, you’ll want to learn some keyboard navigation tricks, and take advantage of re-using text strings from your console history. Both are remarkably easy to do.

### Command-line navigation

You will soon discover that you can’t insert the cursor in a command to fix any typos you happen to notice (like you can do easily in your text editor). This may lead you to think the only options are to use left and right arrow keys to traverse the command-line one character at a time, or run the broken command as an easier way to get a new prompt and start over again, which is a little reckless, if mostly harmless. 

Better is to learn a few navigation shortcuts in combination with the arrow keys so you can dance around the command-line like Ginger Rogers and Fred Astaire.

Try these:

* **Control**+**A** — Jump to beginning of line
* **Control**+**E** — Jump to end of line
* **Control**+**W** — Delete previous Word
* **Control**+**N** — Go to next Line
* **Control**+**P** — Go to previous Line
* **Control**+**U** — Delete everything on line left of cursor
* **Control**+**K** — Delete everything on line right of cursor

And this one works in relation to the previous two:

* **Control**+**Y** — Will paste at cursor any previously deleted content from a Conrol+U or Control+K action.

However, you don’t have to delete text just to paste it again. In that case, use the highlight-’n-drag method described next.

#### Reuse text strings to save time and avoid typos

You may find yourself having to run the same commands a lot, or parts of them. Server login sequences, database export/import routines, long file paths, ugly file names, and so on get tedious when you have to type them repeatedly. Plus typing more often than not just increases the odds of making mistakes. The good news is you don’t have to write things over and over. Just re-use any text string from your console history, even prior command-line strings. 

**It works like this:** Highlight the text (including any leading or trailing blank spaces you want to include) and hold down the mouse key or touch pad as you do. Then slightly drag downward until you see a “+” icon appear on it. Then release. 

The string that you highlighted will add to the end of command-line, no matter how far away you are from it. You don’t have to drag the highlighted string all the way to the command-line yourself, just until you see the “+” sign. Make sure you do this only when you’re ready for the string you need in your command, otherwise you’ll be using the shortcut keys talked about previously to edit your command.

### E.2) Tips for learning the command-line faster

The first questions you might have at this point are: _what are all the possible commands I can use?_, and _is there an index to see them?_ 

There are lots of resources for learning how to use and benefit from the command-line, but there’s lots of redundancy too. some help sources are better than others. Some want to really help you, while others just want your Here are some insights for beginning efforts and for slightly more experienced levels too when you get there.

#### E.2.1) General Commands Manuaul, `man`

MacOS’s internal commands are largely the same as the core set used by any [BSD system](http://www.bsd.org/), so you’ll find a lot of similarity even if you’re not looking at something specifically “Mac”. That said, use a macOS resource whenever you can for best fit. 

A good place to start is with the **BSD General Commands Manual**, which you can invoke right in your Terminal’s console. It works by using the `man` command followed by the name of the command you want to read about. 

For example, this would output the manual page for our old friend the `ls` program:

```
man ls
```

Manual pages display information (and there’s often a lot of it) in a consistent order. The top three sections will be the most useful to you:

1. Name (full name of the program utility)
2. Synopsis (the possible forms of command structure)
3. Description (explanations of the synopsis forms and all possible options)

Here’s the synopsis for the `ls` program:

```
SYNOPSIS
		ls [-ABCFGHLOPRSTUW@abcdefghiklmnopqrstuwx1] [file ...]
```
 
As you can see, the `ls` command has a single command form (obvious by a single line) and 38 possible options (yes, the `@` and number `1` are also valid options). That’s quite a lot of options. Each is described in the Description secton of the manual page. 

Use the `man` command to learn about any other command that’s native to macOS too. For example, following is the synopsis you get if you run the command, `man mv`:

```
SYNOPSIS
     mv [-f | -i | -n] [-v] source target
     mv [-f | -i | -n] [-v] source ... directory
```

You can even `man` the manual to learn how to use the `man` program:

```
man man
```

#### E.2.2) Command help (`help`, `-help`, `--help`, or `-h`)

The manual will always be your go-to for detailed help, but as you get more familiar with things and perhaps use more programs beyond the most basic ones, you may not need as much detail every time. 

In these cases, you can try running the program with one of the possible help parameters (`help`, `-help`, `--help`, or `-h`), which will return summaries of key help points, or won’t return anything at all if the command is too basic. But you may have to try different help parameters to find the right one to use respective to the program your interested in. Using the wrong parameter will just throw a command-line error and new prompt, so no harm done.

The _domain information groper_ program, `dig`, is one you may never end up using, but it provides a good demonstration here. You can pull up the manual page for `dig` and get the full documentation:

```
man dig
```

Or you can simply look at the concise summary of help that’s available for it, which in this case requires the `-h` option (a  fact learned from the manual page):

```
dig -h
```

The resulting help info is more than we want to bother showing here, but it’s less that what you get from the full manual page, and that’s the point — quick reference help.

Not all programs use the `-h` option, however, or when they do, the option may have a completely different function. For example there is no help option for the `ls` command (i.e. it’s the manual page or nothing), but there is a `-h` option for it having no relevance to command help, and it must be used in combination with the `-l` option (i.e., `-hl`) or it will function just like using `ls` by itself:

In short, use the manual pages first. But later as you try different programs, run through the help options noted above to see if there’s one that returns concise forms of help that you can use later for easier reference. It’s all about exploring, learning, and remembering where/how you can learn or get reminders in more concise fashion.  

#### E.2.3) All resources are not the same

Be careful about the resources you find online. Not all are accurate, or at the very least may be confusing. 

For example, here’s a relevant looking [A-Z Index of the Apple OS X command line](http://ss64.com/osx/). But, in the “L” section are two `ls` program alternatives for listing content in “long format” (i.e. full detail display without need for options) — `l` (exclude hidden files) and `ll` (include hidden files). But neither work on the command-line in macOS Sierra, nor have explanatory pages in the manual. So you might wonder why they were listed in a Mac commands list at all. (That would be a good question.)

The correct form for these commands are given in parenthetical reference in each case, which are simply the `ls` command with the appropriate options, respectively:

```
ls -l
ls -la
```

The point is, there are lots of discrepancies like that in resources you may find. Don’t let them trip you up. When in doubt, use the manual.

#### E.2.4) Different ways of doing things
 
It’s often the case there are multiple ways to do something from the command-line, and particularly as you start working with more external applications and your commands get more sophisticated.

One example is when editing files. There are some powerful command-line editing programs you can learn and use, like Vim and Emacs, where the console becomes your text editor. However, while these editors are powerful, they’re also a little advanced, as it requires learning their command and keyboard conventions before you can use them. 

You may find it a lot easier to simply open the GUI of another text editor on your machine by calling it directly from the command-line. This is easily done with the `open` command. It’s worthwhile to look at the manual page for the `open` program to see what the options do, but here’s a simple example… 

```
open -a TextMate /private/etc/apache2/httpd.conf
```
     

This will not be evident when starting out, of course, and you’ll use the first thing you learn. Generally the shorter the command needed to achieve something, the more desirable it is from an efficiency standpoint.

#### E.2.5) StackOverflow is your friend (if you don’t have an account)

[StackOverflow](http://stackoverflow.com/) is a treasure trove of command-line information straight from the trenches, and you don’t need an account there to use it. In fact, I recommend you do not create an account and try to engage there or the experiece will sour. Odds are great that any question you might need to ask has already been asked and answered by others, so just search the public knowledge base from outside and it’s a win-win for you.

StackOverflow’s value comes when you’re trying to do something on the command-line but don’t know what command you need to do it. The trick for finding command solutions is to do a search using “command” plus whatever other keywords adequately frame your known dilemma or objective. 

For example, if you want to know how to export a database from your MySQL server and save it as a file in your user directory. A search string like “[command export database mysql](http://stackoverflow.com/search?q=command+export+database+mysql)” gives a list of promising threads to investigate. 

In that search instance, you may or not find a command that addresses the “user directory” part of your objective too, but you’ll get the export part of your command for sure, which is the important part. The rest is just knowing your working context, as explained earlier in this doc, and/or using the right path argument to output the database file.  

Now go practice the command-line. It’s fun and therapeutic! You’ll be amazed how quickly you pick it up. 

 
