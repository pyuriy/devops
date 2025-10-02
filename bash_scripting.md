# Linux Bash Scripting Cheat Sheet

## 1. Script Structure
```bash
#!/bin/bash
# Comment: First line is shebang, specifies bash interpreter
# Save with .sh extension, make executable: chmod +x script.sh
```

## 2. Variables
```bash
# Declare and assign
name="John"
number=42

# Use variable
echo $name
echo ${name}  # Safer syntax, especially with strings

# Read-only variable
declare -r constant="fixed"

# Environment variable
export MY_VAR="value"
```

## 3. Basic Commands
```bash
# Print to terminal
echo "Hello, World!"

# Current directory
pwd

# List files
ls -l  # Detailed
ls -a  # Show hidden

# Change directory
cd /path/to/dir

# File operations
cp source dest      # Copy
mv source dest      # Move/rename
rm file             # Remove
mkdir directory     # Create directory
```

## 4. Input/Output
```bash
# Read user input
read -p "Enter name: " username

# Redirect output
command > file      # Overwrite
command >> file     # Append

# Pipe output to another command
ls | grep "txt"

# Read file content
cat file.txt
while read line; do
    echo $line
done < file.txt
```

## 5. Conditionals
```bash
# If statement
if [ condition ]; then
    # commands
elif [ condition ]; then
    # commands
else
    # commands
fi

# Test conditions
[ "$var" = "value" ]    # String equality
[ "$var" != "value" ]   # String inequality
[ $num -eq 10 ]        # Numeric equal
[ $num -ne 10 ]        # Numeric not equal
[ $num -gt 10 ]        # Greater than
[ $num -lt 10 ]        # Less than
[ -f file ]            # File exists
[ -d dir ]             # Directory exists
```

## 6. Loops
```bash
# For loop
for i in {1..5}; do
    echo $i
done

# While loop
while [ condition ]; do
    # commands
done

# Until loop
until [ condition ]; do
    # commands
done
```

## 7. Arrays
```bash
# Declare array
fruits=("apple" "banana" "orange")

# Access elements
echo ${fruits[0]}      # First element
echo ${fruits[@]}      # All elements
echo ${#fruits[@]}     # Array length

# Add element
fruits+=("grape")

# Loop through array
for fruit in "${fruits[@]}"; do
    echo $fruit
done
```

## 8. Functions
```bash
function my_function() {
    local var=$1    # Local variable, $1 is first argument
    echo "Argument: $var"
    return 0        # Return status
}

# Call function
my_function "test"
```

## 9. String Manipulation
```bash
str="Hello World"

# Length
echo ${#str}

# Substring
echo ${str:0:5}    # Get "Hello"

# Replace
echo ${str/World/Universe}

# Upper/Lower case
echo ${str^^}      # Uppercase
echo ${str,,}      # Lowercase
```

## 10. Arithmetic
```bash
# Basic operations
let "sum = 5 + 3"
((product = 5 * 3))

# Using expr
expr 5 + 3

# Double parentheses for complex math
((result = (5 + 3) * 2))
```

## 11. File Testing
```bash
[ -e file ]    # Exists
[ -f file ]    # Regular file
[ -d dir ]     # Directory
[ -r file ]    # Readable
[ -w file ]    # Writable
[ -x file ]    # Executable
```

## 12. Command Line Arguments
```bash
# Access arguments
echo $1         # First argument
echo $#         # Number of arguments
echo $@         # All arguments

# Shift arguments
shift           # Move arguments left
```

## 13. Exit Codes
```bash
# Exit with status
exit 0          # Success
exit 1          # Error

# Check last command status
echo $?
```

## 14. Common Utilities
```bash
# Find files
find /path -name "*.txt"

# Search in files
grep "pattern" file.txt
grep -r "pattern" /path  # Recursive

# Text processing
awk '{print $1}' file.txt
sed 's/old/new/g' file.txt

# Sort and unique
sort file.txt
uniq file.txt
```

## 15. Process Management
```bash
# Run in background
command &

# List processes
ps aux

# Kill process
kill PID
kill -9 PID    # Force kill

# Check background jobs
jobs
```

## 16. Error Handling
```bash
# Trap errors
trap 'echo "Error occurred"; exit 1' ERR

# Check if command succeeded
if ! command; then
    echo "Command failed"
fi
```

## 17. Here Document
```bash
cat << EOF
Multiple
lines
of text
EOF
```

## 18. Useful Tips
```bash
# Debug script
bash -x script.sh

# Check syntax
bash -n script.sh

# Set strict mode
set -euo pipefail

# Get script directory
SCRIPT_DIR=$(dirname "$0")

# Generate random number
echo $RANDOM
```
