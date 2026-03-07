### Task 1: For Loop

```bash
#!/bin/bash
for fruits in apple mango banana kiwi grapes
do
        echo $fruits
done
```

```bash
#!/bin/bash
for num in {1..10}
do
        echo $num
done
```

### Task 2: While Loop
```bash
#!/bin/bash
read -p "enter a number: " num
while [ $num -ge 0 ]
do
        echo $num
        ((num--))
done
```

### Task 3: Command-Line Arguments
```bash
#!/bin/bash
echo "Hello $1"
if [ -z "$1" ]; then
  echo "Error: No argument provided."
  exit 1
fi
```

```bash
#!/bin/bash

# Print script name
echo "Script name: $0"

# Print total number of arguments
echo "Total number of arguments: $#"

# Print all arguments
echo "All arguments: $@"
```

### Task 4: Install Packages via Script
```bash
#!/bin/bash

read -p "Enter packages: " -a packages
for pkg in "${packages[@]}"
do
        echo "checking $pkg"

        if dpkg -s "$pkg" &> /dev/null
        then
                echo "$pkg is present"
        else
                echo "$pkg is not present. installing"
                sudo apt-get install -y "$pkg"
                systemctl status "$pkg"
        fi
done
```

### Task 5: Error Handling
```bash
#!/bin/bash

set -e

mkdir /tmp/devops-test || echo "Error: Failed to create directory"
cd /tmp/devops-test || echo "Error: Failed to change directory"
touch devops-test.txt || echo "Error: Failed to create file"

echo "Script completed successfully"
```
# Run as root
```bash
#!/bin/bash
if [ "$EUID" -ne 0 ]; then
        echo "please run as root"
        exit 1
fi
read -p "Enter packages: " -a packages
for pkg in "${packages[@]}"
do
        echo "checking $pkg"

        if dpkg -s "$pkg" &> /dev/null
        then
                echo "$pkg is present"
        else
                echo "$pkg is not present. installing"
                sudo apt-get install -y "$pkg"
                systemctl status "$pkg"
        fi
done
```

Today I learned:
1. How to use for loops and while loops in bash scripting.
2. How to handle command-line arguments in bash scripts.
3. How to check for package installation and handle errors in bash scripts.