# Chapter 14: Files, Directories, and Permissions

## From Reading to Doing

In Chapter 13, you learned to navigate the file system — moving between directories, listing their contents, and understanding where you are. Those were all **read-only** operations: you looked at things but didn't change anything.

This chapter takes the next step. You'll learn to create files, create directories, view file contents, copy, move, rename, delete — and critically, understand **permissions**: the system that decides who can read, modify, or execute files on your system.

By the end of this chapter, you'll be able to manage your entire file system from the terminal without reaching for the mouse.

## Creating Files and Directories

### `touch` — Create an Empty File

The simplest way to create a file:

touch myfile.txt

This creates an empty file called myfile.txt in your current directory. Verify with ls:

ls

Output:

myfile.txt

If the file already exists, touch doesn't overwrite it — it just updates the file's modification timestamp (the last-changed time). This is useful in scripting but for now, think of touch as "create a new empty file."

You can create multiple files at once:

touch notes.txt draft.txt ideas.md todo.txt

Four files created in one command.

    💡 File Extensions in Linux

    On Windows, file extensions (.txt, .docx, .exe) determine what happens when you double-click a file. Linux is different — extensions are conventions, not requirements. The system determines file type by examining the file's content, not its name.

    You could name a text file myfile with no extension and it would still be a text file. Extensions exist for human convenience and for applications to recognize their files — but the system itself doesn't enforce them.

    Convention guide:
    Extension	Convention
    .txt	Plain text file
    .md	Markdown file
    .sh	Shell script
    .py	Python script
    .jpg, .png	Image files
    .pdf	PDF document
    .deb	Debian package
    .conf	Configuration file

mkdir — Create a Directory

mkdir myfolder

Creates a directory called myfolder in your current location. Verify:

ls

Output:

myfolder  myfile.txt

Creating multiple directories at once:

mkdir projects photos music

Creating nested directories (parent + child in one step):

mkdir -p projects/website/images

The -p flag (parents) creates the entire chain: projects/, then projects/website/, then projects/website/images/ — even if none of them existed before. Without -p, the command would fail because the parent projects/website/ doesn't exist yet.

This is one of the most useful flags you'll learn. Remember it.
Viewing File Contents
cat — Display a File's Entire Contents

cat myfile.txt

cat (short for "concatenate") reads a file and prints its entire content to the terminal. For short files, this is perfect:

cat todo.txt

Output:

Buy groceries
Call dentist
Finish Ubuntu manual
Submit expense report

You can also view multiple files in sequence:

cat notes.txt draft.txt

The output shows notes.txt first, then draft.txt, with no separator between them.

    ⚠️ Don't cat Large Files

    If you run cat on a file with thousands of lines, the entire content scrolls past at once — too fast to read. For large files, use less (below) instead.

less — Paginated Viewer

less todo.txt

less opens the file in a full-screen pager — you can scroll through it one page at a time.

Navigation inside less:
Key	Action
Spacebar or Page Down	Forward one page
Page Up or b	Backward one page
↓ / ↑	Scroll line by line
/	Search forward (type keyword, press Enter)
n	Next search match
N	Previous search match
g	Jump to beginning
G	Jump to end
q	Quit (return to terminal)

less is ideal for reading long configuration files, log files, or any file too big for cat. It only loads what fits on screen, so it works instantly even on multi-gigabyte files.

    💡 Less Is More

    The name is a joke: less was created as an improvement over an older pager called more. The motto: "less is more." You'll also see more on some systems, but less is universally preferred.

head and tail — View the Beginning or End

head shows the first 10 lines of a file:

head todo.txt

Show a specific number of lines:

head -n 5 todo.txt

Shows the first 5 lines.

tail shows the last 10 lines:

tail todo.txt

Last 20 lines:

tail -n 20 todo.txt

