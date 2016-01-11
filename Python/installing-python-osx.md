### Installing Python 3:

- Navigate to `brew.sh` in a browser
- Press `Command+C` to copy the text it tells you to copy
- Press `Command+Space`, then type `terminal` and press enter
- Press `Command+V` to paste the copied text into the terminal window, and press enter
- Wait for it to finish, entering `y` at any prompts and entering the password you use to log into the computer when it asks
- After it's finished, enter the command `brew install python3`; again, enter `y` at any prompts
- Ignore anything it says about linking stuff; you don't need to do that
- Enter `python3` to run an interactive Python shell in your terminal
- Enter `print('Hello, world!')` to write and execute your first program!
- Enter `import this` to read and meditate on the Zen of Python
- When you're finished playing around, press `Control+D` on an empty line (or enter `exit()`) to exit the Python shell.


### Setting up a directory (or folder) for your projects:

- In a terminal, enter `cd ~`: this **c**hanges **d**irectory to your _home directory_ <sup>[1](#1)</sup>
- Enter `ls` to **l**i**s**t the directory contents: you should see your Desktop, Music, etc.
- Enter `mkdir dev`: this makes a new directory named `dev` relative to your current working directory (inside your home directory, in this case) <sup>[2](#2)</sup>
- Enter `cd dev` to move into the newly-created directory <sup>[3](#3)</sup>
- Enter `echo 'print("Hello, world!")' > helloworld.py` to put the given text into a file named `helloworld.py` inside your current directory
- Enter `python3 helloworld.py` to use the program `python3` to run the contents of the file <sup>[4](#4)</sup>
- Enter `open ~` to open your home directory using whatever program OS X deems appropriate—in this case, Finder, since it's a directory
- Click and drag the `dev` folder into Finder's side bar for easy access
- That's it! Between `cd`, `ls`, and `python3`, you should be able to navigate around and run Python programs, and with `open` you can open files and folders using non-terminal programs.

<sup><a name="1">[1]</a></sup> Your home directory is located at the path `/Users/whatever-your-username-is`. That's a lot to type, though, so `~` is a handy shortcut that "expands" to that path when you use it at a terminal prompt: `cd ~` does exactly the same thing as `cd /Users/username`, but you don't have to type or even know your username to use it. New terminals usually open up in your home directory by default, so you can probably skip this step in the future.

<sup><a name="2">[2]</a></sup> You could also enter `mkdir ~/dev` or `mkdir /Users/username/dev` to do the same thing using _absolute_ paths—note the leading slash, or the squiggle which expands to `/Users/username`, which begins with a slash. The directory called simply `/` is the "root" of the filesystem: `Users` is a directory just inside that root directory. If a path starts with a slash, it  is "absolute", and will refer to the same location no matter which directory you're in at the time. If a path does not start with a slash, it is "relative": the full path that will actually be used is formed by prefixing it with your working directory, which is wherever you last moved to using `cd`. Thus, the path `dev` evaluates to `/Users/username/dev` if you're in your home directory, but to `/Users/username/Desktop/dev` if you're in your Desktop folder. The path `/dev`, on the other hand, always refers to a directory called `dev` right under the root of the filesystem, no matter where you are at the time.

<sup><a name="3">[3]</a></sup> It's kind of embarrassing that there isn't a standard command to make a directory and move into it all at once, but you can make your own command to do that if you want to put in the effort.

<sup><a name="4">[4]</a></sup> Your terminal supports tab completion, which can save you a lot of typing: if you type just `python3 h` and press `Tab`, it should fill in the rest of the file name for you.


### Installing and using Sublime Text 3:

- In a terminal, enter `brew cask install sublime-text3`
- Press `Command+Space`, type `sublime`, and press enter
- Type `print('Hello from Sublime Text!')`, and press `Command+S`
- In the `Where` field, select `dev` (see previous instructions if it isn't there)
- In the `Save As` field, enter whatever name you like, but make sure it ends with `.py`
- Press `Command+B` to run the program directly from Sublime Text
- In a terminal, enter `cd ~/dev`, then type `python3`, a space, the first few letters of the file name, fill in the rest with `Tab`, and press enter to run the program
- That's it! I like to keep a terminal open next to my Sublime Text window and use `Command+Tab` to quickly switch to it, then press the up arrow and hit enter to repeat the last command.
