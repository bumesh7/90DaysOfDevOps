Task 1: For Loop

    Create for_loop.sh that:
        Loops through a list of 5 fruits and prints each one
```
$ vim for_loop.sh

#!/bin/bash
for fruits in apple banana jackfruit grapes orange:
do
        echo "$fruits"
done

$ chmod 764 for_loop.sh
$ ./for_loop.sh
````
    Create count.sh that:
        Prints numbers 1 to 10 using a for loop

```
$ vim count.sh

#!/bin/bash
for ((i=1; i<=10; i++))
do
        echo "$i"
done

$ chmod +x count.sh
$ ./count.sh
```

Task 2: While Loop

    Create countdown.sh that:
        Takes a number from the user
        Counts down to 0 using a while loop
        Prints "Done!" at the end

```
$ vim countdown.sh

#!/bin/bash
i=10
while [ $i -ge 1 ]
do
    echo "$i"
    (( i-- ))
done

$ chmod +x countdown.sh
$ ./countdown.sh
```

Task 3: Command-Line Arguments

    Create greet.sh that:
        Accepts a name as $1
        Prints Hello, <name>!
        If no argument is passed, prints "Usage: ./greet.sh "

    Create args_demo.sh that:
        Prints total number of arguments ($#)
        Prints all arguments ($@)
        Prints the script name ($0)

```
$ vim greet.sh

!/bin/bash
#read -p "Enter the name of user: " $1
echo "Hi $1"
echo "The script name is: $0"
echo "The total count of arguemnts are: $#"
echo "The total entered arguments are: $@"

$ chmod +x greet.sh
$ ./greet.sh
```

Task 4: Install Packages via Script

    Create install_packages.sh that:
        Defines a list of packages: nginx, curl, wget
        Loops through the list
        Checks if each package is installed (use dpkg -s or rpm -q)
        Installs it if missing, skips if already present
        Prints status for each package

```
$ vim install_pacakes.sh

#!/bin/bash
if [ "$EUID" -ne 0 ]; then
        echo "Run as root"
        exit 1
fi

packages=( "nginx" "wget" "curl" )
for pkg in "${packages[@]}"
do
        if dpkg -s "$pkg" >/dev/null 2>&1; then
                echo "$pkg already installed"
        else
                echo "installing $pkg"
                apt-get install -y "$pkg"
        fi
done
echo "Done"

$ chmod +x insatll_pacakges.sh
$ sudo ./install_packages.sh
```


    Run as root: sudo -i or sudo su

Task 5: Error Handling

    Create safe_script.sh that:
        Uses set -e at the top (exit on error)
        Tries to create a directory /tmp/devops-test
        Tries to navigate into it
        Creates a file inside
        Uses || operator to print an error if any step fails

```
$ vim safe_script.sh

#!/bin/bash
set -e
mkdir /tmp/devops-test || echo "Directory already installed"

$ chmod +x safe_script.sh
$ ./safe_script.sh
```

Example:

mkdir /tmp/devops-test || echo "Directory already exists"

    Modify your install_packages.sh to check if the script is being run as root — exit with a message if not.

