# Chapter 19: Terminal Text Editors — Mastering Vim, Nano, and Emacs

## Why Edit Text in the Terminal?

You've been using GNOME Text Editor for writing notes and editing configuration files. It's comfortable, graphical, and familiar. So why learn terminal-based text editors?

Because **most of the files that matter on your system live in places where GUI editors are inconvenient or impossible to use**:

- `/etc/fstab` — system mount configuration
- `/etc/sudoers` — privilege escalation rules (must use `visudo`, which launches a terminal editor)
- `/etc/dnf/dnf.conf` — package manager settings
- SSH configuration on remote servers with no GUI
- Shell scripts you're writing and testing in the same terminal
- Cron jobs edited via `crontab -e` (opens a terminal editor)
- Git commit messages (Git opens a terminal editor)
- System rescue scenarios where the GUI doesn't start

Every one of these tasks requires a terminal text editor. You could install a GUI editor on every remote server, but that's impractical and insecure. Terminal editors are lightweight, universally available, and work over SSH connections with zero overhead.

This chapter teaches three editors with distinct philosophies:

| Editor | Philosophy | Learning Curve | Power Level |
|---|---|---|---|
| **Nano** | Simplicity first — like Notepad in terminal | Gentle | Basic |
| **Vim** | Modal editing — different modes for navigation vs. insertion | Steep but rewarding | Very high |
| **Emacs** | Extensible environment — an OS inside an editor | Steep but vast | Extremely high |

You don't need to master all three. Most Linux users pick one and go deep. But understanding the landscape helps you choose wisely — and recognize which editor a tutorial or system tool is using when it pops open unexpectedly.

---

## Table of Contents

| Topic | Section |
|---|---|
| Nano: The Gentle Entry Point | 1 |
| Vim: The Modal Powerhouse | 2 |
| Emacs: The Extensible Universe | 3 |
| Other Notable Terminal Editors | 4 |
| Choosing Your Editor | 5 |
| The $EDITOR Environment Variable | 6 |
| Configuration and Customization | 7 |

---

## 1. Nano: The Gentle Entry Point

### What Is Nano?

**GNU Nano** is a small, friendly text editor for the terminal. It was created in 1999 as a free replacement for Pico (the editor bundled with the Pine email client). Nano's design philosophy is the opposite of Vim and Emacs: **be obvious, be simple, show your shortcuts on screen.**

Fedora 44 includes Nano by default. No installation needed:

