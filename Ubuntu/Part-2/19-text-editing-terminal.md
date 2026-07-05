# Chapter 19: Text Editing, Shell Customization, and Your First Scripts

## Making the Terminal Your Own

Until now, you've been using the terminal as it comes — default settings, default prompts, typing commands one at a time. That's perfectly fine for learning. But the real power of Linux emerges when you customize the shell to match your workflow and begin automating repetitive tasks with scripts.

This chapter has three parts:
1. **Terminal text editors** — how to create and edit files without leaving the command line
2. **Shell customization** — aliases, environment variables, and configuration files
3. **Shell scripting basics** — writing your first automated scripts

By the end, you'll have a personalized terminal environment and the ability to write scripts that save you time every day.

## Part 1: Terminal Text Editors

There are times when you need to edit a file from the terminal: configuration files, scripts, system settings. You could open a graphical editor like GNOME Text Editor, but terminal-based editors are faster, work over SSH, and function even when the graphical interface isn't available.

Ubuntu 26.04 includes two terminal text editors by default: **nano** and **vim-tiny** (a minimal Vim). Nano is the system's default editor — it's the one that opens when a command like `visudo` or `crontab -e` launches an editor.

### nano: The Beginner-Friendly Editor

nano is designed to be simple. It shows its key bindings at the bottom of the screen, requires no memorization of modes, and behaves like a basic text editor.

**Opening a file:**

nano myfile.txt

If myfile.txt doesn't exist, nano creates it.

The nano interface:

  GNU nano 8.2              myfile.txt

This is where your text appears.
The cursor blinks right here_









^G Help     ^O Write Out ^W Where Is  ^K Cut      ^T Execute
^X Exit     ^R Read File ^\ Replace   ^U Paste    ^J Justify

The bottom two lines show available commands. The ^ symbol means Ctrl — so ^X means "hold Ctrl and press X."

Essential nano shortcuts:
Shortcut	Action
Ctrl+X	Exit (prompts to save if changes were made)
Ctrl+O	Save (write out) — press Enter to confirm filename
Ctrl+K	Cut the current line (also works for selection)
Ctrl+U	Paste (uncut) the cut text
Ctrl+W	Search for text
Ctrl+\	Find and replace
Ctrl+G	Help screen
Ctrl+A	Move cursor to beginning of line
Ctrl+E	Move cursor to end of line
Alt+A	Start selection (then use arrows to select)
Ctrl+_	Go to a specific line number
Ctrl+C	Show current cursor position (line, column)

Editing workflow:

    nano filename — open or create the file
    Type or edit text — it works like a normal text editor
    Ctrl+O then Enter — save your changes
    Ctrl+X — exit

Opening a file at a specific line:

nano +42 logfile.txt

Opens logfile.txt with the cursor on line 42. Useful when an error message tells you which line has a problem.

Enabling line numbers:

nano -l script.sh

Or press Alt+N while nano is open to toggle line numbers.

    💡 nano Is Your Starting Point

    For this entire manual, nano is the recommended editor. It's installed everywhere, requires no learning curve, and shows its shortcuts on screen. When tutorials say "open this file in your preferred editor," use nano.

vim: The Powerful (But Steep) Editor

Vim (Vi IMproved) is legendary among Linux users — loved by power users, feared by beginners. It operates in modes, which is the main source of confusion. Unlike nano, where you just type, Vim distinguishes between:

    Normal mode — for navigation and commands (default when you open a file)
    Insert mode — for typing text
    Command-line mode — for saving, quitting, searching

Why learn Vim at all? Because it's installed on virtually every Linux system in existence. If you ever manage a remote server over SSH, connect to a minimal rescue environment, or use a container, nano may not be available — but vi almost always will be.

Ubuntu 26.04 includes vim-tiny by default. To get the full Vim experience:

sudo apt install vim

Opening a file:

vim myfile.txt

The mode system — this is the key concept:

When you open a file in Vim, you're in Normal mode. You cannot type text yet. To type, you must enter Insert mode by pressing i. To return to Normal mode, press Esc.

This sounds confusing, and it is at first. But the benefit is enormous: in Normal mode, every key is a command. Want to delete a line? Press dd. Delete 5 lines? 5dd. Copy a line? yy. Paste? p. These commands are fast — no Ctrl combinations needed.

