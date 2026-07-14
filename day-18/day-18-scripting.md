# Day 18 – Shell Scripting: Functions & intermediate Concepts

### Task 1: Basic Functions
```bash
#!/bin/bash
greet(){
        echo "Hello $1"
}

add(){
        sum=$(( $1 + $2 ))
        echo "Sum = $sum"
}

read -p "enter a name: " name
read -p "enter first number: " num1
read -p "enter second number: " num2

greet "$name"
add "$num1" "$num2"
```

### Task 2: Functions with Return Values
```bash
#!/bin/bash
check_disk(){
        df -h /
}
check_memory(){
        free -h
}

check_disk
check_memory
```

### Task 3: Strict Mode
```bash
#!/bin/bash
set -euo pipefail
echo "Strict mode enabled"

echo "Using undefined variable:"
echo "$undefined_var"


echo "Running a failing command:"
ls /nonexistent_directory

echo "Testing pipe failure:"
cat missingfile.txt | grep "hello"

echo "Script finished"
```

- `set -e` → exits immediately if any command fails (returns a non-zero status).
- `set -u` → treats undefined variables as an error and exits immediately.
- `set -o pipefail` → if any command in a pipeline fails, the entire pipeline fails with that error code instead of just the last command.


### Task 4: Local Variables
```bash
#!/bin/bash
# Function using local variable
local_function() {
    local var="This is local"
    echo "Inside local_function: $var"
}
# Function using global variable
global_function() {
    var="This is global"
    echo "Inside global_function: $var"
}

local_function
echo "Outside after local_function: $var"

echo "-----------------------------"

global_function

echo "Outside after global_function: $var"
```

### Task 5: Build a Script — System Info Reporter
```bash
#!/bin/bash
set -euo pipefail

os(){
    echo "===== OS INFO ====="
    hostname
    cat /etc/os-release
}

uptime_info(){
    echo
    echo "===== UPTIME ====="
    uptime
}

disk(){
    echo
    echo "===== DISK USAGE ====="
    du -sh * | sort -hr | head -5
}

memory(){
    echo
    echo "===== MEMORY ====="
    free -h
}

cpu(){
    echo
    echo "===== TOP CPU PROCESSES ====="
    :ps -eo pid,comm,%cpu --sort=-%cpu | head -6
}

main(){
    os
    uptime_info
    disk
    memory
    cpu
}
main
```

What I learned:
1. Functions in bash can take arguments and return values, making scripts more modular and reusable.
2. Using `set -euo pipefail` helps catch errors early and makes scripts more robust by exiting on failures, treating undefined variables as errors, and ensuring that pipeline failures are properly handled.
3. Local variables in functions prevent variable name conflicts and ensure that variables do not leak outside their intended scope, while global variables can lead to unintended side effects if not managed carefully.