```bash
nano --version
# GNU nano version 8.x

Opening and Creating Files

# Open an existing file:
nano ~/.bashrc

# Create a new file (just start typing — file is created on save):
nano ~/notes.txt

# Open at a specific line number:
nano +42 /etc/fstab

# Open multiple files (switch with Ctrl+X then the filename):
nano file1.txt file2.txt

The Interface

When you open Nano, the screen looks like this:

  GNU nano 8.x                    notes.txt

This is where your text appears.
You can type freely, like any text editor.
Lines wrap automatically at the terminal width.



^G Get Help     ^O Write Out    ^W Where Is     ^K Cut          ^T Execute
^X Exit         ^R Read File    ^\ Replace      ^U Uncut        ^J Justify

The bottom two lines show available commands. The ^ symbol means Ctrl. So:
Shortcut	Action
^G (Ctrl+G)	Get help
^X (Ctrl+X)	Exit
^O (Ctrl+O)	Write out (save)
^W (Ctrl+W)	Where is (search)
^\ (Ctrl+)	Search and replace
^K (Ctrl+K)	Cut current line
^U (Ctrl+U)	Uncut (paste cut text)
^J (Ctrl+J)	Justify paragraph
^R (Ctrl+R)	Read file (insert another file's contents)
^T (Ctrl+T)	Execute a command (spell check, etc.)
Essential Workflow
Saving Files

# Press Ctrl+O to save:
^O
# File Name to Write: /home/user/notes.txt
# Press Enter to confirm

# If you want to save as a different filename:
^O
# Edit the path, then press Enter

Exiting

# Press Ctrl+X to exit:
^X
# If unsaved changes exist:
# Save modified buffer?  Y Yes, N No, ^C Cancel
# Press Y to save, N to discard, Ctrl+C to cancel exit

Searching

# Search forward:
^W
# Search: keyword
# Press Enter — cursor jumps to first match

# Search again (repeat last search):
^W, then Enter immediately

# Search and replace:
^\
# Search (to replace): old_word
# Replace with: new_word
# Press Enter, then choose: Y (replace this one), A (replace all), N (skip)

Cut, Copy, and Paste

Nano doesn't have a traditional clipboard. Instead:

# Cut the ENTIRE current line:
^K

# Cut multiple lines (press ^K repeatedly):
^K ^K ^K    # Cuts three lines into the cut buffer

# Paste (uncut) the cut buffer at cursor position:
^U

# To COPY instead of cut: cut (^K) then immediately uncut (^U)
# The text stays in the buffer — move cursor and paste again:
^U

Moving Around
Key	Movement
Arrow keys	Move cursor one character
Ctrl+A	Beginning of line
Ctrl+E	End of line
Ctrl+Y (or PgUp)	Previous page
Ctrl+V (or PgDn)	Next page
Ctrl+_	Go to specific line and column
Going to a Specific Line

^T (when in search context) or:
Ctrl+_
# Enter line number, column number: 42,1

Nano Configuration

Customize Nano via ~/.nanorc:

nano ~/.nanorc

Recommended beginner settings:

# Enable syntax highlighting:
include "/usr/share/nano/*.nanorc"

# Show line numbers:
set linenumbers

# Enable mouse support (click to position cursor):
set mouse

# Auto-indent new lines:
set autoindent

# Smart home key (jumps to first non-whitespace):
set smarthome

# Soft wrapping (don't break long lines):
set softwrap

# Display help text at bottom:
set help

# Enable undo/redo:
set multibuffer
set undo

Apply immediately (Nano reads ~/.nanorc on every launch):

nano ~/.bashrc    # Now with syntax highlighting, line numbers, mouse support

When Nano Isn't Enough

Nano excels at quick edits — changing a config value, jotting notes, fixing a typo. But it lacks:

    Split windows
    Macro recording
    Syntax-aware code completion
    Powerful multi-file workflows
    Extensibility through plugins

For serious coding, writing, or system administration work, Vim or Emacs offer substantially more power. Nano remains your "quick and dirty" fallback — and that's a valuable role.

    💡 Nano as Training Wheels

    There's no shame in starting with Nano. Many Linux veterans keep Nano as their default $EDITOR for trivial tasks. The goal isn't to abandon Nano — it's to ADD capabilities as your needs grow. Nano for quick edits, Vim or Emacs for deep work.

2. Vim: The Modal Powerhouse
The Modal Revolution

Vim (Vi IMproved) operates on a principle that makes it unique among text editors: modes. In Nano, Emacs, or any GUI editor, every keystroke inserts text. In Vim, keystrokes are COMMANDS unless you explicitly enter insert mode.

This sounds bizarre until you experience it. Then it feels like a superpower.

Think of it like driving a car with a manual transmission:

    Nano/Notepad = automatic transmission: one pedal does one thing (typing inserts text)
    Vim = manual transmission: you shift gears (change modes) to access different capabilities

The trade-off: steeper learning curve, dramatically higher efficiency once mastered.
Vim's Modes

Vim has several modes, but three matter most:
Mode	Purpose	How to Enter	How to Exit
Normal	Navigate, delete, copy, paste, execute commands	Press Esc	(already here by default)
Insert	Type text into the file	Press i, a, o, I, A, O	Press Esc
Command-line	Save, quit, search, configure	Press : (colon)	Press Esc or Enter

┌──────────────────────────────────────────────────┐
│                    VIM MODES                     │
│                                                  │
│   ┌─────────┐   i/a/o   ┌─────────┐             │
│   │ NORMAL  │ ────────► │ INSERT  │             │
│   │ (default)│ ◄──────── │ (typing)│             │
│   └────┬────┘    Esc    └─────────┘             │
│        │                                         │
│        │ :  (colon)                             │
│        ▼                                         │
│   ┌─────────┐   Enter  ┌─────────┐              │
│   │COMMAND  │ ────────►│ NORMAL  │              │
│   │  -LINE  │          │ (back to│              │
│   │ (:)     │          │  work)  │              │
│   └─────────┘          └─────────┘              │
└──────────────────────────────────────────────────┘

Installing and Verifying

Fedora 44 includes Vim by default:

vim --version | head -3
# VIM - Vi IMproved 9.x (2024)

If you need the full version (some systems ship vim-minimal):

sudo dnf install vim-enhanced

Opening Files

# Open a file:
vim ~/.bashrc

# Open at a specific line:
vim +42 /etc/fstab

# Open multiple files in tabs:
vim -p file1.py file2.py file3.py

# Open multiple files in split windows:
vim -o file1.py file2.py    # Horizontal splits
vim -O file1.py file2.py    # Vertical splits

# Open read-only (view mode):
view /var/log/messages
# Or:
vim -R /etc/passwd

Normal Mode: Movement

In Normal mode, every key is a movement or action command. This is where Vim's power lives:
Basic Movement
Key	Action	Analogy
h	Move left	Like left arrow
j	Move down	Like down arrow
k	Move up	Like up arrow
l	Move right	Like right arrow

    💡 Why hjkl Instead of Arrow Keys?

    Vim's ancestor, vi, was created in 1976 on an ADM-3A terminal. That terminal's keyboard had arrow keys printed ON the h, j, k, and l keys — they literally WERE the arrow keys. The convention stuck because keeping hands on the home row (without reaching for arrow keys) is faster for touch typists.

    Arrow keys DO work in Vim. But learning hjkl pays dividends in speed.

Word Movement
Key	Action
w	Move forward one word
b	Move backward one word
e	Move to end of current word
W	Move forward one WORD (space-delimited, ignores punctuation)
B	Move backward one WORD
E	Move to end of current WORD

Difference between w and W:

# Cursor at start of this line, pressing w:
def hello_world():
#     ^    ^     ^
#     w    w     w   (stops at each punctuation mark)

# Cursor at start, pressing W:
def hello_world():
#         ^          ^
#         W          W   (jumps over punctuation to next space-delimited word)

Line Movement
Key	Action
0	Beginning of line (column 0)
^	First non-whitespace character on line
$	End of line
gg	First line of file
G	Last line of file
:{number}	Go to line number (e.g., :42 goes to line 42)
{number}G	Go to line number (e.g., 42G goes to line 42)
Ctrl+d	Half page down
Ctrl+u	Half page up
Ctrl+f	Full page down
Ctrl+b	Full page up
%	Jump to matching bracket/parenthesis
Screen Movement
Key	Action
H	Move cursor to top of screen (High)
M	Move cursor to middle of screen (Middle)
L	Move cursor to bottom of screen (Low)
zz	Center current line on screen
zt	Put current line at top of screen
zb	Put current line at bottom of screen
Insert Mode: Entering Text

From Normal mode, several keys enter Insert mode with different cursor positioning:
Key	Cursor Placement
i	Insert BEFORE cursor position
a	Insert AFTER cursor position (append)
I	Insert at beginning of line (first non-whitespace)
A	Insert at end of line
o	Open new line BELOW cursor, enter Insert mode
O	Open new line ABOVE cursor, enter Insert mode
s	Substitute: delete character at cursor, enter Insert mode
S	Substitute line: delete entire line, enter Insert mode
cw	Change word: delete from cursor to end of word, enter Insert mode
cc	Change line: delete entire line, enter Insert mode
C	Change to end of line: delete from cursor to end, enter Insert mode

Once in Insert mode, type normally. Press Esc to return to Normal mode.

    💡 The Esc Reflex

    Vim users develop a Pavlovian reflex: after finishing a thought, hit Esc. This returns to Normal mode where accidental keystrokes can't corrupt your text. If you're ever confused about which mode you're in, press Esc twice — you're guaranteed to be in Normal mode afterward.

Normal Mode: Editing Commands
Delete (Cut)
Key	Action
x	Delete character under cursor
X	Delete character before cursor
dd	Delete (cut) entire line
dw	Delete from cursor to end of word
de	Delete from cursor to end of word (includes trailing char)
d$	Delete from cursor to end of line
d0	Delete from beginning of line to cursor
dgg	Delete from current line to beginning of file
dG	Delete from current line to end of file
5dd	Delete 5 lines
Yank (Copy)
Key	Action
yy	Yank (copy) entire line
yw	Yank word
y$	Yank to end of line
yG	Yank from current line to end of file
5yy	Yank 5 lines
Paste (Put)
Key	Action
p	Paste below cursor (for lines) or after cursor (for characters)
P	Paste above cursor (for lines) or before cursor (for characters)
Undo and Redo
Key	Action
u	Undo last change
U	Undo all changes on current line
Ctrl+r	Redo (undo the undo)
The Power of Count Prefixes

Numbers multiply commands:

# Delete 3 words:
3dw

# Copy 5 lines:
5yy

# Move right 10 characters:
10l

# Delete 2 lines:
2dd

# Insert 20 dashes (type - then 19 more):
20i-<Esc>    # Produces: --------------------

The Dot Operator

The . (period) key repeats the last change:

# Delete a word:
dw
# Press . to delete the next word:
.
# Press . again for the third:
.

This is one of Vim's most powerful features — any edit can be repeated with a single keystroke.
Command-Line Mode

Enter by pressing : (colon) from Normal mode. The cursor drops to the bottom of the screen:
Saving and Quitting
Command	Action
:w	Write (save)
:w filename	Save as new filename
:q	Quit (fails if unsaved changes)
:q!	Quit WITHOUT saving (force)
:wq	Write and quit
:x	Write and quit (only if changes exist)
:wqa	Write all files and quit (when editing multiple)
:qa!	Quit all files without saving
Search
Command	Action
/pattern	Search forward for pattern
?pattern	Search backward for pattern
n	Next match (in search direction)
N	Previous match (opposite direction)
*	Search for word under cursor forward
#	Search for word under cursor backward
Search and Replace

# Replace first occurrence on current line:
:s/old/new/

# Replace ALL occurrences on current line:
:s/old/new/g

# Replace ALL occurrences in entire file:
:%s/old/new/g

# Replace with confirmation prompt for each:
:%s/old/new/gc

# Replace only in lines 10-20:
:10,20s/old/new/g

# Case-insensitive replacement:
:%s/old/new/gi

The search/replace syntax: [range]s/pattern/replacement/flags
Flag	Meaning
g	Global (all occurrences per line, not just first)
i	Case-insensitive
c	Confirm each replacement
e	Don't error if no matches found
Buffers, Tabs, and Splits

# Open another file in a new buffer:
:e /etc/hosts

# List open buffers:
:ls
# 1 #a "filename.txt"     line 42
# 2 %a "/etc/hosts"       line 1

# Switch to buffer 1:
:bp     # Previous buffer
:bn     # Next buffer
:b1     # Buffer 1 specifically

# Open file in new horizontal split:
:split /etc/hosts
# Or shorthand:
:sp /etc/hosts

# Open file in new vertical split:
:vsplit /etc/hosts
# Or shorthand:
:vsp /etc/hosts

# Navigate between splits:
Ctrl+w h    # Move to split on left
Ctrl+w j    # Move to split below
Ctrl+w k    # Move to split above
Ctrl+w l    # Move to split on right
Ctrl+w =    # Equalize split sizes
Ctrl+w _    # Maximize current split height
Ctrl+w |    # Maximize current split width

# Close a split:
:q          # Closes current split only
:only       # Close all splits except current

# Tab management:
:tabnew /etc/hosts    # Open file in new tab
:tabnext              # Next tab (or: gt)
:tabprev              # Previous tab (or: gT)
:tabclose             # Close current tab

Visual Mode: Selecting Text

Visual mode lets you select text ranges, then apply commands to the selection:
Key	Mode
v	Character-wise visual mode
V	Line-wise visual mode
Ctrl+v	Block-wise visual mode (column selection)

In Visual mode, move with normal movement keys (h, j, k, l, w, b, etc.) to extend the selection. Then:
Action	Key
Delete selection	d
Yank (copy) selection	y
Replace selection	c
Indent selection right	>
Indent selection left	<
Exit Visual mode	Esc

Block mode (Ctrl+v) is uniquely powerful — select a rectangular column of text:

# Given a file with aligned columns:
Name    Age    City
Alice   30     NYC
Bob     25     LA
Carol   35     SF

# Use Ctrl+v to select just the "Age" column,
# then press:
#   d   to delete the column
#   c   to change it (type new values)
#   I   to insert at the start of each line in the block
#   r   to replace every character in the block with one character

Vim Configuration: ~/.vimrc

Create your Vim configuration:

nano ~/.vimrc

Recommended beginner configuration:

" ─── General Settings ────────────────────────────────────
set nocompatible            " Use Vim settings, not Vi
syntax on                   " Enable syntax highlighting
filetype plugin indent on   " Enable filetype detection

" ─── Visual ──────────────────────────────────────────────
set number                  " Show line numbers
set relativenumber          " Show relative line numbers (great for jumps)
set cursorline              " Highlight current line
set showmatch               " Highlight matching brackets
set showcmd                 " Show command in bottom bar
set wildmenu                " Visual autocomplete for command menu
set laststatus=2            " Always show status line
set ruler                   " Show cursor position
set scrolloff=5             " Keep 5 lines visible above/below cursor
set wrap                    " Wrap long lines visually
set colorcolumn=80          " Highlight column 80 (PEP-8, etc.)

" ─── Search ──────────────────────────────────────────────
set incsearch               " Highlight matches while typing
set hlsearch                " Highlight all matches
set ignorecase              " Case-insensitive search
set smartcase               " ...unless you use capitals

" ─── Indentation ─────────────────────────────────────────
set autoindent              " Copy indentation from previous line
set smartindent             " Smart auto-indent for code
set expandtab               " Use spaces instead of tabs
set tabstop=4               " Tab key = 4 spaces
set shiftwidth=4            " Auto-indent = 4 spaces
set softtabstop=4           " Backspace deletes 4 spaces

" ─── Convenience ─────────────────────────────────────────
set backspace=indent,eol,start  " Backspace works normally
set mouse=a                     " Enable mouse support
set encoding=utf-8              " UTF-8 encoding
set autoread                    " Auto-reload files changed externally
set hidden                      " Allow switching buffers without saving

" ─── Key Mappings ────────────────────────────────────────
" Clear search highlighting with <leader>h:
let mapleader = " "
nnoremap <leader>h :nohlsearch<CR>

" Quick save:
nnoremap <leader>w :w<CR>

" Quick quit:
nnoremap <leader>q :q<CR>

" Split navigation shortcuts:
nnoremap <C-h> <C-w>h
nnoremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l

Save and reopen Vim — the configuration applies automatically.
Vim Tutor: The Built-In Tutorial

Vim includes an interactive tutorial. If you only do ONE thing in this section, do this:

vimtutor

This launches a 30-minute guided walkthrough covering all the essentials: movement, insertion, deletion, search, replace, and saving. It's the single best way to build initial Vim muscle memory.

    ⚠️ The Vim Learning Wall

    Everyone struggles with Vim initially. The first week feels slower than Nano because you're translating every action through mode-switching. Around week two, something clicks — you stop thinking about modes and start flying through text. By month two, you'll wonder how you ever edited without it.

    The key insight: Vim rewards INVESTMENT. It's not immediately intuitive, but its efficiency ceiling is dramatically higher than any modeless editor. Commit to two weeks of exclusive Vim use (for everything — even this manual's exercises) and you'll understand why it has devoted followers decades after its creation.

3. Emacs: The Extensible Universe
What Is Emacs?

GNU Emacs is not just a text editor — it's a programmable Lisp machine that happens to edit text. Created by Richard Stallman in 1976 (GNU version released 1985), Emacs is built on Emacs Lisp, a dialect of the Lisp programming language.

Emacs's philosophy: the editor should be infinitely customizable and capable of anything. Email, web browsing, Git management, task management, games, a psychotherapist — all run inside Emacs.

There's an old joke: "Emacs is a great operating system, lacking only a decent text editor." The truth behind it: Emacs IS a platform. Editing text is just one thing it does.
Installing Emacs

sudo dnf install emacs

Fedora 44 ships a recent version:

emacs --version
# GNU Emacs 29.x

The Emacs Philosophy: Keys as Chords

Unlike Vim's modal approach, Emacs uses chord-based key combinations. You hold modifier keys while pressing others:
Notation	Meaning
C-x	Ctrl+X
C-c	Ctrl+C
M-x	Meta+X (Alt+X on most keyboards; or Esc then X)
C-x C-s	Ctrl+X then Ctrl+S (sequential, not simultaneous)
C-x C-c	Ctrl+X then Ctrl+C (quit Emacs)

The M (Meta) key is Alt on modern keyboards. If your Alt key is awkward, press Esc, release, then press the next key.

    ⚠️ Emacs Pinky

    Emacs's heavy reliance on Ctrl and Alt keys has led to a notorious condition called "Emacs pinky" — repetitive strain injury in the left pinky from constant Ctrl reach. Solutions include remapping Caps Lock to Ctrl, using a foot pedal, or switching to Evil mode (Vim keybindings inside Emacs — the best of both worlds for some users).

Basic Workflow
Starting Emacs

# Open a file:
emacs myfile.txt

# Open without GUI (terminal mode):
emacs -nw myfile.txt

# Open multiple files:
emacs file1.py file2.py

# Open in daemon mode (faster subsequent launches):
emacs --daemon
emacsclient -t myfile.txt    # Connect to daemon, terminal mode
emacsclient -c myfile.txt    # Connect to daemon, GUI mode

Movement
Key	Action
C-f	Forward one character
C-b	Backward one character
C-n	Next line
C-p	Previous line
C-a	Beginning of line
C-e	End of line
M-f	Forward one word
M-b	Backward one word
M-v	Previous page (scroll up)
C-v	Next page (scroll down)
M-<	Beginning of buffer
M->	End of buffer
M-g M-g	Go to line number
Files and Buffers
Key Sequence	Action
C-x C-f	Find (open) file
C-x C-s	Save buffer
C-x C-w	Write (save as)
C-x b	Switch buffer
C-x C-b	List all buffers
C-x k	Kill (close) buffer
C-x C-c	Quit Emacs
Editing
Key Sequence	Action
C-d	Delete character after cursor
M-d	Delete word after cursor
C-k	Kill to end of line (cut)
C-y	Yank (paste)
M-y	Yank previous (cycle through kill ring)
C-w	Kill region (cut selection)
M-w	Copy region (copy selection)
C-space	Set mark (begin selection)
C-/	Undo
C-x u	Undo (alternative)
C-g	Cancel current command
Search and Replace
Key Sequence	Action
C-s	Search forward (incremental)
C-r	Search backward
M-%	Query replace (search and replace with prompts)
M-x replace-string	Replace all without prompting

Incremental search in Emacs is particularly elegant: as you type each character, the cursor jumps to the nearest match. No need to press Enter first.
Emacs Configuration: ~/.emacs.d/init.el

Emacs uses an initialization file written in Emacs Lisp:

mkdir -p ~/.emacs.d
emacs -nw ~/.emacs.d/init.el

Starter configuration:

;; ─── Package Management ──────────────────────────────────
(require 'package)
(setq package-archives
      '(("gnu" . "https://elpa.gnu.org/packages/")
        ("melpa" . "https://melpa.org/packages/")))
(package-initialize)

;; Install use-package if not present:
(unless (package-installed-p 'use-package)
  (package-refresh-contents)
  (package-install 'use-package))

;; ─── Basic Settings ──────────────────────────────────────
(setq inhibit-startup-screen t)          ; Skip welcome screen
(setq initial-scratch-message nil)       ; Empty scratch buffer
(tool-bar-mode -1)                       ; Disable toolbar
(menu-bar-mode -1)                       ; Disable menu bar
(scroll-bar-mode -1)                     ; Disable scroll bar
(global-display-line-numbers-mode 1)     ; Show line numbers
(setq-default indent-tabs-mode nil)     ; Use spaces, not tabs
(setq-default tab-width 4)              ; Tab = 4 spaces
(show-paren-mode 1)                     ; Highlight matching parens
(electric-pair-mode 1)                  ; Auto-close brackets
(global-hl-line-mode 1)                 ; Highlight current line
(delete-selection-mode 1)               ; Replace selection when typing
(setq make-backup-files nil)            ; No ~ backup files
(setq auto-save-default nil)            ; No #autosave# files

;; ─── Key Bindings ────────────────────────────────────────
(global-set-key (kbd "C-x C-b") 'ibuffer) ; Better buffer list
(global-set-key (kbd "M-o") 'other-window) ; Quick window switch

;; ─── Line Wrapping ──────────────────────────────────────
(global-visual-line-mode t)              ; Soft wrap long lines

;; ─── ido (Interactively DO things) ──────────────────────
(setq ido-enable-flex-matching t)
(setq ido-everywhere t)
(ido-mode 1)

Emacs Packages Worth Knowing

Emacs's package ecosystem (M-x package-list-packages) contains thousands of extensions. Notable ones:
Package	Purpose
Magit	Best Git interface for any editor
Org-mode	Note-taking, task management, literate programming
LSP-mode / eglot	Language Server Protocol support (autocompletion, go-to-definition)
Projectile	Project management (find files, grep within project)
Treemacs	File explorer sidebar
Company	Autocompletion framework
Flycheck	On-the-fly syntax checking
Which-key	Shows available keybindings as you type
Vertico + Consult	Modern completion framework
Evil	Vim keybindings inside Emacs
Org-Mode: The Killer App

Org-mode deserves special mention. It transforms Emacs into a structured document system:

* Project Ideas
** TODO Build a homelab
   - [X] Research hardware
   - [ ] Buy server rack
   - [ ] Install Proxmox
** DONE Write Fedora manual
   Closed: [2026-07-05 Fri]

Features include:

    Outlining with collapsible sections
    TODO keywords with priorities and deadlines
    Checkbox tracking within items
    Agenda views aggregating all TODOs
    Tables that compute formulas
    Code blocks with live execution (literate programming)
    Export to HTML, PDF, LaTeX, ODT

Many people learn Emacs specifically for Org-mode. It's that transformative.
4. Other Notable Terminal Editors
Micro

A modern, user-friendly terminal editor inspired by Nano but with more features:

sudo dnf install micro
micro myfile.txt

Micro adds split windows, tabs, mouse support, syntax highlighting, a plugin system, and — crucially — standard copy/paste shortcuts (Ctrl+C, Ctrl+V work as expected, not as editor commands). It's an excellent stepping stone between Nano and Vim.
MCEdit (Midnight Commander Editor)

Bundled with Midnight Commander file manager:

sudo dnf install mc
mcedit myfile.txt

Function-key driven editor popular in Eastern Europe and server administration circles. F2 saves, F10 quits. Simple but capable.
Ed

The ancient line editor — predecessor to vi:

ed myfile.txt

Ed operates one line at a time with no visual display. It's almost never used interactively today, but knowing it exists helps you understand Unix history and read old documentation. Ed is the ancestor that vi improved upon.

    💡 The Editor Holy War

    Vim vs. Emacs is one of computing's oldest flame wars, dating to the 1980s. In practice, both are excellent. The "winner" is whichever one you commit to learning deeply. Some people use both: Vim for quick edits and Emacs (with Evil mode) for complex workflows. Choose based on philosophy alignment:

        Prefer focused, minimal tools with steep efficiency gains? → Vim
        Prefer an extensible environment that can do everything? → Emacs
        Want neither learning curve right now? → Nano or Micro

5. Choosing Your Editor
Decision Guide

                    Need a terminal editor?
                            │
                    ┌───────┴───────┐
                    │               │
            Quick edits only     Daily driver
            (config files,       (coding, writing,
             cron jobs)           sysadmin work)
                    │               │
                    ▼               │
                 Nano               │
                              ┌─────┴─────┐
                              │           │
                        Want modal?   Want extensibility?
                        (modes,       (everything in
                         efficiency)   one environment)
                              │           │
                              ▼           ▼
                            Vim        Emacs

The Pragmatic Path

For most Fedora beginners, this progression works well:

    Start with Nano — edit config files comfortably, no learning curve
    Run vimtutor — spend 30 minutes learning Vim basics
    Try Vim for one week — force yourself to use it for everything
    If Vim clicks: invest in ~/.vimrc, learn Visual mode, splits, search/replace
    If Vim frustrates: stick with Nano, or try Micro for more features
    Curious about Emacs? Try it after you're comfortable in Vim or Nano — its learning curve is even steeper, but Org-mode and Magit might justify it

Setting Your Default Editor

Many Linux programs invoke an editor automatically (visudo, crontab -e, git commit). Which editor they use is determined by environment variables:

# Check your current default:
echo $EDITOR
echo $VISUAL

# Set Nano as default (add to ~/.bashrc):
export EDITOR=nano
export VISUAL=nano

# Set Vim as default:
export EDITOR=vim
export VISUAL=vim

# Set Emacs (terminal mode) as default:
export EDITOR="emacs -nw"
export VISUAL="emacs -nw"

Apply immediately:

source ~/.bashrc

    💡 EDITORvs.VISUAL

    Historically, $EDITOR was for line-based editors (like ed) and $VISUAL was for full-screen editors (like vi). On modern systems, most programs check $VISUAL first, then fall back to $EDITOR. Setting both to the same value eliminates ambiguity.

6. The $EDITOR Environment Variable

Programs that respect $EDITOR include:
Program	When It Opens Editor
visudo	Editing /etc/sudoers
crontab -e	Editing your cron schedule
git commit	Writing commit messages
systemctl edit	Editing systemd service overrides
mutt / neomutt	Composing email
less (press v)	Edit the file you're viewing
dnf changelog	Viewing changelogs in pager
man (press v in some viewers)	Editing man page source

If your editor opens unexpectedly and you're confused, check:

echo $EDITOR
# Whatever shows here is what system tools will launch

7. Configuration and Customization
Comparing Configuration Approaches
Editor	Config File	Language	Extensibility
Nano	~/.nanorc	INI-style directives	Limited (settings only)
Vim	~/.vimrc	Vimscript	Plugins via Vimscript/Lua
Emacs	~/.emacs.d/init.el	Emacs Lisp	Full programming language
Micro	~/.config/micro/settings.json	JSON	Limited (settings + plugins)
Vim Plugin Management

Modern Vim plugin management uses plugin managers. The simplest is vim-plug:

# Install vim-plug:
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

Add to ~/.vimrc:

call plug#begin('~/.vim/plugged')
" Syntax checking:
Plug 'dense-analysis/ale'
" File explorer:
Plug 'preservim/nerdtree'
" Fuzzy finder:
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
" Git integration:
Plug 'tpope/vim-fugitive'
" Surround management:
Plug 'tpope/vim-surround'
" Comment toggling:
Plug 'tpope/vim-commentary'
call plug#end()

Install plugins:

:source %
:PlugInstall

Emacs Package Management

As shown in the configuration section, Emacs uses its built-in package manager plus use-package for declarative configuration:

(use-package magit
  :ensure t
  :bind ("C-x g" . magit-status))

(use-package org
  :ensure t
  :bind ("C-c l" . org-store-link))

Install with M-x package-install-selected-packages.
Advanced Tips for All Editors
Comparing Files Side by Side

# Vim diff:
vimdiff file1.py file2.py

# Emacs diff:
emacs --eval '(ediff-files "file1.py" "file2.py")'

# Diffview in Vim:
vim -d file1.py file2.py    # Same as vimdiff

# Inside Vim:
:diffsplit file2.py         # Open diff in split

In Vimdiff, highlights show:

    Red background: lines only in one file
    Green background: lines added in one file
    Yellow background: changed characters within matching lines

Navigation in diff mode:
Key	Action
]c	Next difference
[c	Previous difference
dp	Diff put (push change to other window)
do	Diff obtain (pull change from other window)
:diffupdate	Refresh diff highlighting
Editing Remote Files

# Vim over SSH:
vim scp://user@server//etc/fstab

# Emacs over SSH (TRAMP):
emacs /ssh:user@server:/etc/fstab

This opens the remote file as if it were local. Saving writes back over SSH.
Multiple Cursors (Modern Editors)

Vim doesn't have true multiple cursors, but Visual Block mode achieves similar results:

# Insert text at the beginning of multiple selected lines:
# 1. Press Ctrl+v (block visual mode)
# 2. Select lines with j/k
# 3. Press I (capital i)
# 4. Type your text
# 5. Press Esc — text appears on ALL selected lines

Emacs has multiple-cursors package:

(use-package multiple-cursors
  :ensure t
  :bind (("C-S-c C-S-c" . mc/edit-lines)
         ("C->" . mc/mark-next-like-this)
         ("C-<" . mc/mark-previous-like-this)))

Try It Yourself
Exercise 1: Nano Basics

# Create a shopping list:
nano ~/shopping.txt

# Type several items, each on its own line.
# Save with Ctrl+O, Enter, then Ctrl+X to exit.

# Reopen and practice:
nano ~/shopping.txt
# - Search with Ctrl+W for an item
# - Cut a line with Ctrl+K, paste elsewhere with Ctrl+U
# - Go to end of file with Ctrl+V (PgDn), add items
# - Save and exit

# Clean up:
rm ~/shopping.txt

Exercise 2: Vimtutor (Essential!)

# This is the single most valuable exercise in this chapter.
# Spend 30 minutes completing it fully.
vimtutor

Exercise 3: Vim Editing Workflow

# Create a test file:
echo -e "apple\nbanana\ncherry\ndate\nelderberry" > ~/fruits.txt

# Open in Vim:
vim ~/fruits.txt

# Tasks to complete IN VIM:
# 1. Move to line 3 (3G or :3)
# 2. Delete the word "cherry" (dw)
# 3. Enter Insert mode (i), type "citrus"
# 4. Return to Normal mode (Esc)
# 5. Copy line 2 (yy), paste below (p)
# 6. Go to last line (G)
# 7. Add a new line below (o), type "fig"
# 8. Return to Normal mode (Esc)
# 9. Search for "an" (/an<Enter>)
# 10. Replace "banana" with "blueberry" (:s/banana/blueberry/)
# 11. Save and quit (:wq)

# Verify changes:
cat ~/fruits.txt

# Clean up:
rm ~/fruits.txt

Exercise 4: Vim Configuration

# Create your ~/.vimrc with the recommended settings from Section 2.
# Reopen a file and notice the improvements:
# - Line numbers (both absolute and relative)
# - Syntax highlighting
# - Search highlighting
# - Auto-indentation with spaces

# Test the leader key mappings:
# Press Space then h (clear search highlight)
# Press Space then w (quick save)

Exercise 5: Vim Splits and Buffers

# Open two files in splits:
vim -O /etc/hostname /etc/os-release

# Practice:
# - Ctrl+w l (move to right split)
# - Ctrl+w h (move to left split)
# - Ctrl+w = (equalize widths)
# - :q (close current split only)

# Open in tabs:
vim -p /etc/hostname /etc/os-release

# Practice:
# - gt (next tab)
# - gT (previous tab)
# - :tabclose (close current tab)

Exercise 6: Visual Mode Mastery

# Create a test file with columns:
cat > ~/data.txt << 'EOF'
Name    Score    Grade
Alice   92       A
Bob     78       B
Carol   85       B
Dave    96       A
EOF

# Open in Vim:
vim ~/data.txt

# Practice:
# 1. Press V (line visual), select all lines with G
# 2. Press > to indent the block right
# 3. Press u to undo
# 4. Press Ctrl+v (block visual), select just the Score column
# 5. Press r then X to replace all scores with X
# 6. Press u to undo
# 7. Save and quit: :wq

# Clean up:
rm ~/data.txt

Exercise 7: Emacs Exploration (Optional)

# Install Emacs:
sudo dnf install emacs

# Open the built-in tutorial:
emacs -nw
# Then press C-h t (Ctrl+H, then T)
# Complete the interactive tutorial

# Try Org-mode:
emacs -nw ~/notes.org
# Type: * My First Heading
# Type: ** TODO Buy groceries
# Type: *** TODO Milk
# Press Tab to collapse/expand sections
# Press C-c C-t to cycle TODO state
# Save: C-x C-s
# Quit: C-x C-c

Exercise 8: Editor Integration

# Set your preferred editor:
echo "export EDITOR=vim" >> ~/.bashrc
echo "export VISUAL=vim" >> ~/.bashrc
source ~/.bashrc

# Test with crontab:
crontab -e
# Your editor opens! Add a comment line starting with #
# Save and quit — the cron system accepts your edit.

# Test with visudo (if you have sudo access):
sudo visudo
# Notice it uses YOUR configured editor
# DON'T make changes — just observe, then quit without saving

# Test with git:
git config --global core.editor vim
# (Git commits will now open Vim)

Troubleshooting Common Issues
Problem	Editor	Cause	Solution
"Stuck" — can't quit	Vim	In wrong mode or forgot command	Press Esc, then :q! to force quit
Text looks garbled	Vim/Emacs	Terminal doesn't support colors	Add set term=builtin_ansi (Vim) or check $TERM env var
Arrow keys insert letters	Vim	In compatible mode or nocompatible not set	Add set nocompatible to ~/.vimrc
Backspace doesn't work	Vim	backspace option not configured	Add set backspace=indent,eol,start to ~/.vimrc
Paste messes up indentation	Vim	Autoindent fights pasted text	Use :set paste before pasting, :set nopaste after
Can't save — read only	Vim/Emacs	File permissions or SELinux	Use sudo vim or C-x C-q (Emacs toggle read-only)
Emacs won't start (GUI)	Emacs	Missing display or Wayland issue	Use emacs -nw for terminal mode
:wq says "readonly"	Vim	File owned by another user	:w !sudo tee % (saves with sudo trick)
Lost unsaved work	All editors	Crash or accidental quit	Vim: check ~/.filename.swp for recovery; Emacs: check #filename# auto-save
Plugin won't install	Vim	Missing vim-plug or wrong syntax	Run :PlugInstall after sourcing ~/.vimrc
Colors look wrong in SSH	All editors	Remote terminal lacks color support	Set export TERM=xterm-256color in ~/.bashrc
What's Next

You now have a complete terminal editing toolkit: Nano for quick edits, Vim for power editing with modal efficiency, and Emacs for those who want an extensible everything-environment. You understand each editor's philosophy, configuration, core commands, and how to set your system-wide default.

Chapter 20 turns to finding things — advanced file searching techniques beyond what Chapter 13 introduced. You'll master find expressions, locate databases, grep pattern matching with regular expressions, fd (a modern find replacement), ripgrep (a modern grep replacement), and strategies for locating anything on your system quickly.

Until then, practice your chosen editor relentlessly. Configuration files, shell scripts, notes, to-do lists — edit everything in the terminal. The fluency develops through repetition, not study. Pick an editor, commit to two weeks, and watch your efficiency transform.

    ✅ Chapter 19 Complete You understand why terminal text editors are essential (SSH sessions, system tools like visudo/crontab/git commit, GUI-less environments, rescue scenarios), Nano fundamentals (interface layout with ^ shortcuts, file opening/creating/saving/exiting, search and replace, cut/copy/paste with ^K/^U, movement commands, ~/.nanorc configuration with syntax highlighting and line numbers), Vim's modal editing paradigm (Normal/Insert/Command-line modes with transition keys, the Esc reflex, why hjkl replaces arrow keys), Vim movement (character/word/line/screen-level with gg/G/0/^//w/b/e/Ctrl+d/Ctrl+u//yy/yw/p/P/u/Ctrl+r with count prefixes and the dot operator), Vim search and replace (:s/old/new/g, :%s/old/new/gc with flags g/i/c/e, /pattern, n/N, *, #), Vim Visual mode (character v, line V, block Ctrl+v with d/y/c/>/</I/r operations), Vim splits/buffers/tabs (:e/:ls/:bp/:bn/:split/:vsplit/Ctrl+w navigation/:tabnew/gt/gT), Vim configuration via ~/.vimrc (settings for numbers, indentation, search, syntax, key mappings, leader key), vim-plug plugin management, Vimdiff for file comparison (]c/[c/dp/do), remote file editing via scp://, the vimtutor tutorial recommendation, Emacs overview (Lisp-based extensible platform, chord-based keys C-/M- notation, basic movement and editing commands, file/buffer management, search, ~/.emacs.d/init.el configuration with use-package, notable packages including Magit/Org-mode/LSP-mode/Evil), Org-mode as Emacs's killer feature (outlining, TODO management, agenda, code execution, export), Micro and MCEdit as alternative editors, editor selection decision guide, pragmatic learning progression (Nano → vimtutor → Vim week commitment → optional Emacs), EDITORandVISUAL environment variables (setting defaults, programs that respect them), configuration comparison across editors, and comprehensive troubleshooting reference for common editor problems.

