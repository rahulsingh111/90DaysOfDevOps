# Day 21 task

# Summary table

## ⚡ Quick Reference Table

| Topic | Key Syntax | Example |
|------|-------------|---------|
| Variable | `VAR="value"` | `NAME="DevOps"` |
| Argument | `$1`, `$2` | `./script.sh arg1` |
| If Statement | `if [ condition ]; then` | `if [ -f file ]; then echo "Exists"; fi` |
| For Loop | `for i in list; do ... done` | `for i in 1 2 3; do echo $i; done` |
| While Loop | `while [ condition ]; do ... done` | `while [ $i -lt 5 ]; do ((i++)); done` |
| Function | `name() { ... }` | `greet() { echo "Hi"; }` |
| Read Input | `read VAR` | `read -p "Enter name: " NAME` |
| Grep | `grep pattern file` | `grep -i "error" log.txt` |
| Awk | `awk '{print $1}' file` | `awk -F: '{print $1}' /etc/passwd` |
| Sed | `sed 's/old/new/g' file` | `sed -i 's/foo/bar/g' config.txt` |
| File Check | `[ -f file ]` | `if [ -f test.txt ]; then echo "Exists"; fi` |
| Exit Code | `$?` | `echo $?` |
| Debug Mode | `set -x` | `set -x script.sh` |


# Task 1
```bash
1. Shebang (#!/bin/bash)

The shebang tells the system which interpreter should run the script.

#!/bin/bash
echo "Hello DevOps"

Why it matters:

Ensures the script runs with Bash, not another shell.

Makes the script portable and predictable.

2. Running a Script

Make the script executable:

chmod +x script.sh

Run the script directly:

./script.sh

Run using Bash interpreter:

bash script.sh

Difference:

./script.sh requires execute permission.

bash script.sh does not require execute permission.

3. Comments

Comments are used to explain code. They are ignored during execution.

Single-line comment:

# This is a comment

Inline comment:

echo "Hello" # prints Hello
4. Variables

Variables store values.

Declare a variable:

NAME="DevOps"

Use the variable:

echo $NAME

Quoting variables:

echo "$NAME"   # expands variable
echo '$NAME'   # prints literal text

Example:

NAME="Rahul"
echo "Hello $NAME"
5. Reading User Input (read)

Used to take input from the user.

read NAME
echo "Hello $NAME"

Prompt example:

read -p "Enter your name: " NAME
echo "Welcome $NAME"
6. Command-Line Arguments

Arguments are values passed to the script when running it.

Example:

./script.sh DevOps Linux

Common argument variables:

Variable	Meaning
$0	Script name
$1	First argument
$2	Second argument
$#	Number of arguments
$@	All arguments
$?	Exit status of last command

Example script:

#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Total arguments: $#"
echo "All arguments: $@"

Run:

bash script.sh hello world

Output:

Script name: script.sh
First argument: hello
Second argument: world
Total arguments: 2
All arguments: hello world
```
# Task 2: Operators and Conditionals

```bash
1. String Comparisons
Used to compare text values. Always wrap variables in double quotes to prevent errors if the variable is empty.

=: Checks if two strings are equal (e.g., [ "$A" = "$B" ]).

!=: Checks if two strings are not equal (e.g., [ "$A" != "$B" ]).

-z: True if the string is empty/null (zero length).

-n: True if the string is not empty (non-zero length).

2. Integer Comparisons
Used for numerical values. Bash uses letter-based flags for these.

-eq: Equal to (e.g., [ $VAL -eq 10 ]).

-ne: Not equal to.

-lt: Less than.

-gt: Greater than.

-le: Less than or equal to.

-ge: Greater than or equal to.

3. File Test Operators
Essential for DevOps tasks like checking if a config file or directory exists.

-f: True if the path exists and is a regular file.

-d: True if the path exists and is a directory.

-e: True if the path simply exists, regardless of type.

-r / -w / -x: Checks if the file is readable, writable, or executable.

-s: True if the file exists and has a size greater than zero (not empty).

4. if, elif, else Syntax
The standard structure for decision-making in Bash.

Bash
if [ $AGE -ge 18 ]; then
    echo "Access granted"
elif [ $AGE -gt 13 ]; then
    echo "Limited access"
else
    echo "Access denied"
fi
5. Logical Operators
Used to combine multiple conditions.

&& (AND): True only if both conditions are true (e.g., [ $A -gt 1 ] && [ $A -lt 10 ]).

|| (OR): True if at least one condition is true.

! (NOT): Inverts the result of a condition (e.g., if [ ! -f "config.txt" ]).

6. Case Statements
A cleaner alternative to multiple if/elif blocks when checking a single variable against several patterns.

Bash
case $OS_TYPE in
    "Ubuntu" | "Debian")
        echo "Using apt manager" ;;
    "CentOS" | "RedHat")
        echo "Using yum manager" ;;
    *)
        echo "Unknown OS" ;;
esac
```
# Task 3: Loops

