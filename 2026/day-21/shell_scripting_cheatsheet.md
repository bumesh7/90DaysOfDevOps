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
```
List based

for i in 1 2 3 4 5
do
  echo "Number: $i"
done

C-style

for ((i=1; i<=5; i++))
do
  echo "Count: $i"
done
```
2. `while` loop
```
count=1

while [ $count -le 5 ]
do
  echo "Count: $count"
  ((count++))
done
```
3. `until` loop
```
count=1

until [ $count -gt 5 ]
do
  echo "Count: $count"
  ((count++))
done
```
4. Loop control — `break`, `continue`
```
#stop at 3

for i in 1 2 3 4 5
do
  if [ $i -eq 3 ]; then
    break
  fi
  echo $i
done

#skip 3
for i in 1 2 3 4 5
do
  if [ $i -eq 3 ]; then
    continue
  fi
  echo $i
done
```
5. Looping over files — `for file in *.log`
```
for file in *.log
do
  echo "Processing $file"
done
```
6. Looping over command output — `while read line`
```
cat file.txt | while read line
do
  echo "Line: $line"
done
```
---

### Task 4: Functions
Document with examples:
1. Defining a function — `function_name() { ... }`
2. Calling a function
```
greet() {
    echo "Hello, Umesh!"
}

greet   # Function call
```
3. Passing arguments to functions — `$1`, `$2` inside functions
```
add_numbers() {
    sum=$(($1 + $2))
    echo "Sum is: $sum"
}

add_numbers 5 10
```
4. Return values — `return` vs `echo`
```
# using return
check_even() {
    if (( $1 % 2 == 0 ))
    then
        return 0   # success
    else
        return 1   # failure
    fi
}

check_even 4
echo $?   # prints return value

# using echo

multiply() {
    result=$(($1 * $2))
    echo $result
}

output=$(multiply 3 4)
echo "Result: $output"
```
5. Local variables — `local`
```
my_function() {
    local name="Umesh"
    echo "Inside function: $name"
}

my_function
echo "Outside function: $name"

output:
Inside function: Umesh
Outside function: 
```
---

### Task 5: Text Processing Commands
Document the most useful flags/patterns for each:
1. `grep` — search patterns, `-i`, `-r`, `-c`, `-n`, `-v`, `-E`
```
-i → ignore case
-r → recursive search
-c → count matches
-n → show line numbers
-v → invert match
-E → extended regex

grep -in "error" logfile.log
grep -r "nginx" /etc/
grep -c "failed" app.log
grep -v "DEBUG" app.log
grep -E "error|fail" app.log
```
2. `awk` — print columns, field separator, patterns, `BEGIN/END`
```
Default field separator → space
-F → custom delimiter
$1, $2 → columns
BEGIN → before processing
END → after processing

awk '{print $1}' file.txt
awk -F ":" '{print $1}' /etc/passwd
awk '/error/ {print $0}' app.log
awk 'BEGIN {print "Start"} {print $1} END {print "End"}' file.txt
```
3. `sed` — substitution, delete lines, in-place edit
```
s/old/new/ → substitute
d → delete line
-i → edit file in-place

sed 's/error/warning/g' file.txt
sed '2d' file.txt
sed -i 's/8080/9090/g' config.txt
```
4. `cut` — extract columns by delimiter
```
-d → delimiter
-f → field number

cut -d ":" -f1 /etc/passwd
cut -d "," -f2 data.csv
```
5. `sort` — alphabetical, numerical, reverse, unique
```
-n → numeric
-r → reverse
-u → unique

sort file.txt
sort -n numbers.txt
sort -r file.txt
sort -u file.txt
```
6. `uniq` — deduplicate, count
```
-c → count occurrences
-d → show duplicates only

uniq file.txt
uniq -c file.txt
uniq -d file.txt
```
7. `tr` — translate/delete characters
```
replace characters
delete characters

echo "hello" | tr 'a-z' 'A-Z'
echo "hello123" | tr -d '0-9'
```
8. `wc` — line/word/char count
```
-l → lines
-w → words
-c → characters

wc -l file.txt
wc -w file.txt
wc -c file.txt
```
9. `head` / `tail` — first/last N lines, follow mode
```
-n → number of lines
tail -f → follow logs

head -n 5 file.txt
tail -n 10 file.txt
tail -f app.log
```
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
```
# 1. Find & delete files older than 7 days
find /path/to/dir -type f -mtime +7 -delete

# 2. Count lines in all .log files
wc -l *.log

# 3. Replace a string across multiple files
sed -i 's/old_text/new_text/g' *.txt

# 4. Check if a service is running
systemctl is-active nginx

# 5. Alternative service check
ps aux | grep nginx | grep -v grep

# 6. Monitor disk usage (>80% alert)
df -h | awk '$5+0 > 80 {print "Warning: " $0}'

# 7. Get top 5 largest files
du -ah /path | sort -rh | head -5

# 8. Check open ports/services
ss -tulpn

# 9. Tail logs & filter errors in real-time
tail -f app.log | grep -i "error"

# 10. Count unique IPs from logs
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head

# 11. Extract column from CSV
cut -d ',' -f2 data.csv

# 12. Check HTTP status code
curl -o /dev/null -s -w "%{http_code}\n" https://example.com

# 13. Check memory usage
free -h

# 14. Find failed SSH login attempts
grep "Failed password" /var/log/auth.log

# 15. Remove empty files
find . -type f -empty -delete

# 16. Loop over files
for file in *.log; do echo "Processing $file"; done

# 17. Count live errors from logs
tail -f app.log | grep -i "error" | wc -l
```