tail is especially useful for log files — you typically only care about the most recent entries (which are at the end). You'll use this frequently in Chapter 30 (Troubleshooting) when reading system logs.

    💡 Following a Log File in Real Time

    tail -f /var/log/syslog continuously displays new lines as they're added to the file. This is called "following" — essential for watching live system activity. Press Ctrl+C to stop.

Copying, Moving, and Renaming
cp — Copy Files

cp myfile.txt copy_of_myfile.txt

Creates a duplicate of myfile.txt called copy_of_myfile.txt in the same directory.

Copying to a different directory:

cp myfile.txt Documents/

Copies myfile.txt into the Documents folder. The original stays in place.

Copying a directory (recursively):

cp -r myfolder myfolder_backup

The -r flag (recursive) is mandatory for directories — without it, cp refuses to copy a directory. This copies the entire folder and everything inside it.

Preserving attributes:

cp -a myfolder myfolder_backup

The -a flag (archive) preserves permissions, timestamps, and ownership — useful for making exact backups.
mv — Move and Rename

mv does double duty: it moves files to new locations AND renames them. The system doesn't distinguish between "move" and "rename" — both are just changing the path.

Rename a file:

mv oldname.txt newname.txt

The file's name changes. It stays in the same directory.

Move a file to another directory:

mv myfile.txt Documents/

myfile.txt is now in Documents/ and no longer in the current directory.

Move and rename simultaneously:

mv myfile.txt Documents/renamed_file.txt

Moves the file to Documents AND gives it a new name in one action.

Move a directory:

mv myfolder /tmp/

No -r flag needed — mv moves directories natively.

    💡 mv vs. cp

        cp creates a duplicate — the original stays in place
        mv relocates — the original is gone from its old location

    Think: "cp" = copy (keep original), "mv" = move (take it with you).

rm — Remove Files

rm myfile.txt

Deletes the file permanently. There is no recycle bin in the terminal — once you rm a file, it's gone.

    ⚠️ rm Is Permanent

    Unlike the graphical Files app (which moves deleted items to Trash), rm permanently deletes files. There's no undo. Before pressing Enter, double-check the filename.

    If you want a safer alternative that moves files to Trash instead of permanently deleting them, install trash-cli:

    sudo apt install trash-cli

    Then use trash-put filename instead of rm filename. Files go to Trash and can be recovered from the graphical Trash or with trash-restore.

Removing multiple files:

rm file1.txt file2.txt file3.txt

Removing a directory and its contents:

rm -r myfolder

The -r flag (recursive) is required for directories. This deletes the folder and everything inside it.

Force removal without confirmation:

rm -f stubborn_file.txt

The -f flag (force) suppresses confirmation prompts for write-protected files. Use cautiously.

The nuclear option (DO NOT RUN THIS):

