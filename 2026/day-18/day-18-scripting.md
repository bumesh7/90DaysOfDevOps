ask 1: Basic Functions

    Create functions.sh with:
        A function greet that takes a name as argument and prints Hello, <name>!
        A function add that takes two numbers and prints their sum
        Call both functions from the script

```
$ vim functions.sh

#!/bin/bash
greet()
{
        echo "Hi $1"
}
greet $1

add()
{
        read -p "enter the a: " a
        read -p "enter the b: " b
        c=$((a+b))
        echo "$c"
}
add

$ chmod +x functions.sh
$ ./functions.sh umesh
```

Task 2: Functions with Return Values

    Create disk_check.sh with:
        A function check_disk that checks disk usage of / using df -h
        A function check_memory that checks free memory using free -h
        A main section that calls both and prints the results

```
$ vim disk_check.sh

#!/bin/bash
main()
{
        check_disk()
        {
                echo "Disk usage is..."
                df -h
        }
        check_disk
        memory_usage()
        {
                echo "Memory usage is.."
                free -h
        }
        memory_usage
}
main

$ chmod +x disk_check.sh
$ ./disk_check.sh
```

Task 3: Strict Mode — set -euo pipefail

    Create strict_demo.sh with set -euo pipefail at the top
    Try using an undefined variable — what happens with set -u?
    Try a command that fails — what happens with set -e?
    Try a piped command where one part fails — what happens with set -o pipefail?

Document: What does each flag do?

    set -e →
    set -u →
    set -o pipefail →

Task 4: Local Variables

    Create local_demo.sh with:
        A function that uses local keyword for variables
        Show that local variables don't leak outside the function
        Compare with a function that uses regular variables

Task 5: Build a Script — System Info Reporter

Create system_info.sh that uses functions for everything:

    A function to print hostname and OS info
    A function to print uptime
    A function to print disk usage (top 5 by size)
    A function to print memory usage
    A function to print top 5 CPU-consuming processes
    A main function that calls all of the above with section headers
    Use set -euo pipefail at the top

Output should look clean and readable.
Hints

    Function syntax: function_name() { ... }
    Local vars: local MY_VAR="value"
    Strict mode: set -euo pipefail as first line after shebang
    Pass args to functions: greet "Shubham" → access as $1 inside
    $? gives the exit code of last command
