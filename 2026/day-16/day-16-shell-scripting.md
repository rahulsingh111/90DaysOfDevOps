# Day 16

### Task 1: Your First Script
```bash
vi hello.sh
#/bin/bash
echo "Hello, DevOps!"
chmod +x hello.sh
./hello.sh
```
output:
Hello, DevOps!

It printed "Hello, DevOps!" to the terminal. If I remove the shebang line, the script is still running but it may not work as expected on all systems or with all shells. The shebang line tells the system which interpreter to use to execute the script, so without it, the script might be executed with a different shell that may not support certain features used in the script.

### Task 2: Variables
```bash
vi variables.sh
#!/bin/bash
NAME="Rahul"
ROLE="DevOps Engineer"
echo "Hello, I am $NAME and I am a $ROLE""
```
output:
Hello, I am Rahul and I am a DevOps Engineer

When using single quotes the value inside the quotes is treated as a literal string, so everything will be printed as is. When using double quotes, variables are expanded and the variables values will be printed instead of the variable names.

### Task 3: User Input with read
```bash
vi greet.sh
read -p "What is your name? " name
read -p "What is your favourite tool? " tool
echo "Hello $name, your favourite tool is $tool"
```
output:
What is your name? Rahul
What is your favourite tool? Docker
Hello Rahul, your favourite tool is Docker

### Task 4: If-Else Conditions
```bash
#!/bin/bash
read -p "enter a number: " num
if ((num > 0))
then
        echo 'Positive number'
elif ((num < 0))
then
        echo 'Negative number'
else
        echo 'Number is zero'
fi
```
output:
enter a number: 5
Positive number

---------To check the file existence---------
```bash
#!/bin/bash
read -p "enter a file name: " file
if [ -f "$file" ]

then
        echo "File exists"
else
        echo "File doesn't exists"
fi
```
output:
enter a file name: hello.sh
File exists

### Task 5: Combine It All
```bash
#!/bin/bash
read -p "enter the service name: " svc
echo "you entered $svc"

read -p "do you want to check the $svc service? :" ans
if
        [ $ans == "y" ]
then
        systemctl status $svc
elif
        [ $ans == "n" ]
then
        echo "Skipped"
fi
```
output:
enter the service name: nginx

The 3 things that I learned from this task are:
1. How to create and run a shell script using the terminal.
2. The importance of the shebang line in specifying the interpreter for the script.
3. How to use variables, read user input, and implement conditional statements in shell scripting. Additionally, I learned the big difference between single and double quotes when it comes to variable expansion in shell scripts.