```bash
1. for Loop — List-based and C-style
The for loop is used to iterate over a predefined set of items.

List-based: Best for iterating over strings, numbers, or arrays.

Bash
for fruit in apple banana cherry; do
    echo "I like $fruit"
done
C-style: Best for precise numerical iterations.

Bash
for ((i=1; i<=5; i++)); do
    echo "Iteration number: $i"
done
2. while Loop
Runs a block of code as long as the specified condition remains true.

Bash
count=1
while [ $count -le 3 ]; do
    echo "Count is $count"
    ((count++))
done
3. until Loop
The opposite of while; it runs until the condition becomes true (it executes as long as the condition is false).

Bash
counter=1
until [ $counter -gt 3 ]; do
    echo "Counter: $counter"
    ((counter++))
done
4. Loop Control — break and continue
Used to change the flow of a loop based on internal logic.

break: Immediately exits the entire loop.

continue: Skips the rest of the current iteration and jumps to the next one.

Bash
for i in {1..5}; do
    if [ $i -eq 3 ]; then continue; fi # Skip 3
    if [ $i -eq 5 ]; then break; fi    # Stop at 5
    echo "Number: $i"
done
5. Looping Over Files
A powerful pattern for batch processing files in a directory.

Bash
# Example: Rename all .log files to .txt
for file in *.log; do
    mv "$file" "${file%.log}.txt"
done
6. Looping Over Command Output
Commonly used to process file content or command results line-by-line using read.

Bash
# Example: Read a file line by line
cat servers.txt | while read line; do
    echo "Pinging server: $line"
    ping -c 1 "$line"
done
```

# Task 4: Functions

```bash
1. Defining a Function
A function is a block of code that can be reused multiple times throughout a script.

Syntax: function_name() { ... }.

Example:

Bash
greet_user() {
    echo "Hello, Rahul!"
}
2. Calling a Function
To execute a function, simply type its name on a new line.

Note: The function must be defined before it is called in the script.

Example: greet_user.

3. Passing Arguments to Functions
Functions use positional parameters ($1, $2, etc.) just like scripts do, but these are local to the function itself.

Example:

Bash
welcome() {
    echo "Welcome to $1, $2!"
}
welcome "DevOps" "Rahul" # Passes "DevOps" as $1 and "Rahul" as $2
4. Return Values: return vs echo
There are two ways to get information back from a function:

return: Used to send an exit status (a number between 0-255). Access it using $?.

echo: Used to return actual data or strings. You can capture this using command substitution.

Bash
get_sum() {
    echo $(( $1 + $2 ))
}
RESULT=$(get_sum 10 5) # RESULT will be 15
5. Local Variables
By default, all variables in Bash are global. Use the local keyword inside a function to prevent it from overwriting variables in the rest of your script.

Example:

Bash
process_data() {
    local STATUS="Processing"
    echo "$STATUS..."
}
```

# Task 5: Text Processing Commands

