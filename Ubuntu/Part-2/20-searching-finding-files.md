# Chapter 20: Searching and Finding Files

## The Needle in the Haystack

In Chapter 15, you got a first taste of file searching — basic `find` for locating files by name and basic `grep` for searching text inside files. Those introductions were enough to get started, but searching is a skill you'll use constantly, and there's much more depth to explore.

This chapter goes further: `find` with date, size, and permission filters, `locate` for lightning-fast filename lookups, advanced `grep` patterns with regular expressions, and two modern Rust-powered tools — `fd` and `ripgrep` — that are faster and friendlier than their classic counterparts.

By the end, you'll be able to find any file on your system and search any content within seconds.

## Finding Files by Name with `locate`

The `locate` command is the fastest way to find files by name. Instead of scanning your disk (which `find` does), `locate` searches a pre-built database of all files on the system. The result: near-instant results.

Ubuntu 26.04 uses **plocate** — a modern, high-performance replacement for the older `mlocate`. It's not installed by default but is available from the repositories:

sudo apt install plocate

After installation, build the initial database:

sudo updatedb

This scans your entire filesystem and creates the database at /var/lib/plocate/plocate.db. It takes a few seconds. After that, locate is blazing fast.
Using locate

locate firefox

Output:

/usr/bin/firefox
/usr/lib/firefox
/usr/lib/firefox/firefox.sh
/usr/share/applications/firefox.desktop
/usr/share/icons/hicolor/48x48/apps/firefox.png
/home/john/.mozilla/firefox
/home/john/.cache/mozilla/firefox

Every file on the system with "firefox" in its path — instantly.

Search with a specific pattern:

locate "*.pdf"

Finds all PDF files on the system. The wildcard * matches anything.

Limit the number of results:

locate -n 10 "*.log"

Shows only the first 10 matching log files.

Case-insensitive search:

locate -i README

Matches README, readme, ReadMe, etc.

Count matches instead of listing them:

locate -c "*.jpg"

Output:

347

Tells you 347 JPEG files exist on the system.
Keeping the Database Current

locate depends on its database being up to date. Ubuntu automatically runs updatedb daily via a cron job (scheduled task). But if you recently created or downloaded files, locate won't find them until the next database update.

Force a manual update:

sudo updatedb

    💡 locate vs. find — When to Use Which
    Tool	Speed	Freshness	Best For
    locate	Instant	Up to 24 hours stale	Quick name lookups across the whole system
    find	Slower (scans disk)	Always current	Precise searches with filters (date, size, permissions)

    Use locate when you know the filename (or part of it) and want results fast. Use find when you need to filter by attributes like size, date, or permissions, or when you need guaranteed current results.

Advanced find

Chapter 15 introduced find for basic name searches. Now let's explore its full power. find can filter by virtually any file attribute.
Review: Basic Name Search

find /home/john -name "*.txt"

Finds all .txt files in the home directory. Case-sensitive.

find /home/john -iname "*.TXT"

Case-insensitive — matches .txt, .TXT, .Txt, etc.
Searching by File Type

find /var/log -type f -name "*.log"

The -type flag filters by file type:
Code	File Type
f	Regular file
d	Directory
l	Symbolic link
b	Block device (disk, USB)
c	Character device (terminal, serial)
s	Socket
p	Named pipe (FIFO)

find /home/john -type d -name "projects"

Finds all directories named "projects."
Searching by Size

find /home/john/Downloads -type f -size +100M

Finds files larger than 100 MB in Downloads.

Size unit suffixes:
Suffix	Meaning
c	Bytes (e.g., +5000c = more than 5000 bytes)
k	Kilobytes (e.g., +10k)
M	Megabytes (e.g., +100M)
G	Gigabytes (e.g., +1G)

find /var/log -type f -size +10M -size -100M

Finds log files between 10 MB and 100 MB.
Searching by Modification Time

find /home/john -type f -mtime -7

Finds files modified in the last 7 days.

Time-based filters:
Option	Meaning
-mtime -7	Modified less than 7×24 hours ago
-mtime +7	Modified more than 7×24 hours ago
-mtime 7	Modified exactly 7 days ago
-mmin -30	Modified in the last 30 minutes
-mmin +60	Modified more than 60 minutes ago
-newer file.txt	Modified more recently than file.txt

Practical: Find files changed today:

find /home/john -type f -mmin -$((60 * $(date +%H)))

Or more simply:

find /home/john -type f -mtime 0

-mtime 0 means "modified today (less than 24 hours ago)."
Searching by Permissions

find /usr/bin -type f -perm 755

