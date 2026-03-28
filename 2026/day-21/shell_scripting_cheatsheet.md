### Task 1: Basics
Document the following with short descriptions and examples:
1. Shebang (`#!/bin/bash`) — what it does and why it matters
```
Tells the system which interpreter to use to run the script.

Ensures your script runs with the correct shell (bash instead of sh, zsh, etc.).

Example:

#!/bin/bash
echo "Hello, World!"
```
2. Running a script — `chmod +x`, `./script.sh`, `bash script.sh`
```
./script.sh → uses shebang
bash script.sh → explicitly uses bash
```
3. Comments — single line (`#`) and inline
```
# This is a comment
echo "Hello"

echo "Hello"  # This is inline comment prints Hello

# Multi line comment
<<'END'
This is the 1st line
This is the 2nd line
END
```

4. Variables — declaring, using, and quoting (`$VAR`, `"$VAR"`, `'$VAR'`)
```
Declaring a variable
name="umesh"

Using a variable
echo $name
```
5. Reading user input — `read`
```
echo "Take input from user"
read -p "Enter your name: " name
```

6. Command-line arguments — `$0`, `$1`, `$#`, `$@`, `$?`
```
echo $0   # Script name
echo $1   # First argument
echo $2   # Second argument
echo $#   # Number of arguments
echo $@   # All arguments
echo $?   # Exit status of last command

Example:

#!/bin/bash

echo "Script Name: $0"
echo "First Arg: $1"
echo "Total Args: $#"
```
---

### Task 2: Operators and Conditionals
Document with examples:
1. String comparisons — `=`, `!=`, `-z`, `-n`
```
= -> equals
!= -> not equal

name="umesh"

if [ "$name" = "umesh" ]; then
  echo "Name matches"
fi

-z -> string is empty
-n -> string is not empty

str=""

if [ -z "$str" ]; then
  echo "String is empty"
fi

if [ -n "$name" ]; then
  echo "String is not empty"
fi
```
2. Integer comparisons — `-eq`, `-ne`, `-lt`, `-gt`, `-le`, `-ge`
```
-eq(equal)

a=10
b=10

if [ $a -eq $b ]; then
  echo "Equal"
fi

-ne(not equal)

a=10
b=20

if [ $a -ne $b ]; then
  echo "Not equal"
fi

-lt(less than)

if [ 5 -lt 10 ]; then
  echo "5 is less than 10"
fi

-gt(greater than)

if [ 20 -gt 10 ]; then
  echo "20 is greater"
fi

-le(less than or equal)

if [ 10 -le 10 ]; then
  echo "Less than or equal"
fi

-ge(greater than or equal)

if [ 15 -ge 10 ]; then
  echo "Greater or equal"
fi

```
3. File test operators — `-f`, `-d`, `-e`, `-r`, `-w`, `-x`, `-s`
```
-f → file exists & is regular file

if [ -f "file.txt" ]; then
  echo "File exists"
fi

-d → directory exists

if [ -d "myfolder" ]; then
  echo "Directory exists"
fi

-e → file exists

if [ -e "file.txt" ]; then
  echo "Exists"
fi

-r → readable

if [ -r "file.txt" ]; then
  echo "Readable"
fi

-w → writable

if [ -w "file.txt" ]; then
  echo "Writable"
fi

-x → executable

if [ -x "script.sh" ]; then
  echo "Executable"
fi

-s → not empty

if [ -s "file.txt" ]; then
  echo "Not empty"
fi
```
4. `if`, `elif`, `else` syntax
```
num=25

if [ $num -lt 10 ]; then
  echo "Less than 10"
elif [ $num -lt 30 ]; then
  echo "Between 10 and 30"
else
  echo "Greater than 30"
fi
```
5. Logical operators — `&&`, `||`, `!`
```
# &&
age=25

if [ $age -gt 18 ] && [ $age -lt 30 ]; then
  echo "Young adult"
fi

# ||
num=5

if [ $num -lt 10 ] || [ $num -gt 20 ]; then
  echo "Condition matched"
fi

# !
if [ ! -f "test.txt" ]; then
  echo "File does not exist"
fi

```
6. Case statements — `case ... esac`
```
choice="start"

case $choice in
  start)
    echo "Starting service"
    ;;
  stop)
    echo "Stopping service"
    ;;
  restart)
    echo "Restarting service"
    ;;
  *)
    echo "Invalid option"
    ;;
esac
```
---

### Task 3: Loops
Document with examples:
1. `for` loop — list-based and C-style
2. `while` loop
3. `until` loop
4. Loop control — `break`, `continue`
5. Looping over files — `for file in *.log`
6. Looping over command output — `while read line`

---

### Task 4: Functions
Document with examples:
1. Defining a function — `function_name() { ... }`
2. Calling a function
3. Passing arguments to functions — `$1`, `$2` inside functions
4. Return values — `return` vs `echo`
5. Local variables — `local`

---

### Task 5: Text Processing Commands
Document the most useful flags/patterns for each:
1. `grep` — search patterns, `-i`, `-r`, `-c`, `-n`, `-v`, `-E`
2. `awk` — print columns, field separator, patterns, `BEGIN/END`
3. `sed` — substitution, delete lines, in-place edit
4. `cut` — extract columns by delimiter
5. `sort` — alphabetical, numerical, reverse, unique
6. `uniq` — deduplicate, count
7. `tr` — translate/delete characters
8. `wc` — line/word/char count
9. `head` / `tail` — first/last N lines, follow mode

---

### Task 6: Useful Patterns and One-Liners
Include at least 5 real-world one-liners you find useful. Examples:
- Find and delete files older than N days
- Count lines in all `.log` files
- Replace a string across multiple files
- Check if a service is running
- Monitor disk usage with alerts
- Parse CSV or JSON from command line
- Tail a log and filter for errors in real time

---

### Task 7: Error Handling and Debugging
Document with examples:
1. Exit codes — `$?`, `exit 0`, `exit 1`
2. `set -e` — exit on error
3. `set -u` — treat unset variables as error
4. `set -o pipefail` — catch errors in pipes
5. `set -x` — debug mode (trace execution)
6. Trap — `trap 'cleanup' EXIT`

---

### Task 8: Bonus — Quick Reference Table
Create a summary table like this at the top of your cheat sheet:

| Topic | Key Syntax | Example |
|-------|-----------|---------|
| Variable | `VAR="value"` | `NAME="DevOps"` |
| Argument | `$1`, `$2` | `./script.sh arg1` |
| If | `if [ condition ]; then` | `if [ -f file ]; then` |
| For loop | `for i in list; do` | `for i in 1 2 3; do` |
| Function | `name() { ... }` | `greet() { echo "Hi"; }` |
| Grep | `grep pattern file` | `grep -i "error" log.txt` |
| Awk | `awk '{print $1}' file` | `awk -F: '{print $1}' /etc/passwd` |
| Sed | `sed 's/old/new/g' file` | `sed -i 's/foo/bar/g' config.txt` |

---

## Format Guidelines

Your cheat sheet should be:
- Written in **Markdown** (`.md`)
- Organized with **clear headings** for each section
- Include **code blocks** with syntax highlighting (` ```bash `)
- Keep explanations **short** — 1-2 lines max per item
- Focus on **practical examples** over theory
- Something **you would actually refer back to** on the job

---

## Submission
1. Add your `shell_scripting_cheatsheet.md` to `2026/day-21/`
2. Commit and push to your fork

---

## Learn in Public

Share your cheat sheet on LinkedIn — help others revise too!

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
