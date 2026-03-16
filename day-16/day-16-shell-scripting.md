Task 1: Your First Script

    Create a file hello.sh
    Add the shebang line #!/bin/bash at the top
    Print Hello, DevOps! using echo
    Make it executable and run it

chmod +x hello.sh
./hello.sh

```
$ vim hello.sh

#!/bin/bash
echo "Hello, Devops!"

$ chmod +x hello.sh
$ ./hello.sh

Document: What happens if you remove the shebang line?

-> Shebang tells the system to use the bash shell. When we remove shebang it will use default shell 
```

Task 2: Variables

    Create variables.sh with:
        A variable for your NAME
        A variable for your ROLE (e.g., "DevOps Engineer")
        Print: Hello, I am <NAME> and I am a <ROLE>
    Try using single quotes vs double quotes — what's the difference?

```
$ vim variable.sh

#!/bin/bash
read -p "Enter your name: " NAME
read -p "Enter your job role: " ROLE
echo "Hello, I am $NAME and I am a $ROLE"

$ chmod +x variable.sh
$ ./variable.sh

Try using single quotes vs double quotes — what's the difference?

-> with double quotes
I am umesh and I am a Devops Eng

-> with single quotes
I am $NAME and I am a $ROLE
```

Task 3: User Input with read

    Create greet.sh that:
        Asks the user for their name using read
        Asks for their favourite tool
        Prints: Hello <name>, your favourite tool is <tool>

```
$ vim greet.sh

#!/bin/bash
read -p "Enter your name: " NAME
read -p "Enter your favourite tool: " TOOL
echo "Hello $NAME, your favourite tool is $TOOL"

$ chmod +x greet.sh

$ ./greet.ch

o/p => Hello Umesh, your favourite tool is docker
```

Task 4: If-Else Conditions

    Create check_number.sh that:
        Takes a number using read
        Prints whether it is positive, negative, or zero

```
$ vim check_number.sh

#!/bin/bash
read -p "Enter the number: " NUM
if [ $NUM -lt 0 ]; then
        echo "The number is Negative"
elif [ $NUM -gt 0 ]; then
        echo "The number is Positive"
else
        echo "The number is Zero"
fi

$ chmod +x check_number.sh
$ ./check_number.sh
```

    Create file_check.sh that:
        Asks for a filename
        Checks if the file exists using -f
        Prints appropriate message

```
$ vim file_check.sh

#!/bin/bash
read -p "Enter the file name: " FILE
if [ -f "$FILE" ]; then
        echo "The file name  $FILE exists"
else
        echo "The file name $FILE does not exists"
fi

$ chmod +x file_check.sh
$ ./file_check.sh
```

Task 5: Combine It All

Create server_check.sh that:

    Stores a service name in a variable (e.g., nginx, sshd)
    Asks the user: "Do you want to check the status? (y/n)"
    If y — runs systemctl status <service> and prints whether it's active or not
    If n — prints "Skipped."

```
$ vim server_check.sh

#!/bin/bash
read -p "Enter the service name: " SERVICE
echo "The service name is $SERVICE"
read -p "Do you want to check the status of the $SERVICE: " STATUS
if [ $STATUS == "y" ]; then
        sudo systemctl status $SERVICE
elif [ $STATUS == "n" ]; then
        echo "skipped"
else
        echo "Please enter y or n"
fi

$ chmod +x server_check.sh

$ ./server_check.sh
```