Minimal survival guide:
Key(s)	Mode	Action
i	Normal → Insert	Start inserting text at cursor
a	Normal → Insert	Start inserting text after cursor
o	Normal → Insert	Open a new line below and start inserting
Esc	Insert → Normal	Return to Normal mode
:w	Normal → Command	Save (write)
:q	Normal → Command	Quit
:wq	Normal → Command	Save and quit
:q!	Normal → Command	Quit without saving (force)
dd	Normal	Delete the current line
yy	Normal	Copy (yank) the current line
p	Normal	Paste below the cursor
u	Normal	Undo
Ctrl+R	Normal	Redo
/word	Normal	Search for "word" (press n for next match)
gg	Normal	Go to top of file
G	Normal	Go to bottom of file
:set number	Normal	Show line numbers

Survival workflow:

    vim filename — open the file
    Press i — enter Insert mode
    Type or edit text
    Press Esc — return to Normal mode
    Type :wq — save and quit

If you get stuck: press Esc, then :q! to force quit without saving.

    💡 The Vim Learning Curve

    Don't try to learn all of Vim at once. Learn the survival keys above — they're enough to edit any file. Over time, you'll naturally pick up more commands as you use Vim. Many users run vimtutor (an interactive Vim tutorial) to practice:

    vimtutor

    This launches a 30-minute guided tutorial built into Vim. It's the fastest way to build muscle memory.

Choosing Your Default Editor

Some system commands (like visudo, crontab -e, and git commit) automatically launch a text editor. By default, Ubuntu 26.04 uses nano. You can change this:

select-editor

This presents a menu of installed editors. Choose one, and system commands will use it.

Alternatively, set the $EDITOR environment variable (covered later in this chapter):

export EDITOR=nano

Or, for Vim:

export EDITOR=vim

Add whichever you choose to your ~/.bashrc (also covered later) to make it permanent.
Other Notable Editors
Editor	Description	Install
Neovim	Modernized Vim with better defaults, Lua scripting	sudo apt install neovim
micro	Terminal editor with modern keybindings (Ctrl+C, Ctrl+V, Ctrl+S)	Download from micro-editor.github.io
emacs	The other legendary editor — extensible, powerful, huge	sudo apt install emacs
gedit / Text Editor	GNOME's graphical editor — not terminal-based but simple	Pre-installed on Ubuntu

You don't need any of these right now. Start with nano, learn Vim basics when you're ready, and explore others as your needs evolve.
Part 2: Shell Customization

The shell is the program that interprets your terminal commands. On Ubuntu 26.04, the default shell is Bash (GNU Bourne-Again Shell). Every time you open a terminal, Bash starts up, reads its configuration files, and presents you with a prompt.
Configuration Files

Bash reads several configuration files when it starts:
File	When It's Read	What Goes In It
/etc/bash.bashrc	Every interactive shell (system-wide)	System defaults — don't edit this
/etc/profile	Every login shell (system-wide)	System-wide login setup — don't edit this
~/.bashrc	Every interactive shell (your user)	Your customizations go here
~/.bash_profile	Login shells only (your user)	Login-specific settings
~/.profile	Login shells if ~/.bash_profile doesn't exist (your user)	Alternative login settings

    💡 The File That Matters: ~/.bashrc

    For 99% of your customization needs, ~/.bashrc is the file to edit. It's read every time you open a new terminal window. If you make changes to it, reload them immediately with:

    source ~/.bashrc

    Or the shorter equivalent:

    . ~/.bashrc

    Yes, a single dot is an alias for source — it's a Bash built-in command.

Aliases: Shortcuts for Common Commands

An alias is a shortcut — a word that expands into a longer command. You define them in ~/.bashrc.

Basic alias syntax:

alias shortname='long command here'

Open your ~/.bashrc in nano:

nano ~/.bashrc

Scroll to the bottom and add your aliases. Here are practical examples:

# Navigation shortcuts
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'

# Long-form shortcuts
alias update='sudo apt update && sudo apt upgrade -y'
alias cleanup='sudo apt autoremove -y && sudo apt autoclean'
alias ports='sudo netstat -tulpn'

# Safer defaults
alias rm='rm -i'          # Confirm before deleting
alias cp='cp -i'          # Confirm before overwriting
alias mv='mv -i'          # Confirm before overwriting

# Better ls output
alias ll='ls -lah'        # Long format, all files, human-readable sizes
alias la='ls -A'          # All files except . and ..
alias l='ls -CF'          # Column format

# Quick system info
alias myip='curl ifconfig.me'
alias meminfo='free -h'
alias diskinfo='df -h'

After adding aliases, reload:

source ~/.bashrc