```
# CSV → extract 2nd column
cut -d ',' -f2 data.csv

# CSV → skip header & get first column
tail -n +2 data.csv | cut -d ',' -f1

# CSV → filter rows (column 3 > 100)
awk -F ',' '$3 > 100 {print $0}' data.csv

# CSV → get unique values from column 1
awk -F ',' '{print $1}' data.csv | sort | uniq

# JSON → pretty print
jq '.' data.json

# JSON → extract field
jq '.name' data.json

# JSON → extract nested field
jq '.user.name' data.json

# JSON → loop array & get values
jq '.users[] | .name' data.json

# JSON → filter condition
jq '.users[] | select(.age > 25)' data.json

# JSON → extract multiple fields
jq '.users[] | {name, age}' data.json

# JSON → convert to CSV
jq -r '.users[] | [.name, .age] | @csv' data.json
```
---

### Task 7: Error Handling and Debugging
Document with examples:
1. Exit codes — `$?`, `exit 0`, `exit 1`
2. `set -e` — exit on error
3. `set -u` — treat unset variables as error
4. `set -o pipefail` — catch errors in pipes
5. `set -x` — debug mode (trace execution)
6. Trap — `trap 'cleanup' EXIT`
```
set -e → stops script on error
set -u → fails on undefined variables
set -o pipefail → catches pipeline errors
set -x → shows command execution
$? → checks exit code
trap EXIT → runs cleanup always
trap ERR → handles errors gracefully
```
```
#!/bin/bash

# Enable strict mode
set -euo pipefail

# Enable debugging (trace execution)
set -x

# Cleanup function
cleanup() {
    echo "Cleaning up..."
    rm -f temp.txt
}

# Trap to run cleanup on exit
trap cleanup EXIT

# Trap to catch errors
trap 'echo "Error occurred at line $LINENO. Exiting..."; exit 1' ERR

echo "Script started..."

# Create temp file
touch temp.txt

# Example 1: Exit code check
ls temp.txt
echo "Exit code of last command: $?"

# Example 2: Using unset variable (will fail due to set -u)
# Uncomment below line to test
# echo $UNDEFINED_VAR

# Example 3: Pipeline failure (caught due to pipefail)
cat temp.txt | grep "data" | sort

# Example 4: Force an error
cp non_existent_file.txt /tmp/

# This line will not run due to set -e
echo "Script completed successfully"
```
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

```
Topic        | Key Syntax                     | Example
-------------|--------------------------------|--------------------------------------
Variable     | VAR="value"                   | NAME="DevOps"
Argument     | $1, $2                        | ./script.sh arg1
If           | if [ condition ]; then        | if [ -f file ]; then
For Loop     | for i in list; do             | for i in 1 2 3; do
While Loop   | while [ condition ]; do       | while [ $i -lt 5 ]; do
Function     | name() { ... }                | greet() { echo "Hi"; }
Exit Code    | $?                            | echo $?
Grep         | grep pattern file             | grep -i "error" log.txt
Awk          | awk '{print $1}' file         | awk -F: '{print $1}' /etc/passwd
Sed          | sed 's/old/new/g' file        | sed -i 's/foo/bar/g' config.txt
Cut          | cut -d delim -fN              | cut -d ',' -f1 data.csv
Sort         | sort [options]                | sort -nr file.txt
Uniq         | uniq [options]                | uniq -c file.txt
Tr           | tr 'a-z' 'A-Z'                | echo hi | tr 'a-z' 'A-Z'
Wc           | wc -l/-w/-c file              | wc -l file.txt
Head         | head -n N file                | head -n 5 file.txt
Tail         | tail -n N file                | tail -f app.log
Find         | find path [options]           | find . -name "*.log"
Permissions  | chmod 755 file                | chmod +x script.sh
Process      | ps aux | grep name            | ps aux | grep nginx
Network      | ss -tulpn                     | ss -tulpn | grep 80
```
---
