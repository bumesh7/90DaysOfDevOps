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

```
$ vim strict_demo.sh

#!/bin/bash
set -euo pipefail

echo "undefined variable test set -u"
echo "$MY_VAR"

echo "Before failure test  set -e"
ls /umesh/abc/
echo "After failure"

echo "pipeline test set -o pipefail"
cat test.txt | grep "hi"

$ chmod +x strict_demo.sh
$ ./strict_demo.sh
```
Document: What does each flag do?

```
set -e →
Before failure test  set -e
ls: cannot access '/umesh/abc/': No such file or directory
 
set -u →
undefined variable test set -u
./strict_demo.sh: line 5: MY_VAR: unbound variable

set -o pipefail →
pipeline test set -o pipefail
cat: test.txt: No such file or directory
```

Task 4: Local Variables

    Create local_demo.sh with:
        A function that uses local keyword for variables
        Show that local variables don't leak outside the function
        Compare with a function that uses regular variables

```
$ vim local_demo.sh

#!/bin/bash
global_var="I am a global variable"

my_fun()
{
        local local_var="I am a local variable"
        echo "I am golbal variable inside function: $global_var"
        echo "I am local variavle inside funstion: $local_var"

}
my_fun

echo "I am local variable outside the funstion: $local_var"

$ chmod +x local_demo.sh
$./local_demo.sh

I am golbal variable inside function: I am a global variable
I am local variavle inside funstion: I am a local variable
I am local variable outside the funstion: 
```

Task 5: Build a Script — System Info Reporter

Create system_info.sh that uses functions for everything:

    A function to print hostname and OS info
    A function to print uptime
    A function to print disk usage (top 5 by size)
    A function to print memory usage
    A function to print top 5 CPU-consuming processes
    A main function that calls all of the above with section headers
    Use set -euo pipefail at the top

```
$ vim system_info.sh

!/bin/bash
set -euo pipefail
main()
{
        sys_info()
        {
                echo "System information"
                hostname
                uname -o
        }
        sys_info

        uptime_info()
        {
                echo "Uptime information"
                uptime -p
        }
        uptime_info

        disk_info()
        {
                echo "Disk information"
                df -h | sort -hr -k5 | head -5  # sort by column 5 in reverse order and give top 5.
        }
        disk_info

        memory_info()
        {
                echo "Memory information"
                free -h
        }
        memory_info

        cpu_info()
        {
                echo "cpu information"
                ps -eo pid,comm,%cpu --sort=-%cpu | head -6
        }
        cpu_info
}
main

$ chmod +x system_info.sh
$ ./system_info.sh
```
