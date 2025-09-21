# Linux Bash scripting lab

Below is a comprehensive Linux Bash scripting lab designed to guide users through fundamental and intermediate scripting concepts. The lab includes explanations, examples, and exercises to practice Bash scripting skills, covering variables, conditionals, loops, functions, and file manipulation.

```bash
#!/bin/bash

# Bash Scripting Lab
# This script serves as a comprehensive lab for learning Bash scripting.
# It includes explanations, examples, and exercises.

# ----------------------------------------------
# Section 1: Variables and User Input
# ----------------------------------------------
echo "=== Section 1: Variables and User Input ==="

# Example 1: Declaring and using variables
name="User"
echo "Hello, $name!"

# Example 2: Reading user input
echo "Please enter your name:"
read user_name
echo "Nice to meet you, $user_name!"

# Exercise 1: Create a variable for your favorite color and print it
# Uncomment and complete the line below
# favorite_color=""
# echo "My favorite color is $favorite_color"

# ----------------------------------------------
# Section 2: Conditionals
# ----------------------------------------------
echo -e "\n=== Section 2: Conditionals ==="

# Example 3: Simple if statement
age=20
if [ $age -ge 18 ]; then
    echo "You are an adult."
else
    echo "You are a minor."
fi

# Example 4: Using elif for multiple conditions
echo "Enter a number between 1 and 3:"
read number
if [ $number -eq 1 ]; then
    echo "You chose one."
elif [ $number -eq 2 ]; then
    echo "You chose two."
elif [ $number -eq 3 ]; then
    echo "You chose three."
else
    echo "Invalid choice."
fi

# Exercise 2: Write a conditional to check if a variable 'temperature' is above 30 (hot), between 20-30 (pleasant), or below 20 (cold)
# Uncomment and complete the lines below
# temperature=25
# if [ ... ]; then
#     echo "It's hot!"
# elif [ ... ]; then
#     echo "It's pleasant."
# else
#     echo "It's cold."
# fi

# ----------------------------------------------
# Section 3: Loops
# ----------------------------------------------
echo -e "\n=== Section 3: Loops ==="

# Example 5: For loop to iterate over a list
fruits=("apple" "banana" "orange")
for fruit in "${fruits[@]}"; do
    echo "Current fruit: $fruit"
done

# Example 6: While loop to count down
count=5
while [ $count -gt 0 ]; do
    echo "Countdown: $count"
    ((count--))
done

# Exercise 3: Write a for loop to print numbers 1 to 5
# Uncomment and complete the loop below
# for ...; do
#     echo "Number: ..."
# done

# ----------------------------------------------
# Section 4: Functions
# ----------------------------------------------
echo -e "\n=== Section 4: Functions ==="

# Example 7: Defining and calling a function
greet() {
    echo "Hello, $1! Welcome to the Bash lab."
}
greet "Alice"

# Example 8: Function with return value (using echo)
calculate_square() {
    local result=$(( $1 * $1 ))
    echo $result
}
square=$(calculate_square 4)
echo "Square of 4 is: $square"

# Exercise 4: Write a function 'cube' that calculates the cube of a number
# Uncomment and complete the function below
# cube() {
#     local result=...
#     echo $result
# }
# test_cube=$(cube 3)
# echo "Cube of 3 is: $test_cube"

# ----------------------------------------------
# Section 5: File Manipulation
# ----------------------------------------------
echo -e "\n=== Section 5: File Manipulation ==="

# Example 9: Creating and writing to a file
echo "This is a test file." > testfile.txt
echo "Created testfile.txt with some content."

# Example 10: Reading a file
if [ -f testfile.txt ]; then
    echo "Contents of testfile.txt:"
    cat testfile.txt
else
    echo "File does not exist."
fi

# Exercise 5: Write a script to append a line to testfile.txt
# Uncomment and complete the line below
# echo "New line added!" >> ...

# ----------------------------------------------
# Section 6: Error Handling
# ----------------------------------------------
echo -e "\n=== Section 6: Error Handling ==="

# Example 11: Checking for command success
mkdir test_dir
if [ $? -eq 0 ]; then
    echo "Directory created successfully."
else
    echo "Failed to create directory."
fi

# Example 12: Using trap for cleanup
cleanup() {
    echo "Cleaning up..."
    rm -f testfile.txt
    rmdir test_dir
}
trap cleanup EXIT

# Exercise 6: Write a check to see if a file 'data.txt' exists, and print appropriate message
# Uncomment and complete the lines below
# if [ ... ]; then
#     echo "data.txt exists."
# else
#     echo "data.txt does not exist."
# fi

echo -e "\n=== Lab Complete! ==="
echo "Try completing the exercises by uncommenting and filling in the code."
```

This lab is structured to be interactive and educational, with clear examples and exercises to reinforce learning. To use it, save the script, make it executable (`chmod +x bash_scripting_lab.sh`), and run it (`./bash_scripting_lab.sh`). The exercises are commented out for users to complete, encouraging hands-on practice.
