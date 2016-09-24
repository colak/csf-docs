# Using Terminal for the first time

Terminal is Apple’s command-line client. You can find it via Finder in **Applications** > **Utilities**. Drag the icon to your apps bar so it’s handy in the future.

Terminal can do a lot, most of which is out of scope here. But if there’s one thing to emphasize to someone first trying/learning the command-line, it’s this: Everything centers around working with folders and files on your machine. In turn, this means you need to know how to navigate to those places or execute commands on them, sometimes from a remote location — whether a parent directory, another connected drive, or a web server on the other side of the world. 

In that light, this document focuses on the working context of using the command-line. We’ll demonstrate the concept of context by using two common commands: `ls` and `cd`, and you’ll be on your own from there. Once you’re familiar with command-line anatomy and context, there’s no holding you back.

## A) Command-line anatomy

It’s useful to know how to talk about this stuff so other people understand you when you have questions, or when you’re doing more research on your own. We’ll talk about command-line anatomy in relation to Terminal.

**Command-line interface (CLI):** When you open Terminal, you’re looking at the command-line interface. It’s a simple Mac application window that you can resize to your liking, change the color of (Terminal > Preferences… > Profile), and open multiple tabs in. Tabs are wonderful! 

**Console:** The working area of the CLI (or of each tab if you have multiple tabs open). It’s here where you run commands and read the real-time output you generate.

**Command-line:** A special line in the console where you add and run commands. The command-line is easily identified by the leading command-line **prompt**. The active command-line will always be at the bottom of console output, waiting for your next orders.

**Prompt:** The opening string on the command-line that ends with a `$`. The prompt serves to show where to add your commands and what context you’re in when working. The prompts of command-lines already run in console history will appear with a leading straight bracket.

**Command:** Imagine yourself as an army general. Commands are the orders you’re giving to your machine, written on the command-line in a structured way. The structure is composed of three parts from left to right: _program, _option(s)_, and _argument(s)_. A command does not need all three parts to be executed, but you must declare a program, at least. After that it may include options, arguments, or both. (Note: programs are often called “commands”, and options and arguments are two different types of command _parameters_.) 

**Programs:** Command-line programs (a.k.a. “commands”) are of three different classifications: _internal_, _included_, and _external_. 

* Internal — programs are recognized and processed by Terminal itself (i.e. not dependent on included or external program files). These are generally represented by the core Linux command set for file/folder handling. Examples, which you will use most of the time, include `cd`, `ls`, `mkdir`, `cp`, `mv`, and `rm`.
* Included — programs are Apple extensions to the core command set; i.e. separate program files, but included with macOS. Examples include `curl`, `gzip`, `screencapture`, `say`, `ssh` and many others. 
* External — programs are files external to the operating system. You may install these directly, for example, for development reasons, or indirectly in relation to associated third-party software. Either way, once installed, they can be used on the command-line. Examples include `brew`, `composer`, `git`, `grunt`, `mysql`, `svn` and so forth.