Finds files with exactly rwxr-xr-x permissions.
Option	Meaning
-perm 755	Exactly these permissions
-perm -u+s	Has the SUID bit set (runs as owner)
-perm /o+w	World-writable (any "other" user can write)
-perm /o+x	World-executable

Security audit: Find world-writable files:

find /etc -type f -perm /o+w 2>/dev/null

World-writable files in /etc are a security risk — normally there shouldn't be any.
Searching by Owner and Group

find /home -type f -user john

Finds all files owned by user john.

find /shared -type f -group developers

Finds all files owned by the "developers" group.

find / -type f -nouser 2>/dev/null

Finds "orphaned" files — owned by users that no longer exist on the system (common after deleting a user but keeping their files).
Combining Conditions

find supports AND, OR, and NOT logic:

AND (default — all conditions must match):

find /home/john -type f -name "*.pdf" -size +10M

PDFs larger than 10 MB.

OR:

find /home/john -type f \( -name "*.jpg" -o -name "*.png" \)

JPEGs or PNGs. The parentheses group the OR conditions. They must be escaped with backslashes so Bash doesn't interpret them.

NOT:

find /home/john -type f -not -name "*.tmp"

All files except .tmp files.

Complex example:

find /var/log -type f \( -name "*.log" -o -name "*.gz" \) -mtime +30 -size +1M

Log files or compressed logs, older than 30 days, larger than 1 MB — the kind of files you'd want to clean up.
Acting on Results: -exec

Finding files is only half the battle. find can also perform actions on each result.

Execute a command on each found file:

find /home/john/Downloads -name "*.pdf" -exec cp {} /backup/ \;

    {} is replaced with each filename found
    \; ends the -exec command (the backslash escapes the semicolon from Bash)

This copies every PDF from Downloads to /backup/.

Confirm each action:

find /home/john/Downloads -name "*.pdf" -ok cp {} /backup/ \;

The -ok flag works like -exec but asks for confirmation before each file. Safer for destructive operations.

Batch execution (more efficient):

find /home/john/Downloads -name "*.pdf" -exec cp {} /backup/ +

The + (instead of \;) batches multiple files into a single cp command — much faster when there are many results:

cp file1.pdf file2.pdf file3.pdf /backup/

Instead of:

cp file1.pdf /backup/
cp file2.pdf /backup/
cp file3.pdf /backup/

Deleting Files with find

find /tmp -type f -name "*.tmp" -delete

Deletes all .tmp files in /tmp. Fast and clean.

    ⚠️ Be Careful with -delete

    Always run the find command without -delete first to see what would be deleted:

    # Step 1: Check what matches
    find /tmp -type f -name "*.tmp"

    # Step 2: If the list looks right, add -delete
    find /tmp -type f -name "*.tmp" -delete

    Never combine -delete with broad searches like find / -delete — this would destroy your system.

Redirecting find Errors

find outputs permission-denied errors for files it can't read, cluttering your results. Suppress them:

find / -name "*.conf" 2>/dev/null

The 2>/dev/null redirects stderr (error messages) to nowhere — a technique you learned in Chapter 15.
Advanced grep

Chapter 15 introduced grep for basic text search. Now let's unlock its full potential.
Review: Basic grep

grep "error" /var/log/syslog

Searches for the word "error" in syslog.
Recursive grep: Searching Inside All Files

grep -r "TODO" ~/projects/

The -r flag searches recursively through all files in ~/projects/ and its subdirectories. This is how you search an entire codebase or document collection.

grep -ri "todo" ~/projects/

The -i flag makes it case-insensitive — catches TODO, todo, Todo, etc.
Showing Context

Sometimes a match alone isn't enough — you need to see surrounding lines for context.

grep -C 3 "error" /var/log/syslog

The -C 3 flag shows 3 lines before and after each match.
Flag	Context
-C 3	3 lines before AND after (context)
-B 3	3 lines Before each match
-A 3	3 lines After each match
Matching Whole Words

grep -w "cat" document.txt

The -w flag matches only the whole word "cat" — not "catalog," "concatenate," or "scattered." Essential when searching for short terms.
Showing Only Filenames

grep -rl "password" /etc/

The -l flag (list) shows only the filenames that contain matches — not the matching lines themselves. Useful when you want to know which files contain a term, not the exact content.
Counting Matches

grep -c "error" /var/log/syslog

The -c flag counts the number of matching lines instead of showing them. Quick way to assess "how bad is it?"
Inverting the Match

grep -v "#" /etc/fstab

The -v flag inverts the match — shows lines that do not contain "#". This is excellent for stripping comments from config files:

grep -vE "^\s*#" /etc/ssh/sshd_config