rm -rf /

    🚨 Danger Zone

    rm -rf / would attempt to delete your entire file system — every file, every directory, everything. This is the most destructive command in Linux. Never type this. Never paste commands containing this. Never run it as a joke.

    Modern Linux systems have a safeguard (--preserve-root) that blocks this by default, but variants like rm -rf /* can still cause catastrophic damage.

    If you ever see this in a tutorial or forum post, it's either a joke at your expense or malicious. Walk away.

rmdir — Remove Empty Directories

rmdir emptyfolder

rmdir only removes directories that are completely empty. If the directory contains any files, rmdir refuses — protecting you from accidentally removing non-empty folders.

This is the safe alternative to rm -r when you know the directory should be empty. If rmdir fails, it tells you the directory isn't empty — prompting you to check its contents first.
Wildcards (Globbing)

Wildcards let you operate on multiple files at once without listing each one. They're patterns the shell expands before passing to the command.
The Asterisk (*) — Match Anything

The * matches any number of characters (including zero):

ls *.txt

Lists only .txt files.

cp *.txt Documents/

Copies all .txt files to Documents.

rm photo_*.jpg

Removes all files starting with photo_ and ending in .jpg.

ls *

Lists the contents of every directory in the current location (effectively a recursive listing).
The Question Mark (?) — Match One Character

? matches exactly one character:

ls file?.txt

Matches file1.txt, file2.txt, fileA.txt — but not file12.txt (two characters).
Character Classes ([...])

Match one character from a set:

ls file[123].txt

Matches file1.txt, file2.txt, file3.txt — but not file4.txt.

ls photo[0-9]*.jpg

Matches any .jpg file starting with photo followed by a digit.

    💡 Test Before You Act

    Before using wildcards with destructive commands like rm, test with ls first:

    ls photo_*.jpg    # See what matches
    rm photo_*.jpg    # Then delete if correct

    This habit prevents accidental deletion of unintended files.

Understanding File Permissions

This is the concept that separates Linux beginners from intermediate users. Permissions are fundamental to how Linux works — and they're simpler than they look.
The Three Types of Permissions

Every file and directory on Linux has three permission types:
Permission	Letter	For Files	For Directories
Read	r	View file contents (cat, less)	List directory contents (ls)
Write	w	Modify file contents	Create/delete files inside the directory
Execute	x	Run the file as a program	Enter the directory (cd)
The Three Categories of Users

Permissions are assigned to three categories:
Category	Letter	Who They Are
Owner	u	The user who owns the file (usually the creator)
Group	g	A group of users (e.g., staff, sudo, adm)
Others	o	Everyone else on the system
Reading Permission Strings

When you run ls -l, the first column shows the permission string:

-rwxr-xr-- 1 username staff 2048 Jul 4 14:30 script.sh

Breaking down the first column: -rwxr-xr--
Position	Character	Meaning
1	-	File type: - = regular file, d = directory, l = symbolic link
2	r	Owner: read permission
3	w	Owner: write permission
4	x	Owner: execute permission
5	r	Group: read permission
6	-	Group: no write permission
7	x	Group: execute permission
8	r	Others: read permission
9	-	Others: no write permission
10	-	Others: no execute permission

So -rwxr-xr-- means:

    Regular file (not a directory)
    Owner can read, write, and execute
    Group can read and execute (but not write)
    Others can read only (neither write nor execute)

Let's practice reading a few:
String	File Type	Owner	Group	Others
-rw-r--r--	Regular file	rw	r	r
drwxr-xr-x	Directory	rwx	rx	rx
-rwx------	Regular file	rwx	none	none
drwxrwxrwx	Directory	rwx	rwx	rwx (dangerous — everyone has full access)

    💡 Directory Permissions Are Counterintuitive

    For directories, x (execute) means "enter" — you need x on a directory to cd into it. r on a directory means "list its contents" — you need r to ls it.

    If you have x but not r on a directory, you can cd into it and access files by name (if you know their names), but you can't ls to see what's there. If you have r but not x, you can ls filenames but can't cd in or access any files.

chmod — Change Permissions

The chmod command (change mode) modifies permissions. There are two ways to use it: symbolic notation and octal (numeric) notation.
Symbolic Notation (Easier to Learn)

Symbolic notation uses letters and operators:
Operator	What It Does
+	Add a permission
-	Remove a permission
=	Set exact permissions (overwrites existing)

Examples:

chmod +x script.sh

Adds execute permission for everyone (owner, group, others). This is the most common chmod command — it makes a file runnable as a program.

chmod u+x script.sh

Adds execute permission for the owner only.

chmod g+w report.txt

Adds write permission for the group.

chmod o-r secret.txt

Removes read permission from others.

chmod u=rwx,g=rx,o= script.sh

Sets exact permissions: owner gets rwx, group gets rx, others get nothing.
Octal Notation (Faster Once Learned)

Octal notation uses three digits (0–7), one for each category (owner, group, others):
Digit	Permissions	Letters
0	No permissions	---
1	Execute only	--x
2	Write only	-w-
3	Write + Execute	-wx
4	Read only	r--
5	Read + Execute	r-x
6	Read + Write	rw-
7	Read + Write + Execute	rwx

The formula: Read = 4, Write = 2, Execute = 1. Add them together for the desired permission set.

Examples:

chmod 755 script.sh

Sets rwxr-xr-x — owner has full access, group and others can read and execute.

chmod 644 document.txt

Sets rw-r--r-- — owner can read/write, everyone else can read. This is the default for most files.

chmod 600 secrets.txt

Sets rw------- — only the owner can read or write. Nobody else has any access.

chmod 777 sharedfolder/

Sets rwxrwxrwx — everyone has full access. Generally avoid this — it's a security risk on multi-user systems.

    💡 Which Notation Should I Use?

    Start with symbolic (chmod +x script.sh) — it's readable and obvious. Transition to octal (chmod 755 script.sh) as you gain confidence — it's faster and what you'll see in tutorials and documentation.

    The two most common permission sets you'll use:

        chmod 644 — standard file (readable by all, writable by owner)
        chmod 755 — executable/script (executable by all, writable by owner)

chown — Change Ownership (Brief)

sudo chown username file.txt

Changes the owner of file.txt to username. This requires sudo because changing ownership affects security.

sudo chown username:groupname file.txt

Changes both the owner and the group.

You won't use chown frequently as a regular user — the system manages ownership automatically when you create files. It's mainly useful when transferring files between users or fixing permission issues from copied files.
Manual Pages: man

Throughout this chapter, you've seen commands with many options. Rather than memorizing every flag, Linux provides built-in documentation called manual pages (man pages).
Using man

man ls

Opens the complete manual for the ls command. Navigation is identical to less:
Key	Action
Spacebar / Page Down	Forward one page
b / Page Up	Backward one page
/	Search for a keyword
n	Next search match
q	Quit
Sections of a Man Page

Man pages follow a standard structure:
Section	What It Contains
NAME	Command name and brief description
SYNOPSIS	Command syntax (how to invoke it)
DESCRIPTION	Detailed explanation of what the command does
OPTIONS	Every flag and argument explained
EXAMPLES	Usage examples (not all man pages have this)
SEE ALSO	Related commands
AUTHOR	Who wrote the program
BUGS	Known issues or where to report them
Finding What You Need

Man pages are comprehensive but dense. Strategy for using them effectively:

    Open the man page: man ls
    Search for what you need: /sort (finds the sort option)
    Read the relevant section
    Press q to quit
    Try the option in your terminal

Other Documentation Commands

ls --help

Brief help — shorter and less detailed than man pages but often sufficient.

tldr ls

Community-maintained simplified examples (install with sudo apt install tldr):

info ls

GNU info pages — more detailed than man pages for GNU software, but less common.

    💡 When to Use What

        --help — quick reminder of options
        man — thorough reference
        tldr — practical examples
        Google/web search — real-world usage and edge cases

Practical Exercise: Putting It All Together

Follow along in your terminal — this exercise uses every command from this chapter:

# 1. Go to your home directory
cd

# 2. Create a practice directory
mkdir practice

# 3. Enter it
cd practice

# 4. Create some files
touch hello.txt notes.md ideas.txt

# 5. Create a subdirectory
mkdir archive

# 6. Write text to a file
echo "Learning the terminal is fun!" > hello.txt

# 7. View the file
cat hello.txt

# 8. Add more text (>> appends, > overwrites)
echo "This is a second line." >> hello.txt

# 9. View it again
cat hello.txt

# 10. Make a copy
cp hello.txt hello_backup.txt

# 11. Move the backup into the archive folder
mv hello_backup.txt archive/

# 12. List everything including the subdirectory
ls -l

# 13. Check the archive folder contents
ls archive/

# 14. Rename a file
mv ideas.txt brilliant_ideas.md

# 15. Check permissions on hello.txt
ls -l hello.txt

# 16. Add execute permission
chmod +x hello.txt

# 17. Verify the change
ls -l hello.txt

# 18. Remove execute permission
chmod -x hello.txt

# 19. Set specific permissions (owner read/write, others read)
chmod 644 hello.txt

# 20. Clean up everything when done
cd
rm -r practice

    💡 The > and >> Operators

    These are redirection operators — they send command output to a file instead of the screen:

        echo "text" > file.txt — Creates/overwrites file.txt with "text"
        echo "more" >> file.txt — Appends "more" to the end of file.txt

    We cover redirection in depth in Chapter 15.

File Types in Linux

When you run ls -l, the first character of the permission string indicates the file type:
Character	File Type	Description
-	Regular file	Text, images, executables, archives
d	Directory	A folder
l	Symbolic link	A shortcut/pointer to another file
c	Character device	Serial port, terminal (streams data char by char)
b	Block device	Hard drive, USB drive (accessed in blocks)
p	Named pipe	Inter-process communication channel
s	Socket	Network communication endpoint

As a beginner, you'll mostly encounter regular files (-), directories (d), and occasionally symbolic links (l). The others are system-level constructs you'll encounter in advanced usage.
Symbolic Links (Brief)

A symbolic link (symlink) is a file that points to another file — similar to a Windows shortcut:

ln -s original_file.txt link_to_file.txt

Now link_to_file.txt is a pointer — accessing it is the same as accessing original_file.txt. If the original is deleted, the link becomes "broken" (points to nothing).

We cover symlinks in more detail in Chapter 22 (File System Deep Dive).
Permission Summary Cheatsheet

For quick reference until you memorize these:
Permission (Octal)	Symbolic	Typical Use
644	rw-r--r--	Standard text file — owner edits, everyone reads
755	rwxr-xr-x	Executable/script — owner edits, everyone runs
600	rw-------	Private file — only owner can access
700	rwx------	Private executable — only owner can run
664	rw-rw-r--	Shared file — owner and group can edit
775	rwxrwxr-x	Shared directory — group can create files
750	rwxr-x---	Restricted executable — group can run, others locked out
777	rwxrwxrwx	Avoid — everyone has full access
What's Next

You can now create, view, copy, move, rename, and delete files and directories from the terminal. You understand the Linux permission system — read, write, execute — across owner, group, and others. You can change permissions with both symbolic and octal notation. You know how to read man pages for any command.

In Chapter 15, we'll go deeper: redirecting output, piping commands together, searching for text within files, and combining commands to accomplish complex tasks efficiently. This is where the terminal starts to feel genuinely powerful — not just an alternative to clicking, but a tool that can do things no GUI can.
Try It Yourself

    Create a file system structure. From your home directory, create a practice folder with subdirectories called docs, images, and code. Inside each, create two or three empty files using touch.

    Write and view text. Use echo to write a few lines to a file. Use cat to display it. Use less to read it in the pager. Try head -n 2 and tail -n 2 on the same file.

    Copy and move files. Copy a file from one subdirectory to another. Rename a file using mv. Move an entire directory's contents using mv * inside that directory.

    Practice wildcards. Create 10 files named file1.txt through file10.txt. List only files matching file1*. List only files matching file?.txt. Copy all .txt files to another directory in one command.

    Read permissions. Run ls -l in various directories (your home, /etc, /usr/bin). Read the permission strings. Identify which files you can read, which you can execute, and which are restricted.

    Change permissions. Create a file, check its default permissions, then change them using both symbolic (chmod u+x) and octal (chmod 755) notation. Verify each change with ls -l.

    Read a man page. Open man ls. Search for the word "recursive." Find the option for recursive listing. Try it on your practice directory.

    Clean up. When you're done practicing, use rm -r to remove your practice directory. Verify it's gone with ls.

    ✅ Chapter 14 Complete You can create, view, copy, move, rename, and delete files and directories from the terminal. You understand the three permission types (read, write, execute), three user categories (owner, group, others), can read permission strings, change them with chmod, and look up any command's documentation using man. In Chapter 15, we'll learn to chain commands together for real power.