Now typing update runs the full system update. Typing ll shows a detailed file listing. Typing .. moves up one directory.

Viewing your current aliases:

alias

Lists all active aliases.

Removing an alias temporarily:

unalias ll

Removes the alias for the current session only. It returns when you open a new terminal (because it's defined in ~/.bashrc).

    ⚠️ Aliases and Safety

    The alias rm='rm -i' alias makes rm ask for confirmation before deleting. This is a good safety net — but don't rely on it exclusively. Scripts (covered below) don't use your interactive aliases. Always think before pressing Enter on a rm command.

Environment Variables

Environment variables are named values that affect how programs behave. They're like settings — the shell and other programs read them to determine configuration.

Viewing all environment variables:

env

Viewing a specific variable:

echo $HOME
echo $USER
echo $PATH
echo $SHELL

Common environment variables:
Variable	Contains
$HOME	Your home directory path (/home/username)
$USER	Your username
$SHELL	Path to your shell (/bin/bash)
$PATH	Directories searched for executable commands
$PWD	Current working directory
$EDITOR	Your preferred text editor
$LANG	System language and locale
$TERM	Terminal type
$HOSTNAME	Your computer's name

Setting an environment variable temporarily:

export EDITOR=nano

This sets $EDITOR to nano for the current terminal session. When you close the terminal, it's gone.

Setting an environment variable permanently:

Add the export line to ~/.bashrc:

export EDITOR=nano
export VISUAL=nano

Then source ~/.bashrc.
The $PATH Variable — How Commands Are Found

When you type a command like ls, the shell needs to find the ls program on disk. It searches through the directories listed in $PATH, in order:

echo $PATH

Output:

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games

Each directory is separated by a colon. The shell checks /usr/local/sbin first, then /usr/local/bin, and so on. When it finds ls in /usr/bin/ls, it executes it.

Adding a directory to your PATH:

If you create a ~/bin directory for your personal scripts (recommended in Part 3), add it to your PATH:

mkdir -p ~/bin
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

Now any executable script you place in ~/bin/ can be run by name from anywhere — no need to type the full path.

Checking where a command lives:

which python3

Output:

/usr/bin/python3

type ll

Output:

ll is aliased to `ls -lah'

Customizing Your Prompt

The terminal prompt (the text that appears before your cursor) is controlled by the $PS1 variable. The default Ubuntu prompt looks something like:

username@hostname:~/Documents$

You can customize it. Add to ~/.bashrc:

# Minimal prompt — just the current directory
export PS1='\w\$ '

# Show username, hostname, and directory
export PS1='\u@\h:\w\$ '

# Colored prompt with time
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

Prompt special codes:
Code	Displays
\u	Username
\h	Hostname (short)
\H	Hostname (full)
\w	Current directory (full path)
\W	Current directory (just the folder name)
\$	# for root, $ for regular users
\t	Current time (24-hour)
\d	Current date
\[\033[01;32m\]	Start green text
\[\033[00m\]	Reset to default color

    💡 Keep Your Prompt Simple

    A clean, informative prompt helps you work faster. Many people like seeing the current directory and git branch (if in a repository). Don't overdo it — a 200-character prompt wastes screen space. The default Ubuntu prompt is already well-designed.

Part 3: Shell Scripting Basics

A shell script is a text file containing a sequence of commands that Bash executes in order. Instead of typing the same commands repeatedly, you write them once in a script and run the script whenever you need them.
Your First Script

Let's create a simple script step by step.

Step 1: Create the file.

nano ~/hello.sh

Step 2: Write the script.

Type the following exactly:

#!/bin/bash
# This is my first script

echo "Hello, $USER!"
echo "Today is $(date)"
echo "You are in: $PWD"
echo "Your system has been running for:"
uptime

Save (Ctrl+O, Enter) and exit (Ctrl+X).

Step 3: Make it executable.

chmod +x ~/hello.sh

Step 4: Run it.

~/hello.sh

Output:

Hello, john!
Today is Sat Jul 4 14:32:07 UTC 2026
You are in: /home/john
Your system has been running for:
 14:32:07 up  3:25,  2 users,  load average: 0.42, 0.58, 0.61

Congratulations — you've written and executed your first shell script!
Anatomy of a Script

Line 1: The Shebang

#!/bin/bash

The #! (called a "shebang" or "hashbang") tells the system which interpreter to use. #!/bin/bash means "run this script with Bash." Without this line, the script might run in a different shell with different behavior.

Every script you write should start with a shebang. Other common shebangs:
Shebang	Interpreter
#!/bin/bash	Bash shell
#!/bin/sh	POSIX shell (more portable, fewer features)
#!/usr/bin/env python3	Python 3
#!/usr/bin/env perl	Perl

Line 2: Comments

# This is my first script

Anything after # is a comment — ignored by the shell, meant for humans. Use comments generously to explain what your script does and why. Future you will thank present you.

Lines 3+: Commands

Each line is a command, executed in order. You can use any command you've learned: echo, date, uptime, ls, grep, apt, anything.
Variables in Scripts

Variables store values for later use. No spaces around the = sign:

#!/bin/bash

name="World"
count=5
price=9.99

echo "Hello, $name!"
echo "Count is: $count"
echo "Price: $price"

Important rules:

    No spaces around = — name="World" is correct, name = "World" is an error
    Use $ to read a variable — echo $name
    Double quotes allow variable expansion — "$name" becomes World
    Single quotes prevent expansion — '$name' stays literally $name

name="Alice"
echo "Hello, $name"     # Output: Hello, Alice
echo 'Hello, $name'     # Output: Hello, $name

Command substitution — capturing command output:

current_date=$(date +%Y-%m-%d)
echo "Today's date is $current_date"

The $(...) syntax runs the command inside and captures its output. Older scripts use backticks (`date`), but $() is preferred — it's nestable and more readable.

Reading user input:

#!/bin/bash

echo "What is your name?"
read username
echo "Hello, $username! Nice to meet you."

read pauses the script and waits for the user to type a line of input.
Command-Line Arguments

Scripts can accept arguments when you run them:

~/greet.sh Alice 25

Inside the script, these are accessed as:
Variable	Contains
$0	The script's name
$1	First argument
$2	Second argument
$#	Number of arguments
$@	All arguments (individually quoted)
$$	The script's process ID

Example:

#!/bin/bash
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Total arguments: $#"
echo "All arguments: $@"

Run it:

~/args.sh hello world test

Output:

Script name: /home/john/args.sh
First argument: hello
Second argument: world
Total arguments: 3
All arguments: hello world test

Conditionals: Making Decisions

Scripts need to make decisions based on conditions — "if this file exists, do X; otherwise, do Y."

Basic if statement:

#!/bin/bash

if [ -f /etc/hostname ]; then
    echo "Hostname file exists!"
    cat /etc/hostname
else
    echo "Hostname file not found."
fi

The [ -f /etc/hostname ] is a test condition. The spaces around the brackets are mandatory — [ is actually a command, and ] is its argument terminator.

Common test conditions:
Test	True When
[ -f file ]	File exists and is a regular file
[ -d dir ]	Directory exists
[ -e path ]	Path exists (file or directory)
[ -r file ]	File is readable
[ -w file ]	File is writable
[ -x file ]	File is executable
[ -z "$var" ]	Variable is empty
[ -n "$var" ]	Variable is not empty
[ "$a" = "$b" ]	String a equals string b
[ "$a" != "$b" ]	String a does not equal string b
[ "$a" -eq "$b" ]	Integer a equals integer b
[ "$a" -lt "$b" ]	Integer a is less than b
[ "$a" -gt "$b" ]	Integer a is greater than b

Elif (else-if):

#!/bin/bash

count=$1

if [ "$count" -gt 100 ]; then
    echo "Large number"
elif [ "$count" -gt 10 ]; then
    echo "Medium number"
else
    echo "Small number"
fi

    💡 Always Quote Variables in Tests

    Use [ "$var" = "value" ] (with quotes), not [ $var = value ]. If $var is empty, unquoted [ $var = value ] becomes [ = value ] — a syntax error. Quoted [ "$var" = "value" ] becomes [ "" = "value" ] — which works correctly.

Loops: Repeating Actions

For loop — iterate over a list:

#!/bin/bash

for file in *.txt; do
    echo "Found text file: $file"
done

This loops through every .txt file in the current directory.

For loop with a range:

for i in {1..5}; do
    echo "Iteration $i"
done

Output:

Iteration 1
Iteration 2
Iteration 3
Iteration 4
Iteration 5

C-style for loop:

for ((i=0; i<10; i++)); do
    echo "Number: $i"
done

While loop — repeat until a condition is false:

#!/bin/bash

count=1

while [ $count -le 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done

Output:

Count: 1
Count: 2
Count: 3
Count: 4
Count: 5

The $(( )) syntax performs arithmetic — count=$((count + 1)) increments count by 1.

Looping through command output:

#!/bin/bash

# Process each line of a file
while IFS= read -r line; do
    echo "Processing: $line"
done < /etc/passwd

This reads /etc/passwd line by line. The IFS= read -r pattern preserves whitespace and special characters — it's the safe way to read lines in Bash.
Functions: Reusable Code Blocks

Functions let you define a block of code once and call it multiple times:

#!/bin/bash

greet() {
    echo "Hello, $1!"
    echo "Welcome to $HOSTNAME."
}

greet Alice
greet Bob
greet Charlie

Function arguments work just like script arguments — $1, $2, $@ refer to the arguments passed to the function, not the script.

A practical function — checking if a package is installed:

#!/bin/bash

check_installed() {
    if dpkg -s "$1" >/dev/null 2>&1; then
        echo "$1 is installed"
    else
        echo "$1 is NOT installed"
    fi
}

check_installed htop
check_installed vlc
check_installed nginx

The >/dev/null 2>&1 redirects both stdout and stderr to nowhere — we don't care about the output, only the exit code (success/failure).
Exit Codes

Every command returns an exit code — a number indicating success or failure:

    0 = success
    Any non-zero number = failure (different codes for different errors)

Check the last command's exit code:

ls /nonexistent
echo $?

Output:

2

Exit code 2 means "no such file or directory." After a successful command:

ls /home
echo $?

Output:

0

In scripts, you can set your own exit codes:

#!/bin/bash

if [ -z "$1" ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

if [ ! -f "$1" ]; then
    echo "Error: File '$1' not found"
    exit 2
fi

echo "File found: $1"
exit 0

Exit codes let other scripts and tools check whether your script succeeded. The && and || operators use them:

mkdir /tmp/testdir && echo "Created!" || echo "Failed!"

If mkdir succeeds (exit code 0), echo "Created!" runs. If it fails (non-zero), echo "Failed!" runs.
A Practical Script: System Report

Let's combine everything into a useful script you'll actually want to keep:

#!/bin/bash
# system-report.sh — Generates a quick system health report

echo "========================================"
echo "  System Health Report"
echo "  Generated: $(date)"
echo "========================================"
echo ""

echo "--- HOSTNAME ---"
hostname
echo ""

echo "--- UPTIME ---"
uptime
echo ""

echo "--- DISK USAGE ---"
df -h / /home 2>/dev/null
echo ""

echo "--- MEMORY ---"
free -h
echo ""

echo "--- TOP 5 CPU CONSUMERS ---"
ps aux --sort=-%cpu | head -6
echo ""

echo "--- TOP 5 MEMORY CONSUMERS ---"
ps aux --sort=-%mem | head -6
echo ""

echo "--- FAILED SERVICES ---"
systemctl --failed
echo ""

echo "--- RECENT ERRORS (last 10) ---"
journalctl -p err -b --no-pager | tail -10
echo ""

echo "========================================"
echo "  Report complete."
echo "========================================"

Save this as ~/bin/system-report.sh, make it executable:

chmod +x ~/bin/system-report.sh

(If you set up ~/bin in your PATH earlier, you can now run system-report.sh from anywhere.)

Run it:

system-report.sh

This script combines commands from Chapters 16 and 18 — disk, memory, processes, services, and logs — into a single convenient report. You could even schedule it to run automatically (covered in Chapter 27, Advanced Scripting).
Another Practical Script: Bulk File Rename

#!/bin/bash
# rename-backup.sh — Adds a date stamp to files

if [ -z "$1" ]; then
    echo "Usage: $0 <file>"
    echo "Example: $0 document.txt"
    exit 1
fi

if [ ! -f "$1" ]; then
    echo "Error: '$1' is not a file."
    exit 2
fi

date_stamp=$(date +%Y%m%d)
filename="$1"
extension="${filename##*.}"
basename="${filename%.*}"

new_name="${basename}_${date_stamp}.${extension}"

cp "$filename" "$new_name"
echo "Copied: $filename → $new_name"

This script takes a filename, adds a date stamp, and saves a copy. Running rename-backup.sh report.pdf creates report_20260704.pdf.

The ${filename##*.} and ${filename%.*} are parameter expansion — Bash's built-in string manipulation. They extract the extension and basename respectively, without needing external commands.
Organizing Your Scripts

As you write more scripts, keep them organized:

~/bin/
├── system-report.sh     # System health report
├── rename-backup.sh     # Add date stamps to files
├── update-all.sh        # Update APT + Snap + Flatpak
├── cleanup-snaps.sh     # Remove old Snap versions
└── backup-home.sh       # (Chapter 27)

If ~/bin is in your PATH (as set up earlier), all these scripts are available by name from any directory.

Making scripts runnable without the .sh extension:

Many Linux users drop the .sh extension entirely — the shebang inside the file tells the system how to run it. So ~/bin/system-report (no extension) works just as well as ~/bin/system-report.sh. This is a matter of preference.
Script Safety Checklist

Before running a script (especially one you found online), check:

    Read it first. Open the script in nano and scan for suspicious commands — especially rm -rf, curl | bash, or unfamiliar URLs.
    Check the shebang. #!/bin/bash is expected. Anything else might be running a different interpreter.
    Look for sudo. Scripts that require root should be examined carefully.
    Watch for destructive commands. rm, dd, mkfs, fdisk — these can destroy data.
    Test on dummy data first. If a script renames files, test it on copies, not originals.

    ⚠️ Never Run Scripts You Don't Understand

    The command curl -s https://example.com/script.sh | bash downloads and executes a script in one step — without letting you read it first. This is extremely dangerous. Always download the script, read it, then run it:

    # Safe approach
    curl -O https://example.com/script.sh
    nano script.sh      # Read it!
    chmod +x script.sh
    ./script.sh

Shell Scripting Cheat Sheet
Concept	Syntax	Example
Shebang	#!/bin/bash	First line of every script
Comment	# text	# This is a comment
Variable	name="value"	greeting="Hello"
Read variable	$name	echo $greeting
Command substitution	$(command)	today=$(date +%A)
Read input	read var	read username
Script argument	$1, $2, ...	echo $1
Argument count	$#	if [ $# -eq 0 ]
All arguments	$@	for arg in "$@"
If/else	if [ test ]; then ... fi	See examples above
For loop	for x in list; do ... done	for f in *.txt; do
While loop	while [ test ]; do ... done	while [ $i -lt 10 ]
Function	name() { ... }	greet() { echo "Hi"; }
Arithmetic	$(( expr ))	sum=$((a + b))
Exit code	exit N	exit 0 (success)
Last exit code	$?	if [ $? -eq 0 ]
And / Or	&& / `	
What's Next

You now have the tools to edit files, customize your shell environment, and write scripts that automate repetitive tasks. This chapter brought together everything from Chapters 13–18 — commands, piping, permissions, package management, and system monitoring — into reusable scripts.

In Chapter 20, we'll cover searching and finding files in depth — from simple find and locate commands to powerful grep patterns, regular expressions, and fd/ripgrep as modern alternatives.
Try It Yourself

    Practice nano. Create a file called notes.txt with nano ~/notes.txt. Type a few lines, save with Ctrl+O, exit with Ctrl+X. Open it again to verify your text was saved.

    Practice Vim. Run vimtutor and complete the first lesson. Even learning just i, Esc, :wq, and :q! gives you a fallback editor.

    Set up aliases. Open ~/.bashrc in nano. Add at least three aliases from the examples in this chapter. Run source ~/.bashrc. Try your new shortcuts.

    Organize your scripts. Create ~/bin/ and add it to your PATH (if you haven't already). This is where your scripts will live.

    Write hello.sh. Create the hello.sh script from Part 3. Make it executable and run it. Modify it to ask for the user's name with read and greet them personally.

    Write an arguments script. Create a script that takes two numbers as arguments and prints their sum:

    #!/bin/bash
    sum=$(($1 + $2))
    echo "$1 + $2 = $sum"

    Write a conditional script. Create a script that checks if a file exists:

    #!/bin/bash
    if [ -f "$1" ]; then
        echo "Found: $1 ($(wc -l < "$1") lines)"
    else
        echo "Not found: $1"
    fi

    Create the system report script. Save the system-report.sh script from this chapter in ~/bin/. Make it executable. Run it and review your system's health in one command.

    Customize your prompt. Experiment with different $PS1 values in your terminal (without making them permanent yet). Find one you like, then add it to ~/.bashrc.

    ✅ Chapter 19 Complete You can edit files in nano (and survive in Vim), customize your shell with aliases and environment variables, understand the $PATH variable, and write shell scripts with variables, conditionals, loops, functions, and command-line arguments. You have a personalized terminal and your first practical scripts. In Chapter 20, we'll master file searching with find, locate, grep, and modern alternatives.