Shows sshd_config with all comment lines removed (^\s*# matches lines that start with optional whitespace followed by #). The -E flag enables extended regex (explained below).
Multiple Patterns

grep -e "error" -e "warning" /var/log/syslog

The -e flag lets you specify multiple search patterns — matches lines containing "error" OR "warning."

grep -E "error|warning|critical" /var/log/syslog

The -E flag enables extended regular expressions, where | means OR. This achieves the same result more concisely.
Regular Expressions in grep

Regular expressions (regex) are a mini-language for describing text patterns. grep supports two flavors:

    Basic regex (default): Limited metacharacters. . matches any character, * means "zero or more of preceding."
    Extended regex (-E or egrep): Adds +, ?, |, (), {} for more powerful patterns.

Practical regex examples:
Pattern	What It Matches
grep "foo"	Lines containing "foo"
grep "^foo"	Lines starting with "foo"
grep "foo$"	Lines ending with "foo"
grep "^$"	Empty lines
grep -E "cat|dog"	Lines containing "cat" or "dog"
grep -E "colou?r"	"color" or "colour" (? = optional u)
grep -E "[0-9]{4}"	Any 4 consecutive digits
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"	IP addresses (simplified)
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"	Email addresses
grep "\.pdf$"	Lines ending with ".pdf"

    💡 Regex Is a Deep Topic

    Regular expressions deserve their own book. For this manual, you've learned enough practical patterns to handle most everyday searching. When you need more, man grep and online regex testers (like regex101.com) are your friends.

Colorized Output

grep --color=auto "error" /var/log/syslog

Highlights the matching text in color (red by default). This makes spotting matches in dense log files much easier. Ubuntu's default ~/.bashrc already includes alias grep='grep --color=auto', so this is likely already active.
Reading from stdin

grep can filter the output of other commands (as you learned in Chapter 15):

ps aux | grep firefox

journalctl -b | grep -i "fail"

dpkg -l | grep -i python

grep with Binary Files

By default, grep prints "Binary file X matches" for binary files instead of showing the match. To see the match:

grep -a "pattern" binaryfile

The -a flag treats binary files as text. Use cautiously — binary output can mess up your terminal display. If your terminal gets garbled, run reset to fix it.
Finding Executables and Documentation

Three commands help you locate programs and their documentation:
which — Find a Command's Location

which python3

Output:

/usr/bin/python3

Shows the full path of an executable that would run if you typed its name. Searches your $PATH directories.

which -a python3

Shows all matching locations (useful when you have multiple versions installed).
whereis — Find Binary, Source, and Man Page

whereis ls

Output:

ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz

Shows the binary location and the man page location — more comprehensive than which.
type — Identify What a Command Really Is

type ll

Output:

ll is aliased to `ls -lah'

type python3

Output:

python3 is /usr/bin/python3

type cd

Output:

cd is a shell builtin

type tells you what a command actually is — an alias, an executable, a shell built-in, or a function. This is invaluable when a command behaves unexpectedly.
Modern Alternatives: fd and ripgrep

The classic find and grep are powerful but have quirks: verbose syntax, confusing defaults, and no awareness of version control. Two modern tools — both written in Rust — address these shortcomings.
fd: A Friendlier find

fd is a fast, user-friendly alternative to find. It's not installed by default:

sudo apt install fd-find

On Ubuntu, the binary is named fdfind (to avoid a naming conflict with another package). Most users create an alias:

echo 'alias fd=fdfind' >> ~/.bashrc
source ~/.bashrc

Basic usage:

fd firefox

Searches the current directory for files matching "firefox." That's it — no -name, no quotes, no . for current directory. fd just works.

Key advantages over find:
Feature	find	fd
Default search location	Requires explicit path	Current directory
Pattern matching	Exact glob (-name "*.txt")	Smart partial match
Case sensitivity	Case-sensitive by default	Smart (case-insensitive unless uppercase used)
Hidden files	Included by default	Excluded by default (cleaner results)
.gitignore	Not respected	Automatically respected
Speed	Slower	Faster (parallelized, Rust)
Syntax	Verbose, easy to mistype	Minimal, intuitive

Common fd patterns:

# Find all PDFs
fd -e pdf

# Find all PDFs and JPEGs
fd -e pdf -e jpg

# Search a specific directory
fd config /etc/

# Include hidden files
fd -H cache

# Search for exact name (glob match)
fd -g "README.md"

# List all files (no pattern)
fd

# Show file types alongside results
fd -t f -e py    # files only, Python extension
fd -t d projects # directories only named "projects"

# Execute a command on each result (like find -exec)
fd -e jpg -x convert {} {.}.png

The {} syntax in -x (exec) is the same as find. The {.} is a fd extension — it expands to the filename without extension.
ripgrep (rg): A Faster, Smarter grep

ripgrep is a recursive text search tool that's significantly faster than grep -r. It's written in Rust and respects .gitignore files automatically.

sudo apt install ripgrep

The binary is named rg.

Basic usage:

rg "error"

Searches the current directory recursively for "error." No -r flag needed — recursion is the default.

Key advantages over grep -r:
Feature	grep -r	rg
Recursion	Must specify -r	Default behavior
Speed	Good	Exceptional (Rust, parallelized)
.gitignore	Not respected	Automatically respected
Hidden files	Included by default	Excluded by default
Binary files	Prints "Binary file matches"	Skips silently
File type filtering	--include with globs	Built-in type filters (-t, -T)

Common rg patterns:

# Basic search (recursive, current directory)
rg "TODO"

# Case-insensitive
rg -i "todo"

# Search in a specific directory
rg "error" /var/log/

# Show context (2 lines before and after)
rg -C 2 "error"

# List only filenames containing matches
rg -l "password"

# Count matches per file
rg -c "error" /var/log/

# Search only in specific file types
rg -t py "import"        # Python files only
rg -t sh "#!/bin/bash"   # Shell scripts only
rg -T py "test"          # All files EXCEPT Python

# Search by file extension
rg -g "*.md" "syntax"

# Include hidden files
rg --hidden "config"

# Replace preview (shows what would change, doesn't modify)
rg -r "WARNING" "ERROR" *.log

# Invert match (lines NOT containing pattern)
rg -v "^#" /etc/fstab

# Whole word match
rg -w "cat"

# Multiple patterns (OR)
rg "error|warning|critical"

# Search for a literal string (not regex)
rg -F "192.168.1.1"

    💡 When to Use rg vs. grep

    Use rg for searching codebases, project directories, and any recursive text search — it's faster, smarter, and more ergonomic.

    Use grep for:

        Searching a single file (grep "error" /var/log/syslog)
        Piping from other commands (ps aux | grep firefox)
        Scripts that need to run on systems without rg installed
        Simple searches where you already know the syntax

    Both tools are valuable — they complement each other.

fzf: Fuzzy Finder (Bonus)

fzf is an interactive, fuzzy file finder. Instead of specifying exact patterns, you type partial matches and fzf shows results in real time:

sudo apt install fzf

Quick file search:

fzf

Opens an interactive window showing all files under the current directory. Start typing — results filter instantly. Use arrow keys to navigate, Enter to select, Esc to cancel.

Pipe command output into fzf:

find ~ -type f | fzf

apt list --installed 2>/dev/null | fzf

Browse installed packages interactively.

Integrate fzf with Bash (powerful):

Add to ~/.bashrc:

# Fuzzy find files and open in editor
bind '"\C-p": "\C-x\C-e$fzf-selected\n"'

This is more advanced — fzf integration is a deep topic. For now, try fzf standalone and see if it fits your workflow.
Practical Searching Scenarios

Let's combine tools into real-world workflows.
Scenario 1: Find All Large Files Eating Disk Space

find / -type f -size +500M 2>/dev/null | xargs du -h | sort -rh | head -20

    find / -type f -size +500M — finds files larger than 500 MB system-wide
    2>/dev/null — suppresses permission errors
    xargs du -h — shows human-readable sizes for each result
    sort -rh — sorts by size, largest first
    head -20 — shows top 20

With fd (simpler but less precise):

fdfind --full-path / -e iso -e img -e vdi | xargs du -h | sort -rh | head -20

Scenario 2: Find Which Config File Sets a Parameter

grep -rl "Port 22" /etc/ssh/

Finds which SSH config file specifies port 22. The -l flag lists only filenames.

rg "Port" /etc/ssh/

Same search with ripgrep — shows the matching content too.
Scenario 3: Find Recently Modified Files (Last Hour)

find /home/john -type f -mmin -60

All files in your home directory modified in the last 60 minutes. Useful when you can't remember where you saved something.
Scenario 4: Find Duplicate Filenames

find /home/john -type f -printf "%f\n" | sort | uniq -d

    find ... -printf "%f\n" — prints just the filename (without path) for each file
    sort — alphabetizes
    uniq -d — shows only duplicated lines

If any output appears, those filenames exist in multiple directories.
Scenario 5: Search Log Files for Errors Across Multiple Services

grep -rl "error\|fail\|critical" /var/log/ 2>/dev/null | less

Lists all log files containing any of those keywords. Then use less to browse the list.

With ripgrep:

rg -l -i "error|fail|critical" /var/log/

Faster, and automatically handles binary files gracefully.
Scenario 6: Find All Scripts You've Written

find ~/bin -type f -exec file {} \; | grep "shell script"

Uses the file command to identify shell scripts by their content (not just by extension).

Simpler approach:

fd -e sh ~/bin

Scenario 7: Clean Up Old Downloads

find ~/Downloads -type f -mtime +90 -name "*.iso"

Finds ISO files in Downloads older than 90 days — candidates for deletion. Review the list, then:

find ~/Downloads -type f -mtime +90 -name "*.iso" -delete

Searching Cheat Sheet
Task	Command
Quick filename search	locate filename
Update locate database	sudo updatedb
Find by name (current dir)	find . -name "*.txt"
Find by name (case-insensitive)	find . -iname "*.TXT"
Find by type (files only)	find . -type f -name "*.log"
Find by type (dirs only)	find . -type d -name "projects"
Find by size	find . -type f -size +100M
Find by modification time	find . -type f -mtime -7
Find by modification (minutes)	find . -type f -mmin -30
Find by permission	find /usr/bin -perm 755
Find world-writable files	find /etc -perm /o+w 2>/dev/null
Find by owner	find /home -user john
Find orphaned files	find / -nouser 2>/dev/null
Find and execute	find . -name "*.txt" -exec wc -l {} +
Find and delete	find /tmp -name "*.tmp" -delete
Find newest file	find . -type f -printf '%T@ %p\n' | sort -n | tail -1
grep in a file	grep "pattern" file.txt
grep recursive	grep -r "pattern" directory/
grep case-insensitive	grep -i "pattern" file
grep with context	grep -C 3 "pattern" file
grep whole word	grep -w "word" file
grep inverted	grep -v "pattern" file
grep count	grep -c "pattern" file
grep filenames only	grep -l "pattern" directory/*
grep multiple patterns	grep -E "a|b|c" file
Find command location	which command
Find binary + man page	whereis command
Identify command type	type command
fd basic search	fd pattern
fd by extension	fd -e pdf
fd include hidden	fd -H pattern
rg basic search	rg "pattern"
rg case-insensitive	rg -i "pattern"
rg with context	rg -C 3 "pattern"
rg file type filter	rg -t py "import"
rg list filenames	rg -l "pattern"
fzf interactive search	fzf
What's Next

You now have a comprehensive toolkit for finding files and searching content. locate for instant name lookups, find for precise attribute-based searches, grep for text pattern matching, and fd/ripgrep as modern, faster alternatives. Combined with piping and redirection from Chapter 15, these tools let you answer virtually any "where is it?" or "what contains this?" question on your system.

In Chapter 21, we'll cover networking basics — how to check your connection, find your IP address, transfer files, and troubleshoot network issues from the terminal.
Try It Yourself

    Install and use locate. Run sudo apt install plocate && sudo updatedb. Then search for a file: locate firefox. Notice how fast it is compared to find.

    Find large files. Run find /home/$USER -type f -size +50M 2>/dev/null | xargs du -h | sort -rh | head -10. Identify your largest files.

    Search with find by date. Run find /home/$USER -type f -mtime -1. See every file you've modified in the last 24 hours.

    Search with find by permission. Run find /usr/bin -type f -perm 755 | head -10. These are executable programs available to all users.

    Practice advanced grep. Run grep -v "^#" /etc/fstab to view your fstab file with comments stripped. Run grep -cv "^#" /etc/fstab to count how many non-comment lines exist.

    Use grep -r. Pick a directory and search recursively: grep -ri "ubuntu" ~/Documents/ 2>/dev/null | head -20.

    Install and try fd and ripgrep. Run sudo apt install fd-find ripgrep. Then:
        fdfind .py ~ — find all Python files in your home directory
        rg "TODO" ~/bin/ 2>/dev/null — search your scripts for TODO comments

    Try a practical scenario. Run the "find large files" scenario from this chapter. Then run the "recently modified" scenario. These two searches solve the most common "I can't find it" problems.

    Install fzf. Run sudo apt install fzf && eval "$(fzf --bash)" (or just run fzf standalone). Try browsing your home directory interactively.

    ✅ Chapter 20 Complete You can search for files by name with locate (fast database lookup) and find (precise attribute filtering), search file contents with grep (traditional) and ripgrep (modern, faster), locate executables with which/whereis/type, and use fd as a friendlier find alternative. You can combine these tools with piping to answer complex questions about your filesystem. In Chapter 21, we'll cover networking from the terminal.

