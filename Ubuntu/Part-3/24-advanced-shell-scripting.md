# Chapter 24: Advanced Shell Scripting

## Beyond the Basics

In Chapter 19, you wrote your first scripts — variables, simple conditionals, loops, and functions. Those fundamentals handle a surprising amount of automation. But as your scripts grow more complex, you'll need more powerful tools: arrays, case statements for multi-branch logic, robust error handling, advanced string manipulation, file processing, scheduled execution, and the discipline of writing scripts that fail gracefully.

This chapter takes your scripting skills from "works on my machine" to "reliable and maintainable." Every concept includes a practical example you can adapt for real use.

## Arrays

Arrays store multiple values under a single variable name. Instead of creating `file1`, `file2`, `file3`, you create one array called `files` and access elements by index.

### Indexed Arrays

**Creating an array:**
```bash
fruits=("apple" "banana" "cherry" "date")

Accessing elements (zero-indexed):

echo ${fruits[0]}    # apple
echo ${fruits[1]}    # banana
echo ${fruits[2]}    # cherry

Accessing all elements:

echo ${fruits[@]}    # apple banana cherry date

Number of elements:

echo ${#fruits[@]}   # 4

Getting all indices:

echo ${!fruits[@]}   # 0 1 2 3

Adding an element:

fruits[4]="elderberry"

Removing an element:

unset fruits[1]      # removes "banana"

Iterating over an array:

for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

    💡 Always Quote Array Expansions

    Use "${fruits[@]}" (with quotes), not ${fruits[@]} (without). Without quotes, elements containing spaces are split — an array element like "dragon fruit" becomes two words. Quoting preserves each element as a single unit.

Building Arrays from Command Output

A common pattern — populate an array from a command:

#!/bin/bash

# Store all .log files in /var/log as an array
mapfile -t log_files < <(ls /var/log/*.log 2>/dev/null)

echo "Found ${#log_files[@]} log files"
for f in "${log_files[@]}"; do
    echo "  - $f ($(du -h "$f" | cut -f1))"
done

mapfile (also called readarray) reads lines from stdin into an array, one element per line. The -t flag strips trailing newlines. The < <(...) syntax is process substitution — it runs the command in a subshell and feeds its output to mapfile.
Associative Arrays (Key-Value)

Bash 4.0+ supports associative arrays — dictionaries with string keys:

#!/bin/bash

declare -A ages

ages["alice"]=30
ages["bob"]=25
ages["charlie"]=35

echo "Alice is ${ages[alice]} years old"
echo "Bob is ${ages[bob]} years old"

Iterating over an associative array:

for name in "${!ages[@]}"; do
    echo "$name is ${ages[$name]} years old"
done

Practical example — file extension counts:

#!/bin/bash

declare -A counts

for file in *; do
    if [ -f "$file" ]; then
        ext="${file##*.}"
        ((counts["$ext"]++))
    fi
done

for ext in "${!counts[@]}"; do
    echo ".$ext: ${counts[$ext]} files"
done

Output:

.txt: 12 files
.pdf: 3 files
.jpg: 8 files
.sh: 5 files

Case Statements

When you have multiple conditions to check against a single value, if/elif/else chains become unwieldy. The case statement is cleaner:

#!/bin/bash

echo "What would you like to do?"
echo "1) Check disk space"
echo "2) Check memory"
echo "3) Check running processes"
echo "4) Exit"
read choice

case "$choice" in
    1)
        df -h
        ;;
    2)
        free -h
        ;;
    3)
        ps aux --sort=-%cpu | head -10
        ;;
    4|q|quit|exit)
        echo "Goodbye!"
        exit 0
        ;;
    *)
        echo "Invalid choice: $choice"
        ;;
esac

Case Pattern Matching

Case patterns support wildcards and | (OR):

#!/bin/bash

file="$1"

case "$file" in
    *.jpg|*.jpeg|*.png|*.gif|*.bmp|*.svg)
        echo "Image file"
        ;;
    *.mp4|*.avi|*.mkv|*.webm)
        echo "Video file"
        ;;
    *.mp3|*.flac|*.wav|*.ogg)
        echo "Audio file"
        ;;
    *.txt|*.md|*.pdf|*.docx)
        echo "Document file"
        ;;
    *)
        echo "Unknown file type: $file"
        ;;
esac

Pattern	Matches
*.txt	Anything ending in .txt
a|b	"a" or "b"
[0-9]	Any single digit
?	Any single character
*)	Everything (default case)
Functions: Advanced Techniques
Local Variables

By default, variables in Bash are global — even inside functions. Use local to restrict scope:

#!/bin/bash

calculate() {
    local result=0
    result=$(( $1 + $2 ))
    echo $result
}

sum=$(calculate 5 10)
echo "Sum is: $sum"       # Sum is: 15

# 'result' is not accessible outside the function
echo "$result"            # (empty — local scope)

    💡 Always Use local in Functions

    Without local, a variable set inside a function leaks into the global scope, potentially overwriting variables elsewhere in your script. This is a very common source of bugs. Make local your default for all function variables.

Return Values via Echo

Bash functions can't return arbitrary values directly — the return statement only returns an exit code (0-255). To return a string or number, use echo and capture the output:

get_user_home() {
    local user="$1"
    local home_dir
    
    home_dir=$(getent passwd "$user" | cut -d: -f6)
    
    if [ -z "$home_dir" ]; then
        echo ""
        return 1    # failure
    fi
    
    echo "$home_dir"
    return 0        # success
}

# Usage
if home=$(get_user_home "john"); then
    echo "John's home: $home"
else
    echo "User not found"
fi

Functions with Multiple Arguments

create_backup() {
    local source="$1"
    local dest="$2"
    local stamp
    
    stamp=$(date +%Y%m%d_%H%M%S)
    
    if [ ! -e "$source" ]; then
        echo "Error: Source '$source' does not exist" >&2
        return 1
    fi
    
    tar czf "${dest}/${stamp}_backup.tar.gz" "$source"
    echo "Backup created: ${dest}/${stamp}_backup.tar.gz"
    return 0
}

# Usage
create_backup ~/Documents ~/Backups

Notice the >&2 — this sends error messages to stderr (standard error), not stdout. When callers capture the function's output with $( ), error messages still appear on screen instead of being captured.
Error Handling

By default, Bash scripts keep going even when commands fail. A cp that fails silently, an apt install that errors out — the script continues, potentially causing cascading problems. Robust scripts handle errors explicitly.
set -e — Exit on Error

#!/bin/bash
set -e

echo "Creating directory..."
mkdir /tmp/myproject

echo "Copying files..."
cp /nonexistent/file.txt /tmp/myproject/    # This fails

echo "Done!"    # This line never executes — script exits at the failed cp

With set -e, the script aborts immediately when any command returns a non-zero exit code. The failing line is the last thing executed.
set -u — Error on Undefined Variables

#!/bin/bash
set -u

echo "Hello, $USERNAME"    # Works if USERNAME is set

echo "Value: $UNDEFINED_VAR"    # Script exits here — UNDEFINED_VAR is not set

Prevents typos in variable names from silently expanding to empty strings.
set -o pipefail — Fail on Pipeline Errors

#!/bin/bash
set -o pipefail

# If 'grep' fails (no match), the pipeline fails
false | true
echo "Without pipefail, this prints because 'true' is the last command"

set -o pipefail
false | true    # Now the pipeline fails because 'false' failed

Without pipefail, a pipeline's exit code is the exit code of the last command. If an earlier command in the pipe fails but the last succeeds, the failure is hidden. pipefail makes the pipeline fail if any component fails.
The Safe Script Header

Combine all three for maximum safety:

#!/bin/bash
set -euo pipefail

This single line makes your script:

    Exit on any error (-e)
    Error on undefined variables (-u)
    Fail pipelines if any component fails (-o pipefail)

    💡 The Golden Trio

    Add set -euo pipefail to the top of every script you write. It catches errors early, prevents silent failures, and makes debugging dramatically easier. Beginners and experts alike use this combination — it's the single most impactful quality improvement for shell scripts.

    The only downside: sometimes you expect a command to fail (e.g., grep finding no matches) and need to handle it. In that case, wrap the command explicitly:

    if grep -q "pattern" file.txt; then
        echo "Found"
    else
        echo "Not found"
    fi

    The if construct prevents set -e from triggering because the command's failure is "handled" by the conditional.

trap — Cleanup on Exit

Sometimes you need to clean up temporary files or restore state when a script exits — whether it exits successfully, fails, or is interrupted with Ctrl+C. trap lets you define actions for these situations:

#!/bin/bash
set -euo pipefail

# Create a temporary directory
TEMP_DIR=$(mktemp -d)

# Define cleanup function
cleanup() {
    echo "Cleaning up temporary files..."
    rm -rf "$TEMP_DIR"
}

# Register cleanup to run on exit, interrupt, or error
trap cleanup EXIT INT TERM

echo "Working in: $TEMP_DIR"
echo "Doing important work..."
# ... script body ...

# When the script exits (normally or due to error),
# cleanup() runs automatically

Signal	When
EXIT	Script exits (any reason)
INT	Ctrl+C pressed
TERM	Process terminated (e.g., by kill)
ERR	A command fails (with set -e)

Debugging trap:

trap 'echo "Error on line $LINENO"' ERR

Prints the line number whenever a command fails — extremely useful for debugging scripts with set -e enabled.
Advanced String Manipulation

Bash has powerful built-in string operations — no need to call sed, awk, or cut for simple transforms.
Parameter Expansion Reference
Syntax	What It Does
${#var}	Length of the string
${var:-default}	Use default if var is empty
${var:=default}	Set var to default if empty
${var:offset}	Substring from offset to end
${var:offset:length}	Substring of given length
${var#pattern}	Remove shortest match from front
${var##pattern}	Remove longest match from front
${var%pattern}	Remove shortest match from end
${var%%pattern}	Remove longest match from end
${var/old/new}	Replace first occurrence
${var//old/new}	Replace all occurrences
${var^pattern}	Uppercase first matching char
${var^^}	Uppercase entire string
${var,}	Lowercase first char
${var,,}	Lowercase entire string
Practical Examples

Extract filename and extension:

filepath="/home/john/Documents/report.pdf"

filename="${filepath##*/}"           # report.pdf
basename="${filename%.*}"            # report
extension="${filename##*.}"          # pdf
dirname="${filepath%/*}"             # /home/john/Documents

echo "Directory:  $dirname"
echo "Filename:   $filename"
echo "Base name:  $basename"
echo "Extension:  $extension"

Replace text in a variable:

text="The quick brown fox"
echo "${text/quick/slow}"          # The slow brown fox
echo "${text//o/0}"               # The quick br0wn f0x

Provide default values:

config_file="${1:-/etc/default/myapp.conf}"
echo "Using config: $config_file"

If no argument is passed ($1 is empty), defaults to the specified path. Extremely useful for configurable scripts.

Substring extraction:

phone="+1-555-123-4567"
area="${phone:3:3}"               # 555
echo "Area code: $area"

3:3 means "start at offset 3, take 3 characters."

Case conversion:

name="john doe"
echo "${name^^}"                  # JOHN DOE
echo "${name,,}"                  # john doe
echo "${name^}"                   # John doe (first char only)

Reading and Processing Files
Reading a File Line by Line

#!/bin/bash

while IFS= read -r line; do
    echo "Processing: $line"
done < /etc/passwd

This is the safe, standard pattern for reading files line by line:

    IFS= — preserves leading/trailing whitespace
    read -r — prevents backslash interpretation (handles paths with backslashes correctly)
    done < file — redirects the file as input to the loop

Reading CSV Files

#!/bin/bash
# Parse a simple CSV (no quoted commas)

while IFS=',' read -r name age email; do
    echo "Name: $name, Age: $age, Email: $email"
done < contacts.csv

Sets IFS=',' so read splits each line on commas. Each field is assigned to a variable.
Here Documents and Here Strings

Here Document (multi-line input):

cat << EOF
Configuration file for $APP_NAME
================================
Server port: $PORT
Database: $DB_NAME
Log level: $LOG_LEVEL
EOF

A here document (<<) feeds multiple lines as input to a command. The delimiter (EOF) marks the end. Variables inside are expanded by default.

Here document without variable expansion (literal):

cat << 'EOF'
This text has $variables that won't be expanded
because the delimiter is quoted.
EOF

Here String (single-line input):

grep "error" <<< "$log_output"

The <<< syntax feeds a string as stdin to a command. Cleaner than echo "$log_output" | grep "error".
Writing to Files from Scripts

#!/bin/bash

config_file="/tmp/myapp.conf"

cat > "$config_file" << EOF
# Generated on $(date)
HOST=$(hostname)
USER=$USER
PORT=8080
EOF

echo "Config written to $config_file"
cat "$config_file"

This pattern — using cat with a here document — generates config files programmatically. The > redirects the entire output to the file.
Command-Line Argument Parsing with getopts

For scripts that need flags and options (like -v for verbose, -f file for input), use getopts:

#!/bin/bash

verbose=false
input_file=""
output_dir="."

# Parse options
while getopts ":vf:o:" opt; do
    case "$opt" in
        v)  verbose=true ;;
        f)  input_file="$OPTARG" ;;
        o)  output_dir="$OPTARG" ;;
        \?) echo "Invalid option: -$OPTARG" >&2; exit 1 ;;
        :)  echo "Option -$OPTARG requires an argument." >&2; exit 1 ;;
    esac
done

# Shift parsed options away
shift $((OPTIND - 1))

echo "Verbose: $verbose"
echo "Input: $input_file"
echo "Output: $output_dir"
echo "Remaining args: $@"

Usage:

./myscript.sh -v -f input.txt -o /tmp/output arg1 arg2

The getopts string :vf:o: means:

    v — flag (no argument)
    f: — option requiring an argument
    o: — option requiring an argument
    Leading : — enables silent error reporting (you handle errors yourself)

Scheduling with Cron

cron is Linux's built-in task scheduler. It runs commands at specified times — hourly, daily, weekly, or any custom schedule.
Editing the Crontab

crontab -e

Opens your user's crontab file in the default editor (nano, unless you changed it with select-editor in Chapter 19).
Cron Syntax

# minute  hour  day-of-month  month  day-of-week  command
# 0-59    0-23  1-31           1-12   0-7 (0/7=Sun)

Each field is a number (or range, or list). An asterisk (*) means "every."

Examples:
Schedule	Cron Entry
Every day at 3 AM	0 3 * * * /home/john/bin/backup.sh
Every hour	0 * * * * /home/john/bin/check.sh
Every 15 minutes	*/15 * * * * /home/john/bin/monitor.sh
Every Monday at 9 AM	0 9 * * 1 /home/john/bin/report.sh
First of every month at midnight	0 0 1 * * /home/john/bin/cleanup.sh
Weekdays at 5 PM	0 17 * * 1-5 /home/john/bin/eod.sh
Every Sunday at 2 AM	0 2 * * 0 /home/john/bin/weekly.sh
Special Cron Shortcuts

Some cron implementations support shorthand:
Shortcut	Meaning
@daily / @midnight	Every day at midnight
@hourly	Every hour
@weekly	Every Sunday at midnight
@monthly	First of every month at midnight
@yearly / @annually	January 1st at midnight
@reboot	Once at startup
Managing Cron Jobs

View your crontab:

crontab -l

Remove all cron jobs:

crontab -r

Edit another user's crontab (requires sudo):

sudo crontab -u mary -e

    ⚠️ Cron Environment Limitations

    Cron jobs run in a minimal environment — they don't have your full $PATH, aliases, or interactive shell settings. Always use absolute paths for both commands and files in cron jobs:

    # Bad — may fail in cron
    0 3 * * * backup.sh >> backup.log 2>&1

    # Good — absolute paths
    0 3 * * * /home/john/bin/backup.sh >> /home/john/logs/backup.log 2>&1

    Also redirect output to a log file — cron emails output by default, which may not be configured. The >> log 2>&1 pattern captures both stdout and stderr.

systemd timers — The Modern Alternative

systemd offers timers as an alternative to cron. They're more powerful but more complex:

systemctl list-timers

Shows all active timers (including those Ubuntu pre-configures for system maintenance). You won't need to create your own timers as a beginner — cron handles personal scheduling just fine.
Putting It All Together: Three Practical Scripts
Script 1: Automated Backup with Logging

#!/bin/bash
set -euo pipefail

# backup-home.sh — Compressed backup of important directories
# Usage: backup-home.sh [destination]

SOURCE="${HOME}"
DEST="${1:-${HOME}/Backups}"
STAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${DEST}/backup_${STAMP}.tar.gz"
LOG_FILE="${DEST}/backup_${STAMP}.log"

mkdir -p "$DEST"

echo "=== Backup started: $(date) ===" | tee "$LOG_FILE"
echo "Source: $SOURCE" | tee -a "$LOG_FILE"
echo "Destination: $BACKUP_FILE" | tee -a "$LOG_FILE"

# Trap to log errors
trap 'echo "ERROR: Backup failed on line $LINENO" | tee -a "$LOG_FILE"' ERR

# Perform the backup
tar czf "$BACKUP_FILE" \
    --exclude="*.cache" \
    --exclude="Backups" \
    --exclude=".Trash" \
    "$SOURCE/Documents" \
    "$SOURCE/Pictures" \
    "$SOURCE/Projects" \
    2>&1 | tee -a "$LOG_FILE"

# Verify the backup
if [ -f "$BACKUP_FILE" ]; then
    SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
    echo "Backup completed: $BACKUP_FILE ($SIZE)" | tee -a "$LOG_FILE"
    
    # Keep only the last 5 backups
    ls -t "${DEST}"/backup_*.tar.gz | tail -n +6 | xargs -r rm
    echo "Old backups cleaned up" | tee -a "$LOG_FILE"
else
    echo "ERROR: Backup file not created!" | tee -a "$LOG_FILE"
    exit 1
fi

echo "=== Backup finished: $(date) ===" | tee -a "$LOG_FILE"

This script demonstrates:

    set -euo pipefail for safety
    Default arguments (${1:-fallback})
    trap for error logging
    tee to write to both screen and log file
    Retention management (keeping only 5 backups)
    Exclusion patterns with tar

Schedule it with cron:

0 3 * * * /home/john/bin/backup-home.sh >> /home/john/logs/cron-backup.log 2>&1

Script 2: Network Diagnostic Tool

#!/bin/bash
set -euo pipefail

# netdiag.sh — Network diagnostic tool
# Usage: netdiag.sh [--full] [hostname]

FULL=false
TARGET="${2:-google.com}"

# Parse --full flag
if [ "${1:-}" = "--full" ]; then
    FULL=true
    TARGET="${2:-google.com}"
fi

echo "========================================"
echo "  Network Diagnostics"
echo "  Target: $TARGET"
echo "  $(date)"
echo "========================================"
echo ""

echo "--- INTERFACE ---"
ip -br a
echo ""

echo "--- DEFAULT ROUTE ---"
ip route | grep default
echo ""

echo "--- DNS RESOLUTION ---"
if host "$TARGET" > /dev/null 2>&1; then
    echo "✓ DNS resolution: OK"
    host "$TARGET" | head -3
else
    echo "✗ DNS resolution: FAILED"
    echo "Attempting direct IP..."
fi
echo ""

echo "--- PING TEST ---"
if ping -c 4 -W 3 "$TARGET" > /dev/null 2>&1; then
    echo "✓ Ping: OK"
    ping -c 4 "$TARGET" | tail -2
else
    echo "✗ Ping: FAILED"
fi
echo ""

if [ "$FULL" = true ]; then
    echo "--- TRACEROUTE ---"
    traceroute -m 15 "$TARGET" 2>/dev/null || echo "traceroute not available"
    echo ""

    echo "--- LISTENING PORTS ---"
    ss -tln
    echo ""
fi

echo "--- FIREWALL STATUS ---"
sudo ufw status 2>/dev/null || echo "ufw not available"
echo ""

echo "========================================"
echo "  Diagnostics complete."
echo "========================================"

This script demonstrates:

    Optional flags with argument parsing
    Conditional execution (if command; then)
    Functions returning success/failure
    Tilde expansion and command substitution
    Extension based on flags (--full mode)

Script 3: Package Update Reporter

#!/bin/bash
set -euo pipefail

# update-report.sh — Check for available updates and report
# Suitable for cron scheduling

OUTPUT_FILE="/tmp/update-report.txt"
EMAIL_RECIPIENT=""  # Set to receive email (requires mailutils)

# Refresh package lists
sudo apt update -qq > /dev/null 2>&1

# Count available updates
UPGRADABLE=$(apt list --upgradable 2>/dev/null | grep -c upgradable)
SECURITY=$(apt list --upgradable 2>/dev/null | grep -ci security)

{
    echo "=== System Update Report ==="
    echo "Host: $(hostname)"
    echo "Date: $(date)"
    echo "Total upgradable packages: $UPGRADABLE"
    echo "Security updates: $SECURITY"
    echo ""
    
    if [ "$UPGRADABLE" -gt 0 ]; then
        echo "Upgradable packages:"
        apt list --upgradable 2>/dev/null | grep upgradable | head -20
    else
        echo "System is up to date."
    fi
    
    echo ""
    echo "=== End of report ==="
} | tee "$OUTPUT_FILE"

# Optional: send via email
if [ -n "$EMAIL_RECIPIENT" ]; then
    mail -s "Update Report for $(hostname)" "$EMAIL_RECIPIENT" < "$OUTPUT_FILE"
fi

Schedule weekly:

0 8 * * 1 /home/john/bin/update-report.sh

Runs every Monday at 8 AM, generates a report of pending updates.
Script Writing Best Practices

    Start with the shebang. #!/bin/bash on every script.

    Use set -euo pipefail. Catch errors early.

    Comment generously. Future you (or someone else) will read this.

    Use meaningful variable names. source_dir not sd.

    Quote all variables. "$var" not $var. Handles spaces in filenames.

    Use local in functions. Prevents scope leakage.

    Provide usage text. Help users (including yourself) understand how to run the script:

    usage() {
        echo "Usage: $0 [options]"
        echo "  -f FILE    Input file"
        echo "  -o DIR     Output directory (default: .)"
        echo "  -v         Verbose output"
        echo "  -h         Show this help"
    }

    Handle missing dependencies. Check for required commands:

    for cmd in tar gzip curl; do
        if ! command -v "$cmd" > /dev/null; then
            echo "Error: $cmd is not installed" >&2
            exit 1
        fi
    done

    Redirect errors to stderr. Use >&2 for error messages so they don't mix with normal output.

    Make scripts idempotent. Running the script twice should be safe — use mkdir -p, check before overwriting, etc.

    Test with edge cases. Empty arguments, missing files, special characters in filenames.

    Version control your scripts. Keep them in a Git repository (covered in Chapter 25).

Shell Scripting Cheat Sheet (Advanced)
Concept	Syntax
Indexed array	arr=("a" "b" "c")
Access element	${arr[0]}
All elements	"${arr[@]}"
Array length	${#arr[@]}
All indices	${!arr[@]}
Associative array	declare -A arr; arr[key]=val
Case statement	case "$x" in pat) ... ;; esac
Local variable	local var="value"
String length	${#var}
Default value	${var:-default}
Remove from front	${var#pattern} / ${var##pattern}
Remove from end	${var%pattern} / ${var%%pattern}
Replace	${var/old/new} / ${var//old/new}
Uppercase	${var^^}
Lowercase	${var,,}
Substring	${var:offset:length}
Exit on error	set -e
Error on undefined	set -u
Fail on pipe error	set -o pipefail
Safe trio	set -euo pipefail
Cleanup on exit	trap func EXIT
Error line number	$LINENO
Read file line by line	while IFS= read -r line; do ...; done < file
Here document	cat << EOF ... EOF
Here string	cmd <<< "$string"
Process substitution	cmd < <(other_cmd)
getopts	while getopts ":vf:o:" opt; do ...
Cron (daily 3AM)	0 3 * * * /path/to/script.sh
Check command exists	command -v "$cmd" > /dev/null
What's Next

You now have a serious scripting toolkit: arrays, case statements, advanced functions, robust error handling with set -euo pipefail and trap, string manipulation via parameter expansion, file processing, argument parsing with getopts, scheduled execution with cron, and three complete practical scripts. These skills translate directly to professional Linux administration and development.

In Chapter 25, we'll shift gears to customization — making your Ubuntu desktop truly yours. We'll cover GNOME Shell extensions, themes, keyboard shortcuts, Tweaks tool, dock configuration, and performance tuning.
Try It Yourself

    Experiment with arrays. Create a script that stores your three favorite programs in an indexed array and prints them with numbers. Then create an associative array mapping usernames to home directories.

    Write a case statement. Create a script that takes a file extension as an argument and outputs the file type using a case statement (image, video, audio, document, unknown).

    Add error handling. Take any script you wrote in Chapter 19 and add set -euo pipefail to the top. Add a trap that prints the line number on error. Test by intentionally causing an error (e.g., cat /nonexistent).

    Practice string manipulation. Write a script that takes a filepath as an argument and extracts the directory, filename, base name, and extension using only parameter expansion (no cut, sed, or awk).

    Parse a CSV file. Create a simple contacts.csv with name,email,phone columns. Write a script that reads it line by line and prints formatted output.

    Schedule a cron job. Create a simple script that logs the current time and system uptime to a file. Schedule it to run every 5 minutes with crontab -e. Check the log file after 15 minutes. Remove the cron job when done with crontab -r.

    Build a real script. Choose one of the three practical scripts from this chapter and adapt it to your needs. Install it in ~/bin/, make it executable, and test it thoroughly.

    Write a script with getopts. Create a script that accepts -v (verbose), -f FILE (input file), and -h (help) options. Print the parsed values.

    ✅ Chapter 24 Complete You can use arrays (indexed and associative), case statements, functions with local variables and echo-based return values, error handling with set -euo pipefail and trap, parameter expansion for string manipulation, file processing with safe line-by-line reading, getopts for command-line argument parsing, and cron for scheduled execution. You have three production-quality script templates. In Chapter 25, we'll customize your Ubuntu desktop.

