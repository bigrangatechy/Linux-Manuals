# Chapter 15: Piping, Redirection, and Searching

## From Single Commands to Command Chains

So far, every command you've learned operates in isolation: one command, one output, done. You type `ls`, you see a list. You type `cat hello.txt`, you see the file's contents. That's useful, but it's limited.

The real power of the terminal emerges when you **chain commands together** — taking the output of one command and feeding it directly into another, filtering it, transforming it, sorting it, and saving the results. This is what separates using the terminal from merely tolerating it.

This chapter introduces the three skills that unlock that power: **pipes**, **redirection**, and **text searching**. By the end, you'll be building command pipelines that accomplish in a single line what would take dozens of clicks in a graphical interface.

## Standard Streams: The Hidden Plumbing

Before learning pipes and redirection, you need to understand what's happening beneath the surface. Every Linux program has three built-in communication channels called **standard streams**:

| Stream | Number | Abbreviation | Default Destination |
|---|---|---|---|
| **Standard Input** | 0 | stdin | Keyboard (what you type) |
| **Standard Output** | 1 | stdout | Terminal screen (normal output) |
| **Standard Error** | 2 | stderr | Terminal screen (error messages) |

When you run `ls`, the list of files goes to **stdout** — which appears on your screen. If `ls` encounters an error (say, a directory you don't have permission to read), the error message goes to **stderr** — which also appears on your screen, but as a separate stream.

This distinction matters because pipes and redirection work on these streams independently. You can redirect output without touching errors, or capture errors separately, or send both to the same place. Understanding these three streams unlocks everything in this chapter.

> 💡 **Visual Mental Model**
>
> Think of each stream as a pipe in a plumbing system:
>
> - **stdin** = water coming IN from the supply
> - **stdout** = clean water going OUT to the faucet
> - **stderr** = wastewater going to the drain
>
> Pipes and redirection are valves that reroute these flows. You can redirect the clean water to a bucket (a file), or connect one program's faucet to another program's intake.

## Redirection: Sending Output to Files

Redirection captures a command's output and sends it to a file instead of (or in addition to) the screen.

### Output Redirection (`>`)

echo "Hello, terminal!" > greeting.txt

Instead of printing "Hello, terminal!" on the screen, the > operator redirects it into a file called greeting.txt. If the file doesn't exist, it's created. If it does exist, it's overwritten — the old contents are gone.

Verify:

cat greeting.txt

Output:

Hello, terminal!

    ⚠️ Overwrite Warning

    > overwrites without asking. If greeting.txt had 500 lines of important data, echo "test" > greeting.txt would replace all 500 lines with a single line: "test". Always use >> (append) when you want to preserve existing content.

Append Redirection (>>)

echo "This is line two." >> greeting.txt

The >> operator adds to the end of the file without overwriting existing content. Now the file contains:

Hello, terminal!
This is line two.

This is the safe default for writing to files — you'll use >> far more often than >.
Redirecting Errors (2>)

Remember that stderr (stream 2) is separate from stdout (stream 1). By default, both appear on screen. But you can redirect them independently:

ls /home /nonexistent

Output on screen:

/home:
username  lost+found

ls: cannot access '/nonexistent': No such file or directory

The directory listing goes to stdout; the error goes to stderr. Both show on screen. Now let's separate them:

ls /home /nonexistent > output.txt 2> errors.txt

    output.txt contains the listing of /home (stdout)
    errors.txt contains the error message (stderr)
    Nothing appears on screen — both streams were captured

Redirecting Everything (&>)

To send both stdout and stderr to the same file:

ls /home /nonexistent &> everything.txt

Both the listing and the error go into everything.txt.
Discarding Output (/dev/null)

/dev/null is a special file that throws away everything sent to it — a black hole for data:

command > /dev/null

This runs command but discards its output. Why would you want that? When you only care whether the command succeeds (exit code), not what it prints:

find / -name "target.txt" > /dev/null 2>&1

This searches the entire file system for target.txt, discards all output and errors, and runs silently. You'd follow it with echo $? to check if it succeeded (0 = found, non-0 = not found).

The 2>&1 part means "redirect stderr to wherever stdout is going" — combining both streams before sending them to /dev/null. We'll see this pattern frequently in scripts (Chapter 27).
Input Redirection (<)

Just as > sends output to a file, < takes input from a file instead of the keyboard:

sort < shopping_list.txt

This feeds the contents of shopping_list.txt into the sort command, which outputs the lines in alphabetical order on screen. The file itself isn't modified — only read.

You can combine input and output redirection:

sort < shopping_list.txt > sorted_list.txt

Reads shopping_list.txt, sorts the lines, writes the result to sorted_list.txt. The original file is untouched.
The Pipe (|): Connecting Commands

The pipe is the most powerful operator in the terminal. It takes the stdout of one command and feeds it directly into the stdin of another — no temporary files needed.

command1 | command2

The | character (called "pipe" or "vertical bar") is typically located above the Enter key on US keyboards, accessed with Shift+.
Your First Pipeline

ls | less

ls lists the directory contents. Instead of printing to the screen, the output is piped into less. You can now scroll through the listing page by page — useful when a directory has hundreds of files.

Press q to exit less.
A More Useful Pipeline

ls -l | grep "Jul"

This lists files in long format, then pipes the result into grep, which filters for lines containing "Jul" (files modified in July). The output shows only July files — everything else is filtered out.
Chaining Multiple Pipes

Pipes can be chained indefinitely:

ls -l | grep "Jul" | sort -k5 -n

    ls -l — list files with details
    grep "Jul" — keep only lines containing "Jul"
    sort -k5 -n — sort by the 5th column (file size), numerically

The result: all July files, sorted smallest to largest. Each command processes the previous command's output.

    💡 The Factory Assembly Line

    Think of a pipeline as a factory assembly line. Raw material enters (the first command), gets processed (each subsequent command transforms it), and finished product comes out the end. Each station specializes in one thing:

        ls produces raw listing data
        grep filters it
        sort orders it
        wc counts it
        less displays it

    You combine stations to build exactly the processing you need.

grep: The Text Search Powerhouse

grep (Global Regular Expression Print) is the most-used search tool in Linux. It searches through text and prints lines that match a pattern.
Basic Search

grep "error" logfile.txt

Prints every line in logfile.txt that contains the word "error".
Searching in a Pipeline

ls -l | grep ".txt"

Lists only .txt files from the ls output.
Common grep Flags
Flag	What It Does	Example
-i	Case-insensitive search	grep -i "error" log.txt (matches Error, ERROR, error)
-v	Invert match (lines that DON'T contain pattern)	grep -v "comment" code.py (shows lines without "comment")
-n	Show line numbers	grep -n "todo" code.py (shows line numbers with matches)
-c	Count matching lines	grep -c "error" log.txt (outputs just a number)
-r	Recursive search through directories	grep -r "password" /etc/ (searches all files under /etc/)
-w	Match whole words only	grep -w "cat" file.txt (matches "cat" but not "catalog")
-l	Show only filenames that contain matches	grep -rl "TODO" . (lists files containing TODO)
-A N	Show N lines after each match	grep -A 3 "error" log.txt (shows error line + 3 lines after)
-B N	Show N lines before each match	grep -B 2 "error" log.txt (shows 2 lines before the error)
-C N	Show N lines around each match	grep -C 5 "error" log.txt (shows 5 lines before and after)
--color	Highlight matches in color	grep --color "error" log.txt (matches appear in red)
Practical grep Examples

Find all "error" entries in a log file:

grep -i "error" /var/log/syslog

Count how many errors occurred:

grep -c -i "error" /var/log/syslog

Find a process by name:

ps aux | grep firefox

Find all Python files containing "import os":

grep -rl "import os" ~/projects/

Show errors with surrounding context:

grep -C 3 "failed" /var/log/auth.log

    💡 What's ps aux?

    ps aux lists all running processes on the system. Piping it through grep lets you find specific programs — a technique you'll use constantly when checking if something is running or troubleshooting. We cover ps in detail in Chapter 16.

find: Finding Files by Properties

Where grep searches inside files, find searches for files by name, size, date, type, or other properties.
Basic Searches

Find files by name (case-sensitive):

find . -name "todo.txt"

Searches the current directory (.) and all subdirectories for a file named exactly todo.txt.

Case-insensitive name search:

find . -iname "todo.txt"

Matches todo.txt, TODO.TXT, Todo.Txt, etc.

Find all .jpg files:

find ~ -name "*.jpg"

Searches your entire home directory for JPEG files.
Search by Type

find . -type d -name "projects"

Finds directories (-type d) named "projects".

find . -type f -name "*.py"

Finds regular files (-type f) ending in .py.
Type Code	What It Matches
f	Regular files
d	Directories
l	Symbolic links
Search by Size

Find files larger than 100 MB:

find ~ -type f -size +100M

Find files smaller than 1 KB:

find ~ -type f -size -1k

Find files between 1 KB and 10 KB:

find ~ -type f -size +1k -size -10k

Unit	Meaning
c	Bytes
k	Kilobytes
M	Megabytes
G	Gigabytes
Search by Modification Time

Find files modified in the last 24 hours:

find ~ -type f -mtime 0

Find files modified in the last 7 days:

find ~ -type f -mtime -7

Find files not modified in the last 30 days:

find ~ -type f -mtime +30

Combining find with Other Commands

find is powerful alone but devastating when piped or combined with xargs (below). Example: find all .log files and search them for "error":

find /var/log -name "*.log" -exec grep -l "error" {} \;

The -exec flag runs a command on each file found. {} is a placeholder for the current file, and \; marks the end of the command. We'll keep -exec usage light in this chapter — it's covered more in Chapter 25 (Advanced Tools).
sort and uniq: Ordering and Deduplicating
sort

sort names.txt

Outputs the contents of names.txt in alphabetical order.

Common flags:
Flag	What It Does
-n	Sort numerically (so 10 comes after 9, not after 1)
-r	Reverse order (descending)
-u	Unique — remove duplicate lines during sort
-k N	Sort by column N (fields separated by whitespace)
-t ,	Use comma as field separator (for CSV files)

Examples:

sort -n sizes.txt          # Numerical sort
sort -rn scores.txt        # Highest score first
sort -t , -k 2 data.csv    # Sort CSV by second column

uniq

uniq removes adjacent duplicate lines. It only works on consecutive duplicates, so you almost always pipe sort into it first:

sort names.txt | uniq

This sorts all names alphabetically (bringing duplicates together), then removes adjacent duplicates. This is the standard deduplication pattern.

With counting:

sort names.txt | uniq -c

Outputs each unique name with a count of how many times it appeared:

      3 Alice
      1 Bob
      2 Charlie

Sorted by frequency (most common first):

sort names.txt | uniq -c | sort -rn

    Sort names alphabetically
    Count duplicates
    Sort the counts numerically, descending

Result:

      3 Alice
      2 Charlie
      1 Bob

This is an extremely common pipeline for analyzing log files, survey data, and any text with repeated entries.
wc: Counting Words, Lines, and Characters

wc todo.txt

Output:

12  87 540 todo.txt

Columns: 12 lines, 87 words, 540 characters, filename.

Individual counts:
Flag	Counts
-l	Lines only
-w	Words only
-c	Characters (bytes) only

Practical use in pipelines:

grep -i "error" /var/log/syslog | wc -l

Counts how many error lines exist in the system log. grep filters for error lines, wc -l counts them.

ls | wc -l

Counts how many files and directories are in the current location.
tee: Output to Screen AND File

Normally, when you redirect output to a file, it doesn't appear on screen. tee solves this — it splits output, sending it to both the screen AND a file simultaneously:

ls -l | tee listing.txt

You see the listing on screen AND it's saved to listing.txt.

Append mode:

ls -l | tee -a listing.txt

Appends to the file instead of overwriting.

tee is primarily used in pipelines where you want to inspect intermediate output while also capturing it. For example:

ls -l | tee listing.txt | grep ".txt"

    ls -l produces the full listing
    tee listing.txt saves the full listing to a file AND passes it through
    grep ".txt" filters for .txt files and shows them on screen

You get the full listing saved, and the filtered version on screen.
xargs: Building Command Arguments

Some commands don't accept piped input directly. xargs bridges this gap by taking piped input and converting it into command-line arguments.
The Problem

find . -name "*.tmp" | rm

This doesn't work. rm doesn't read from stdin — it needs filenames as arguments. The pipe sends filenames to rm's input stream, but rm ignores input streams.
The Solution

find . -name "*.tmp" | xargs rm

xargs takes the piped output (a list of filenames) and converts it into arguments for rm. The result: all .tmp files are deleted.
Safer Usage with Null Separator

Filenames can contain spaces, newlines, and other special characters that break xargs. To handle this safely, use find -print0 and xargs -0:

find . -name "*.tmp" -print0 | xargs -0 rm

    -print0 separates filenames with null characters instead of newlines
    -xargs -0 expects null-separated input

This ensures filenames with spaces, quotes, or special characters are handled correctly. Always use this pattern when combining find with xargs.
Practical Examples

Find all .py files and count lines in each:

find ~ -name "*.py" -print0 | xargs -0 wc -l

Find all .jpg files and copy them to a backup directory:

find ~ -name "*.jpg" -print0 | xargs -0 -I {} cp {} /backup/photos/

(The -I {} replaces {} with each filename — useful when the argument isn't at the end of the command.)

    💡 xargs Preview vs. Execution

    Before running an xargs command that could modify or delete files, preview what it would do:

    find . -name "*.tmp" -print0 | xargs -0 echo

    This prints the filenames xargs would pass to rm — letting you verify before executing. Replace echo with rm only when satisfied.

cut and tr: Simple Text Extraction
cut — Extract Columns or Fields

cut -d , -f 1,3 data.csv

Extracts fields 1 and 3 from a CSV file (comma-delimited).
Flag	What It Does
-d ,	Set delimiter (here: comma)
-f 1,3	Extract fields 1 and 3
-f 1-5	Extract fields 1 through 5
-c 1-10	Extract characters 1 through 10

Example with passwd file (colon-delimited):

cut -d : -f 1 /etc/passwd

Lists all usernames on the system (first field of each colon-separated line).
tr — Translate or Delete Characters

echo "Hello World" | tr 'a-z' 'A-Z'

Output:

HELLO WORLD

Converts lowercase to uppercase. tr works on characters, not files — it needs piped or redirected input:

cat names.txt | tr '\n' ',' 

Converts newlines to commas — joining all names into a single comma-separated line.

echo "remove spaces" | tr -d ' '

Output:

removespaces

The -d flag deletes the specified character.
Building Real Pipelines

Let's combine everything into practical, real-world scenarios:
Scenario 1: Find the 10 Largest Files in Your Home Directory

find ~ -type f -exec du -h {} + 2>/dev/null | sort -rh | head -10

    find ~ -type f — find all regular files in home directory
    -exec du -h {} + — get the size of each file
    2>/dev/null — discard permission errors
    sort -rh — sort by size, largest first (human-readable)
    head -10 — show only the top 10

Scenario 2: Count Occurrences of Each Word in a File

cat essay.txt | tr ' ' '\n' | sort | uniq -c | sort -rn | head -10

    cat essay.txt — read the file
    tr ' ' '\n' — replace spaces with newlines (one word per line)
    sort — alphabetize
    uniq -c — count each word
    sort -rn — sort by count, highest first
    head -10 — top 10 most frequent words

Scenario 3: Find All Files Modified Today and Back Them Up

find ~ -type f -mtime 0 -print0 | xargs -0 -I {} cp {} ~/backup_today/

    Find all files modified today
    Copy each one to a backup directory

Scenario 4: Analyze System Log Errors

grep -i "error" /var/log/syslog | cut -d ' ' -f 1-3 | sort | uniq -c | sort -rn | head -20

    Find all error lines in the system log
    Extract the timestamp portion (first 3 fields)
    Sort by timestamp
    Count errors per timestamp
    Sort by frequency
    Show top 20

Scenario 5: List All Installed Packages Sorted by Size

dpkg-query -W --showformat='${Installed-Size}\t${Package}\n' | sort -rn | head -20

    Query all installed packages with their sizes
    Sort by size, largest first
    Show top 20

Pipeline Construction Strategy

When building complex pipelines, work left to right, testing at each step:

    Start with the data source: ls -l
    Add a filter: ls -l | grep ".txt"
    Add a transformation: ls -l | grep ".txt" | sort -k5 -n
    Add output: ls -l | grep ".txt" | sort -k5 -n | head -10

After each addition, press Enter and check the output. If it looks wrong, you know the problem is in the most recently added command. This incremental approach prevents debugging an entire pipeline at once.

    💡 Debugging Pipelines

    If a pipeline produces unexpected results, insert tee /dev/stderr between stages to see intermediate output:

    find ~ -name "*.log" -print0 | xargs -0 grep "error" | tee /dev/stderr | wc -l

    The tee /dev/stderr shows the matched lines on screen while still passing them to wc -l for counting. This lets you see what's flowing through the pipe at each stage.

Redirection and Pipe Summary
Operator	What It Does
>	Redirect stdout to a file (overwrite)
>>	Append stdout to a file
2>	Redirect stderr to a file
2>&1	Redirect stderr to same destination as stdout
&>	Redirect both stdout and stderr to a file
<	Feed a file into a command's stdin
/dev/null	Discard anything sent to it
pipe	Connect stdout of one command to stdin of another
What's Next

You now possess the fundamental building blocks of terminal power: redirection to capture output, pipes to chain commands, grep to search, find to locate files, sort/uniq to organize, wc to count, tee to split output, and xargs to bridge gaps. These tools combine to accomplish tasks that would be tedious or impossible through a graphical interface.

In Chapter 16, we'll shift to system monitoring — using the terminal to check CPU usage, memory, disk space, running processes, and network activity. You'll learn top, htop, df, du, free, ps, and more — all using the piping and redirection skills you just acquired.
Try It Yourself

    Redirect output to a file. Run ls -la > my_listing.txt, then cat my_listing.txt to verify.

    Append to a file. Run date >> my_listing.txt several times, then cat my_listing.txt to see the appended timestamps.

    Build a pipeline. Run ls -l | grep ".txt" | wc -l to count how many .txt files you have.

    Search with grep. Create a text file with several lines. Use grep -i, grep -c, grep -n, and grep -v on it. Observe the different outputs.

    Use find. Run find ~ -name "*.jpg" -type f to find all JPEG files in your home directory. Then try find ~ -type f -size +10M to find large files.

    Sort and deduplicate. Create a file with duplicate names. Run sort names.txt | uniq -c | sort -rn to see frequency counts.

    Use tee. Run ls -l | tee output.txt | grep ".md" to save the full listing while filtering on screen.

    Build a real pipeline. Try Scenario 1 from this chapter: find ~ -type f -exec du -h {} + 2>/dev/null | sort -rh | head -10 — find the 10 largest files in your home directory.

    Combine find with grep. Run grep -rl "Linux" ~/Documents/ to find all files in your Documents folder that mention "Linux."

    ✅ Chapter 15 Complete You can redirect output to files, append without overwriting, discard errors, pipe commands together, search file contents with grep, find files by name/size/date, sort and deduplicate text, count lines and words, split output with tee, and bridge gaps with xargs. You can build multi-stage pipelines to process data in ways no graphical tool can match. In Chapter 16, we'll monitor your system's health and performance from the terminal.