```bash
1. grep (Global Regular Expression Print)
Used to search text for specific patterns.

-i: Ignore case (search for "Error" and "error").

-r: Recursive search through directories.

-c: Count the number of matching lines.

-n: Show the line number of matches.

-v: Invert match (show lines that do not match).

-E: Extended regex (allows use of |, +, ?).

2. awk
A powerful tool for processing data in columns/fields.

Print Columns: awk '{print $1, $3}' file (Prints 1st and 3rd column).

Field Separator: -F: (Uses : as a delimiter, like in /etc/passwd).

Patterns: awk '/Error/ {print $0}' (Only processes lines containing "Error").

BEGIN/END: Code to run before or after processing lines (e.g., printing headers/totals).

3. sed (Stream Editor)
Used for finding, replacing, and deleting text.

Substitution: sed 's/old/new/g' file (Replace all "old" with "new").

Delete Lines: sed '3d' file (Delete the 3rd line).

In-place Edit: -i (Saves changes directly to the file).

4. cut
Extracts specific sections from each line of a file.

Example: cut -d':' -f1 /etc/passwd (Extracts the first column using : as the delimiter).

5. sort
Arranges lines of text in a specific order.

-n: Sort numerically.

-r: Sort in reverse order.

-k: Sort based on a specific column (e.g., -k2).

6. uniq
Removes or identifies duplicate lines (requires the input to be sorted first).

-c: Count how many times each line occurs.

-u: Only show unique lines.

7. tr (Translate)
Deletes or replaces specific characters.

Example: echo "hello" | tr 'a-z' 'A-Z' (Converts to uppercase).

-d: Delete specific characters (e.g., tr -d '\r' to fix Windows line endings).

8. wc (Word Count)
-l: Count lines.

-w: Count words.

-c: Count characters/bytes.

9. head / tail
head -n 10: Show the first 10 lines.

tail -n 10: Show the last 10 lines.

tail -f: Follow mode (keeps the file open and prints new lines as they are added—great for logs!).
```

# Task 6: Useful Patterns and One-Liners

```bash
1. Find and Delete Old Files
Perfect for cleaning up old backups or temporary logs to keep your E: drive tidy.

Bash
find /home/rahul/scripts/archive -type f -mtime +30 -delete
Explanation: Finds files (-type f) modified more than 30 days ago (-mtime +30) and deletes them.

2. Search and Replace Across Multiple Files
Useful for updating configuration paths or variables across all your scripts at once.

Bash
grep -rl "old_path" . | xargs sed -i 's/old_path/new_path/g'
Explanation: grep finds filenames containing the string; xargs passes those names to sed for an in-place (-i) global replacement.

3. Real-Time Error Monitoring
The standard way to watch application logs as they happen.

Bash
tail -f /var/log/syslog | grep --line-buffered -i "error"
Explanation: tail -f follows the file live; grep filters for "error" (case-insensitive); --line-buffered ensures you see results immediately.

4. Disk Usage Alert
A quick check to see if your partitions are reaching a critical limit.

Bash
df -h | awk '$5+0 > 80 {print "Warning: " $1 " is at " $5}'
Explanation: df -h lists disk space; awk treats the 5th column (Percentage) as a number and prints a warning if it exceeds 80%.

5. Count Total Lines in All Log Files
Useful for getting a quick sense of how much data your system is generating.

Bash
find . -name "*.log" | xargs wc -l | grep "total"
Explanation: Finds all .log files and uses wc -l to count lines, then filters to show only the final sum.
```

# Task 7: Error Handling and Debugging

```bash
1. Exit Codes
Every command returns a status code (0–255) to the system after it finishes.

$?: A special variable that holds the exit status of the last command executed.

exit 0: Signals that the script finished successfully.

exit 1 (or any non-zero): Signals that an error occurred. You can use different numbers for different types of errors.

2. set -e (Exit on Error)
By default, Bash continues to the next line even if a command fails.

What it does: Tells the script to stop and exit immediately if any command returns a non-zero exit code.

Example: set -e placed at the top of your script prevents a "chain reaction" of failures.

3. set -u (No Unset Variables)
Prevents the script from using variables that haven't been defined.

What it does: Treats unset variables as an error and exits immediately.

Why it matters: Prevents disasters like rm -rf /$UNSET_VAR, which would accidentally run rm -rf /.

4. set -o pipefail
Standard set -e only looks at the exit code of the last command in a pipe.

What it does: Ensures that if any command in a pipe (e.g., step1 | step2 | step3) fails, the entire pipe returns a failure code.

5. set -x (Debug Mode)
The "X-ray" for your scripts.

What it does: Prints every command to the terminal before executing it, showing variable expansions.

Usage: You can use set -x to start debugging and set +x to turn it off for a specific block of code.

6. Trap (Cleanup on Exit)
Ensures your script cleans up after itself, regardless of how it finishes (success, error, or manual cancellation).

Syntax: trap 'command' SIGNAL.

Example:

Bash
trap 'rm -f /tmp/temp_file; echo "Cleanup complete"' EXIT
Explanation: This will automatically delete your temporary file whenever the script exits.
```