The [A-Z listing of macOS command-line programs](http://ss64.com/osx/) is reasonably complete, but seems to lump _internal_ and _included_ types together. Also, not all possible _included_ programs are listed (e.g. `ssh` is missing).

**Options:** A type of command parameters than can be added to to enhance or alter the program’s function in some way. Options are single-letter characters — upper- or lowercase — preceded by a hyphen (e.g. `-a`, `-r`, `-R`, `-v`, `-V` and so on), each of which has its own influence on a command. Multiple options can be added in series, in which case they’re collapsed together behind a single hyphen (e.g. `-fR`). 

The options available to you are respective to the command in question. And while there are lots of web resources to learn about commands, such as the A-Z list linked above, the `man` command is convenient for learning about options. For example, if you wanted to know what options apply to the ‘list directory contents’ program (`ls`), then run `man ls` on the command-line and you get that page of the _manual_ showing all the applicable options and what they do. 

Don’t dive headlong into reading about commands and options with no objective to apply the knowledge. You’ll just forget everything you read. Instead, study programs and options when you actually need to use them, like when following instructions to setup and AMP stack. That gives you immediate context, plus an opportunity to read more about those particular commands. 

Last point about options… External programs will often introduce  their own option sets too. For example `mysql` employs both core Linux options as described here, and options specific to `mysql` commands alone, which are recognizable by double-hyphens.

**Arguments:** These are the other parameter type, which come at the end of a command. They are often folder/file paths relative to what your context in the file tree is, as the command prompt will make clear. In that respect, following are two path constructs that are very good to know:

* `~` (is equivalent to _/User/username_, you user directory)
* `/` (is equivalent to the OS disk root)

File tree paths will be demonstrated more in later examples.

## B) Command-line context (the key to success)

When you open Terminal’s console in a fresh window or tab, you should see something like below, where the date, time, and names reflecting your situation:

```
Last login: Wed Sep 21 18:19:36 on ttys000
MacName:~ username$ 
```

For purposes here, we’re concerned with the second line — the command-line — recognizable by the prompt, which is the entire string from `MacName` to `$`. 

The “MacName” will be whatever is provided in **System Preferences** > **Sharing**. The “username” will be your (administrator) account name created at the time you first setup your machine after buying it.

The command-line is where you execute commands by typing or pasting them immediately after the `$` and hitting Enter. 

The prompt holds the key to knowing what your context in the file tree is at any time, so let’s dissect it… The Mac’s name and colon (`MacName:`) and your “username” with dollar sign (`username$`) are constants at the beginning and end of the prompt, respectively. They never change, unless you go out of your way to change them.[^1]

The middle part of the prompt, however, changes according to the directory your in, the context by which you write commands. This is very useful to realize, because it helps you write argument paths in your commands correctly, respective to where you are in the file tree. 

By default, Terminal puts you in context of your “username” directory located at _/Users/username_ (a path relative to the macOS root). That’s exactly the context in the prompt above, but it’s using a special notation, `~`, to indicate the location (i.e. `~` = `/Users/username`). 

The user’s directory notation is a nice convenience for a couple of reasons. For one thing it’s a single character that’s easy to see and use. It also eliminates what would otherwise be an unsightly redundancy of the username in the prompt. So whenever you see `~` in the prompt, it means the context is at the “username” directory.

Now that you understand the command-line and prompt, how to read the prompt for working context, what the prompt looks like for your user directory (the default prompt when starting a new Terminal console), and the structure of a command, let’s demonstrate working context a bit more using two basic commands: `ls` and `cd`.

**Note:** In the command-line examples that follow, we’ll be showing the full prompt in the command-line, in addition to the command itself. We do this only to put focus on looking at how the working context changes in the prompt when it does. Normally when writing instructions you would not include the prompt for simplicity. Read more about this at end of document.

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

## E) Last tips for leaving the nest

A few offbeat survival tips as you head off into Terminal land.

### Fewer mistakes, faster commands, more productivity

It’s very easy to make typographical and syntactical mistakes when working on the command-line; especially when you’re getting used to the working context and writing command parameters. While such mistakes never hurt anything if you accidently run a faulty command, they are annoying and cut into productivity. The whole point of the command-line, after all, is to do things faster that you would by click, click, clicking around in a graphical user interface. To counter attack your occasional typos and get back in the captain’s chair, you’ll want to learn some keyboard navigation tricks, and take advantage of re-using text strings from your console history. Both are remarkably easy to do.

#### Quickly navigate the console and command-line

You will quickly discover that you can’t insert the cursor in a command to fix any typos you happen to notice (like you could easily do in your text editor). This may lead you to think the only options are to use left and right arrow keys to traverse back and forth across the command-line one character at a time, or run the broken command as an easier way to get a new prompt and start over again, which is a little reckless. 

Better is to learn a few navigation shortcuts to use in combination with the arrow keys so you can dance around the command-line like Fred Astaire.

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

However, you don’t have to delete text just to paste it again, in that case, use the highlight-’n-drag method described next.

#### Reuse text strings to save time and avoid typos

You’ll often find yourself needing to run the same commands a lot, or parts of them. Server login sequences, database export/import routines, long file paths, ugly file names, and so on get tedious when you have to type them repeatedly. Plus typing more often than not just increases the odds of making mistakes.

The good news is you don’t have to write things over and over. Just re-use whatever string already worked from your console history. 

**It works like this:** Highlight the text (including any blank spaces you want to include, which can be useful) and hold-down the mouse key or touch pad as you do. Then slightly drag downward until you see a “+” icon appear on it. Then release. 

The string that you highlighted will add to the end of command-line, no matter how far away you are from it. In other words, you don’t have to drag the highlighted string all the way to the command-line, just until you see the “+” sign. Make sure you do this only when you’re ready for the string you need in your command, otherwise you’ll be using the shortcut keys talked about previously to edit it.

### Command-line instructions anywhere else

The command-line examples we used in earlier sections of this documentation were written to show the full command prompt each time, in addition to the actual commands being run. That’s because the objective was to show how the context changed in the prompt when you changed directory locations. But now that you know about working context and how that context reflects in the prompt, we don’t have to be so verbose anymore. In fact, you won’t see instructional commands written like that anywhere else because it’s unnecessary. 

Instead, you’re likely to see instructions that either use the last character of the prompt only (`$`), distinguishing the command-line from output lines…

```
$ ls /
Applications			Volumes				net
Developer			bin				private
Library				cores				sbin
Network				dev				tmp
System				etc				traces.log
usr        Users				var
```

Or none of the prompt will be used at all, as it’s assumed the reader will recognize the program part of the command and thus know it’s on the command-line. 

```
ls ~/Downloads
ied_plugin_composer-master
ied_plugin_composer_v1.1.0.txt
kuo_github_viewer_v0.2_zip.txt
magic.csv
rah_wrach-master
smd_bio-master
smd_bio_v0.41.txt
smd_macro_v0.30.txt
spf_writer_v0.1.txt
wet-maintenance
```

The latter is the ideal way of writing instructions, because it makes it easy for readers to copy the command as needed, without getting confused by the terminal prompt character.

Now go practice the command-line. It’s fun and therapeutic! 

 
