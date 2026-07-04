# Chapter 13: Terminal Absolute Basics

## Welcome to Phase 2

Everything in Chapters 1 through 12 was done through the graphical interface — clicking buttons, dragging windows, browsing menus. That's how most people use their computers, and there's nothing wrong with it. You can use Ubuntu for years without ever opening the terminal.

But here's the thing: the terminal unlocks capabilities that the graphical interface can't provide. It's faster, more precise, more powerful, and — once you get used to it — genuinely enjoyable to use.

This chapter assumes **zero prior terminal experience.** If you've never typed a command in your life, you're in the right place. We'll go slowly, explain everything, and by the end you'll be navigating your file system with confidence.

## What Is the Terminal?

The **terminal** is a text-based interface for interacting with your computer. Instead of clicking icons and menus, you type commands and press Enter. The computer executes the command and displays text output.

That's it. It's not magic, it's not hacking, and it's not dangerous (as long as you understand what you're typing). It's just a different way of talking to your computer — one that happens to be extremely efficient once you learn the language.

### Terminal vs. Shell vs. Console

These terms get thrown around interchangeably, and honestly, for a beginner, the distinction doesn't matter much. But for clarity:

- **Terminal** — the window/application you type into (the visual container)
- **Shell** — the program running inside the terminal that interprets your commands (Ubuntu uses **Bash** by default, though **Zsh** and **Fish** are alternatives)
- **Console** — historically, the physical hardware terminal; today, it refers to the text-mode interface accessed via `Ctrl+Alt+F2`–`F6` (you won't need this)

When we say "open the terminal," we mean opening the terminal application. When we say "type a command," the shell inside that terminal is what processes it.

## Your Terminal: Ptyxis

Ubuntu 26.04 ships with **Ptyxis** as the default terminal application, replacing the older GNOME Terminal. Ptyxis is a modern, container-aware terminal built for GNOME 50 with a clean libadwaita interface.

### Opening Ptyxis

Three ways to open it:

1. **Super key → type "Terminal" → press Enter** — the fastest method
2. **Super key → type "Ptyxis" → press Enter** — if you want to be specific
3. **Keyboard shortcut: Ctrl+Alt+T** — the classic shortcut (may need to be configured in Settings → Keyboard)

> 💡 **What Does "Container-Aware" Mean?**
>
> Ptyxis can detect when you're inside a containerized environment (like a toolbox or distrobox container) and adjusts its appearance accordingly — showing a different color theme or banner to remind you which environment you're in. For beginners, this doesn't matter yet. It's a feature that becomes useful later when you work with development containers (Chapter 23).

### What You See When Ptyxis Opens

When the terminal opens, you'll see something like this:

username@hostname:~$

This single line of text is called the prompt (or command prompt). It's the terminal's way of saying "I'm ready — what do you want me to do?"

Let's break it down:
Part	Meaning
username	Your logged-in username
@	Separator (read as "at")
hostname	Your computer's name (set during installation)
:	Separator between hostname and directory
~	Your current directory — ~ is shorthand for your home directory
$	Indicates you're a regular user (not root/administrator)
(blinking cursor)	Where your text appears when you type

So john@laptop:~$ means: "John is logged into a computer called laptop, currently in the home directory, as a regular user."

Read the whole thing as: "John at laptop, in home, dollar." The $ at the end is the universal symbol for "regular user prompt." When you see # instead of $, it means you're operating as root (administrator), which we'll cover when we get to sudo.
The Golden Rules of the Terminal

Before we type anything, memorize these rules:
Rule 1: Read Before You Press Enter

The terminal does exactly what you tell it to — nothing more, nothing less. If you make a typo, the terminal doesn't guess what you meant. It tries to execute the command as typed and either succeeds or shows an error. Always read the command before pressing Enter.
Rule 2: Case Matters

In the graphical interface, File.txt and file.txt might as well be the same thing. In the terminal, they are completely different files. Linux is case-sensitive:

    Documents ≠ documents ≠ DOCUMENTS
    ls (list files) is a valid command; LS is not
    CD won't work; cd will

This trips up Windows migrants constantly. Train yourself to pay attention to capitalization.
Rule 3: Spaces Matter

Spaces separate commands from their arguments:

ls -la

Here, ls is the command and -la is an argument (a modifier). If you wanted to list a folder called "My Photos":

ls My Photos

The terminal sees this as two arguments: My and Photos. It will try to list a file called My and a file called Photos separately. To handle spaces in filenames:

    Option A: Use quotes: ls "My Photos"
    Option B: Escape the space with a backslash: ls My\ Photos

    💡 Pro Tip: Avoid Spaces in Filenames

    When you create files and folders from the terminal, use hyphens or underscores instead of spaces: my-photos, vacation_pics, project_notes.txt. This saves you from quoting and escaping headaches later. Existing files with spaces still work fine — you just need quotes when referencing them in commands.

Rule 4: Don't Panic on Errors

Errors in the terminal look intimidating but are usually straightforward. When a command fails, read the error message — it typically tells you exactly what went wrong:

bash: lss: command not found

Translation: "You typed lss, but there's no such command. Did you mean ls?"

ls: cannot access '/home/user/nonexistent': No such file or directory

Translation: "That file or folder doesn't exist at the path you specified."

Errors aren't punishments — they're information. Read them, understand them, correct your command, and try again.
Rule 5: You Can't Break Much Without sudo

Almost every command you'll learn in this chapter is read-only — it displays information but doesn't change anything. You can't accidentally delete files or break your system with ls, pwd, cd, or cat. The dangerous commands require sudo (administrator privileges), which always asks for your password as a safety checkpoint.
Your First Commands

Let's type some real commands. Open your terminal and follow along.
Command 1: pwd (Print Working Directory)

Type this and press Enter:

pwd

Output:

/home/yourusername

pwd tells you where you currently are in the file system. Right now, you're in your home directory. Think of pwd as asking "Where am I?" — it prints the full path of your current location.

    💡 Why "Print"?

    In Unix terminology, "print" often means "display on screen," not "send to a printer." pwd = "print working directory" = "show me where I am." This naming convention appears throughout the terminal.

Command 2: ls (List)

Type:

ls

Output (varies by system):

Desktop    Downloads  Music     Pictures  Templates  Videos
Documents  Movies     Public    snap

ls lists the contents of your current directory. These are the same folders you see when you open the Files app — the terminal is showing you the same thing, just as text instead of icons.

Useful variations:

ls -l

Shows a detailed list with file sizes, permissions, owners, and dates:

drwxr-xr-x 2 username username 4096 Jul  4 10:30 Desktop
drwxr-xr-x 2 username username 4096 Jul  4 10:30 Documents
drwxr-xr-x 2 username username 4096 Jul  4 10:30 Downloads

ls -a

Shows all files, including hidden ones (files starting with a .):

.              .bashrc    .profile    Desktop    Downloads
..             .cache     .config     Documents  Music

ls -la

Combines both — detailed list including hidden files. This is the most commonly used variant among experienced users.

What's with the hidden files? On Linux, any file or folder whose name starts with a dot (.) is hidden from normal directory listings. These are typically configuration files — .bashrc (shell settings), .config (app configs), .profile (login settings). You'll rarely need to touch these directly, but knowing they exist is useful.

    💡 Hidden Files in Files (Nautilus)

    To see hidden files in the graphical file manager, press Ctrl+H. The same dot-prefixed files will appear. Press Ctrl+H again to hide them.

Command 3: cd (Change Directory)

cd moves you to a different directory. It's how you "navigate" in the terminal — the equivalent of double-clicking a folder in Files.

cd Documents

You won't see any output — but if you run pwd now:

pwd

Output:

/home/yourusername/Documents

You've moved! The prompt should also have changed to reflect your new location:

username@hostname:~/Documents$

Essential cd patterns:
Command	What It Does
cd Documents	Move into the Documents folder (relative path)
cd /home/username/Documents	Move using the full path (absolute path)
cd ..	Go up one directory (to the parent)
cd .	Stay in the current directory (not useful, but valid)
cd ~ or just cd	Go back to your home directory
cd -	Go back to the previous directory you were in
cd ~/Pictures	Go to Pictures inside your home directory

Relative vs. Absolute Paths:

    Relative path — relative to where you are now: cd Documents (from home) or cd ../Pictures (from Documents, go up one level, then into Pictures)
    Absolute path — starts from the root (/): cd /home/username/Documents (works from anywhere)

    💡 The ~ Shortcut

    The tilde (~) character is shorthand for "my home directory." So ~/Documents means "/home/yourusername/Documents." You'll use this constantly — it's faster than typing your full path.

Command 4: clear (Clear the Screen)

After practicing a few commands, your terminal will be cluttered with output. Type:

clear

The screen wipes clean, and your prompt returns to the top. Nothing is deleted — the previous commands are still in your history. It's just a visual reset.

Alternative: Press Ctrl+L for the same effect without typing.
Command 5: echo (Print Text)

echo "Hello, World!"

Output:

Hello, World!

echo simply prints back whatever you give it. It seems useless now, but it's invaluable for:

    Checking the value of variables: echo $HOME
    Writing text to files: echo "Shopping list" > list.txt
    Testing that the terminal is responsive

Command 6: whoami

whoami

Output:

yourusername

Tells you which user account you're currently operating as. Useful when you have multiple accounts or when you've used sudo and want to verify your identity.
Command 7: date

date

Output:

Sat Jul  4 14:32:07 UTC 2026

Displays the current date and time. Simple but handy.
Understanding the Linux File System Structure

Now that you can navigate, let's understand what you're navigating through. The Linux file system is organized as a single tree — everything starts from the root directory, represented by a single forward slash: /

Here's the key difference from Windows: Windows has multiple roots (C:\, D:\, E:\). Linux has one root (/), and everything — including other drives and partitions — lives underneath it.

/
├── home/
│   ├── username/          ← Your personal files live here
│   │   ├── Desktop/
│   │   ├── Documents/
│   │   ├── Downloads/
│   │   ├── Music/
│   │   ├── Pictures/
│   │   └── Videos/
│   └── anotheruser/
├── bin/                    ← Essential programs (ls, cp, mv)
├── boot/                   ← Boot loader and kernel files
├── dev/                    ← Device files (hardware)
├── etc/                    ← System-wide configuration files
├── lib/                    ← Shared libraries
├── media/                  ← Auto-mounted USB drives, SD cards
├── mnt/                    ← Manually mounted drives
├── opt/                    ← Third-party software
├── proc/                   ← Process/kernel information (virtual)
├── root/                   ← Root user's home directory (not yours!)
├── sbin/                   ← System administration programs
├── srv/                    ← Service data (web servers, etc.)
├── tmp/                    ← Temporary files
├── usr/                    ← User programs and data
├── var/                    ← Variable data (logs, caches, mail)
└── snap/                   ← Snap package data

As a regular user, you'll spend 95% of your time in /home/yourusername/. The other directories are mostly system-managed — you can look at them (with ls) but shouldn't modify them without sudo.

    💡 Exploring Safely

    You can cd into any system directory and ls its contents without risk. Reading is always safe. The terminal only becomes dangerous when you start writing (creating, modifying, deleting) — and those commands require either being in your home directory or using sudo.

External Drives: When you plug in a USB drive, it doesn't appear as D:\ or E:\. Instead, it's automatically mounted under /media/yourusername/DriveName/. You can navigate there with:

cd /media/yourusername/
ls

Essential Terminal Skills

These skills will make your terminal experience dramatically smoother. Learn them now — they pay off forever.
Tab Completion (The Most Important Skill)

You almost never need to type full file or directory names. Press the Tab key, and the terminal completes the name for you.

Try it:

    Type cd Doc (don't press Enter)
    Press Tab
    The terminal completes it to cd Documents/
    Press Enter

Tab completion works on:

    File names: cat fil + Tab → cat file.txt
    Directory names: cd Down + Tab → cd Downloads/
    Command names: apt + Tab + Tab → shows all commands starting with apt
    Command options: ls --h + Tab → ls --help

If there's ambiguity (multiple matches), pressing Tab twice shows all possibilities:

cd D

Press Tab twice:

Desktop/   Documents/   Downloads/

Now type enough to disambiguate (cd De + Tab) and it completes to Desktop/.

Use tab completion religiously. It prevents typos, saves time, and teaches you what's available. If you see experienced users blazing through commands, tab completion is their secret.

    💡 Tab Completion Doesn't Work?

    If pressing Tab does nothing, check:

        You typed enough characters to be unambiguous
        The file/directory actually exists
        You're in the right directory
        You haven't left a dangling space before pressing Tab

Command History (Arrow Keys)

The terminal remembers every command you've typed. Use the Up arrow and Down arrow keys to scroll through your history:

    ↑ — Previous command
    ↑↑ — Two commands ago
    ↓ — Next command (when scrolling backward)
    Ctrl+R — Search backward through history by keyword

Type Ctrl+R, then start typing part of a command you used earlier. The terminal finds and displays the most recent match. Press Enter to run it, or Right arrow to edit it before running.

Example:

    Earlier you typed: ls -la /home/username/Downloads
    Press Ctrl+R, type "Down"
    Terminal shows: ls -la /home/username/Downloads
    Press Enter to re-run it

Copy and Paste (Different in the Terminal!)

In a normal application, you copy with Ctrl+C and paste with Ctrl+V. In the terminal, this doesn't work because Ctrl+C has a different meaning (it cancels/kills the current command).
Operation	Method
Copy selected text	Ctrl+Shift+C or right-click → Copy
Paste	Ctrl+Shift+V or right-click → Paste
Cancel a running command	Ctrl+C

Alternatively, you can select text with your mouse and paste it with a middle-click (click the scroll wheel). This is the classic Unix way — select to copy, middle-click to paste. Many users find this faster once they're used to it.

    ⚠️ Ctrl+C Means "Stop!"

    If you accidentally press Ctrl+C while a command is running, it will terminate that command immediately. This is actually useful — if a command is stuck or taking too long, Ctrl+C stops it. But don't press it expecting to copy text.

Opening Multiple Tabs

Ptyxis supports tabs, just like a browser:

    Ctrl+Shift+T — Open a new tab
    Ctrl+Shift+W — Close the current tab
    Ctrl+Page Up/Page Down — Switch between tabs

Tabs are useful when you need to run commands in different directories simultaneously. For example, one tab in your home directory, another in a project folder.
Anatomy of a Command

Let's look at a typical command to understand its structure:

ls -la /home/username/Documents

Breaking this down:
Component	Value	Role
Command	ls	The program to run
Option (flag)	-la	Modifies behavior (-l = long format, -a = all files)
Argument	/home/username/Documents	What the command operates on (a directory path)

Not every command needs all three parts:

    ls — just the command (lists current directory)
    ls -l — command + option (detailed listing of current directory)
    ls Documents — command + argument (lists the Documents folder)
    ls -l Documents — command + option + argument (detailed listing of Documents)

Options vs. Arguments:

    Options (also called flags) start with - (single dash, single character) or -- (double dash, full word). They modify how the command behaves.
    Arguments specify what the command acts upon — typically file paths or names.

Examples of both option styles:

    ls -l — short option (-l for long format)
    ls --human-readable — long option (same as -h, easier to read)

Most commands support --help to show available options:

ls --help

This displays a summary of all options the ls command accepts. Reading --help output is a skill — it's terse by design. With practice, you'll learn to scan it quickly for what you need.

    💡 Reading Help Output

    Don't try to memorize all options. Most commands have dozens. Instead:

        Run command --help to see what's available
        Scan for the option you need
        Try it
        If you need deeper detail, run man command (opens the full manual — press q to quit)

    We cover man pages in Chapter 14.

Putting It Together: A Practice Session

Let's combine everything we've learned into a realistic navigation scenario. Follow along in your terminal:

# 1. Check where you are
pwd
# Output: /home/yourusername

# 2. See what's in your home directory
ls

# 3. See everything including hidden files, in detail
ls -la

# 4. Move into your Documents folder
cd Documents

# 5. Confirm your new location
pwd
# Output: /home/yourusername/Documents

# 6. List the contents
ls

# 7. Go back home
cd

# 8. Check who you are
whoami

# 9. Display the date
date

# 10. Clear the screen
clear

Congratulations — you've just navigated a Linux file system entirely by text. No mouse clicks, no graphical interface. This is the foundation everything else in this manual builds upon.
Common Beginner Mistakes
Mistake 1: Typing the $

In this manual (and all Linux documentation), commands are shown with a $ prefix:

$ ls -la

The $ represents the prompt — you're not supposed to type it. Only type what comes after the $:

✅ Type: ls -la ❌ Don't type: $ ls -la

Similarly, lines starting with # are comments (in bash scripts) or indicate root prompt. In this manual, # at the start of a code block line is a comment — don't type those either.
Mistake 2: Forgetting to Press Enter

The terminal doesn't execute anything until you press Enter. If you type a command and nothing happens, check if there's a blinking cursor on the same line — you probably forgot to press Enter.
Mistake 3: Pressing Ctrl+C by Accident

If you press Ctrl+C (intending to copy), the terminal cancels whatever it was doing. If nothing was running, nothing happens. If a command was running, it stops. Just re-type your command and continue.
Mistake 4: Typing in the Wrong Case

CD is not the same as cd. LS is not the same as ls. Linux is case-sensitive — always match the case shown in documentation exactly.
Mistake 5: Being in the Wrong Directory

If a command can't find a file, check your location:

pwd

Make sure you're in the directory you think you're in. If not, cd to the right place.
Mistake 6: Not Reading the Error

When something fails, the error message almost always explains why. Read it carefully before asking for help. Common errors:
Error Message	What It Means
command not found	You misspelled the command or it's not installed
No such file or directory	The path you typed doesn't exist
Permission denied	You don't have rights to access that file/directory
Is a directory	You tried to use a command on a directory that expects a file
What sudo Does (Preview)

You've seen sudo sprinkled throughout earlier chapters (Chapter 7's post-install checklist, Chapter 10's software installation). Now let's briefly explain what it is — we'll cover it in depth in Chapter 14.

sudo stands for "superuser do." It temporarily elevates your privileges to administrator level for a single command.

When you type:

sudo apt update

The terminal responds:

[sudo] password for yourusername:

You type your password (nothing appears on screen as you type — this is normal, not a bug) and press Enter. The command then runs with administrator privileges.

Key points for now:

    sudo asks for your password, not a separate admin password
    Nothing appears when typing the password (no asterisks, no dots) — this is intentional
    The elevated privilege lasts for about 15 minutes before asking again
    Never run sudo with a command you don't understand

We'll use sudo more in the coming chapters, but this preview should demystify what you've already encountered.
How to Exit the Terminal

To close the terminal:

    Type exit and press Enter
    Or press Ctrl+D (signals end of session)
    Or click the X button like any other window

All three methods close the terminal gracefully. Your command history is saved automatically and will be available next time you open Ptyxis.
What's Next

You've taken your first steps into the terminal. You can navigate the file system, list directory contents, understand the prompt, use tab completion, recall command history, and copy/paste correctly. This is genuinely the hardest part — the initial hurdle of feeling comfortable with a text interface.

In Chapter 14, we'll expand your toolkit: creating and deleting files and directories, viewing file contents, searching for text, and understanding file permissions. Each chapter from here builds incrementally — you're never asked to use a command we haven't explained.
Try It Yourself

    Navigate your home directory. Use pwd to see where you are. Use ls to see what's there. Use cd to move into different folders. Use cd to come back home.

    Practice tab completion. Navigate to /home using cd /ho + Tab. Then list what's there with ls. Tab-complete your own username.

    View hidden files. Run ls -a in your home directory. Notice the dot-prefixed files. Run ls -la to see details.

    Explore the root directory. Run cd / to go to the root of the file system. Run ls to see the top-level directories. Navigate into a few (cd etc, cd usr) and ls their contents. Use cd to return home.

    Practice command history. Type several commands, then use the Up arrow to recall them. Try Ctrl+R to search for a command by keyword.

    Use echo. Run echo $HOME to see your home directory path. Run echo "My name is $(whoami)" to combine commands.

    Read help output. Run ls --help and scan the options. Try one you haven't used before, like ls -lh (human-readable file sizes) or ls -t (sort by modification time).

    ✅ Chapter 13 Complete You can open the terminal, read the prompt, navigate directories with cd, list contents with ls, use tab completion, recall command history, copy/paste correctly, and understand the basic structure of Linux commands. In Chapter 14, we'll create, edit, and manage files from the terminal.

