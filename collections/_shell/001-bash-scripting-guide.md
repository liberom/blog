---
layout: post
title: "Bash Scripting Guide: Automation and System Administration"
author: "Liberom"
date: 2025-01-07
category: "Shell"
toc: true
excerpt: "Master Bash scripting: variables, functions, control flow, and automation techniques for DevOps."
---

# Bash Scripting Guide: Automation and System Administration

Bash scripting automates repetitive tasks, manages systems, and orchestrates complex workflows. Strong Bash skills are essential for DevOps and system administration.

## Bash Basics

### Script Structure

```bash
#!/bin/bash
# Shebang line (tells system to use bash interpreter)

# Comments start with #

# Set strict mode (fail on error, undefined variables, pipe failures)
set -euo pipefail

# Your script code here
echo "Hello, World!"
```

### Executing Scripts

```bash
# Make script executable
chmod +x script.sh

# Run script
./script.sh

# Run with bash explicitly
bash script.sh

# Run with bash from anywhere
bash /path/to/script.sh
```

## Variables and Parameters

### Variable Assignment

```bash
#!/bin/bash

# Create variable (no spaces around =)
name="John"
age=30
is_active=true

# Use variable with $
echo "Hello, $name!"
echo "Name: ${name}"  # Safer with braces

# Read from input
read -p "Enter your name: " user_name

# Command substitution
current_date=$(date +%Y-%m-%d)
file_count=$(ls -1 | wc -l)

# Default values
${variable:-default}       # Use default if unset
${variable:=default}       # Use and set default if unset
${variable:?Error message} # Exit with error if unset
```

### Special Variables

```bash
#!/bin/bash

# Script parameters
$0     # Script name
$1, $2, $3  # Arguments
$*     # All arguments as one string
$@     # All arguments as separate strings
$#     # Number of arguments
$?     # Exit code of last command
$$     # Process ID of script
$!     # Process ID of last background job

# Example: Function with parameters
function greet() {
    echo "Hello, $1!"  # First parameter
    echo "Total args: $#"
}

greet "Alice"  # Output: Hello, Alice! / Total args: 1
```

## String Manipulation

### String Operations

```bash
#!/bin/bash

text="Hello, World!"

# Length
echo ${#text}           # 13

# Substring
echo ${text:0:5}        # "Hello"
echo ${text:7}          # "World!"

# Replace
echo ${text/World/Bash} # "Hello, Bash!"
echo ${text//o/0}       # "Hell0, W0rld!" (all)

# Remove pattern
echo ${text%,*}         # "Hello"
echo ${text#*,}         # " World!"

# Case conversion (Bash 4+)
text_lower=${text,,}    # lowercase
text_upper=${text^^}    # UPPERCASE
```

### String Testing

```bash
#!/bin/bash

text="hello"

# Check if empty
if [[ -z "$text" ]]; then
    echo "String is empty"
fi

# Check if not empty
if [[ -n "$text" ]]; then
    echo "String is not empty"
fi

# Check if equals
if [[ "$text" == "hello" ]]; then
    echo "Match"
fi

# Pattern matching
if [[ "$text" =~ ^h.*o$ ]]; then
    echo "Matches pattern"
fi
```

## Control Flow

### if/else Statements

```bash
#!/bin/bash

age=25

# Basic if
if [[ $age -ge 18 ]]; then
    echo "Adult"
fi

# if/else
if [[ $age -lt 13 ]]; then
    echo "Child"
elif [[ $age -lt 18 ]]; then
    echo "Teenager"
else
    echo "Adult"
fi

# Comparison operators
[[ $a -eq $b ]]   # Equal (numbers)
[[ $a -ne $b ]]   # Not equal
[[ $a -lt $b ]]   # Less than
[[ $a -le $b ]]   # Less than or equal
[[ $a -gt $b ]]   # Greater than
[[ $a -ge $b ]]   # Greater than or equal

# String comparison
[[ "$str1" == "$str2" ]]   # Equal
[[ "$str1" != "$str2" ]]   # Not equal
[[ "$str1" < "$str2" ]]    # Less than

# File testing
[[ -f "$file" ]]    # File exists
[[ -d "$dir" ]]     # Directory exists
[[ -e "$path" ]]    # Path exists
[[ -r "$file" ]]    # Readable
[[ -w "$file" ]]    # Writable
[[ -x "$file" ]]    # Executable
```

### Loops

```bash
#!/bin/bash

# For loop - C style
for ((i = 0; i < 5; i++)); do
    echo $i
done

# For loop - iteration
for item in apple banana cherry; do
    echo $item
done

# For loop - arrays
files=(file1.txt file2.txt file3.txt)
for file in "${files[@]}"; do
    echo "Processing: $file"
done

# For loop - globbing
for file in *.txt; do
    echo "Found: $file"
done

# While loop
counter=0
while [[ $counter -lt 5 ]]; do
    echo $counter
    ((counter++))
done

# Until loop (opposite of while)
counter=0
until [[ $counter -eq 5 ]]; do
    echo $counter
    ((counter++))
done

# Break and continue
for i in {1..10}; do
    if [[ $i -eq 3 ]]; then
        continue  # Skip this iteration
    fi
    if [[ $i -eq 8 ]]; then
        break     # Exit loop
    fi
    echo $i
done
```

## Functions

### Function Definition

```bash
#!/bin/bash

# Function definition
function greet() {
    local name=$1              # Local variable
    local greeting="Hello, $name!"
    echo "$greeting"
}

# Function with return value
function add() {
    local result=$(($1 + $2))
    return $result
}

# Call function
greet "Alice"
greet "Bob"

# Capture function output
result=$(greet "Charlie")
echo "Result: $result"

# Check return code
if greet "Invalid"; then
    echo "Success"
else
    echo "Failed"
fi
```

### Function Best Practices

```bash
#!/bin/bash

# GOOD - Clear function with local variables
function process_file() {
    local filename=$1
    local output_dir=${2:-.}  # Default to current dir

    if [[ ! -f "$filename" ]]; then
        echo "Error: File not found" >&2
        return 1
    fi

    # Process file
    grep "pattern" "$filename" > "$output_dir/output.txt"
    return 0
}

# GOOD - Error handling
function safe_copy() {
    local src=$1
    local dest=$2

    if [[ ! -f "$src" ]]; then
        echo "Error: Source file not found: $src" >&2
        return 1
    fi

    if ! cp "$src" "$dest"; then
        echo "Error: Failed to copy $src to $dest" >&2
        return 1
    fi
}

# Call with error checking
if ! process_file "data.txt"; then
    echo "Processing failed"
    exit 1
fi
```

## Arrays and Loops

```bash
#!/bin/bash

# Array declaration
fruits=(apple banana cherry date)

# Access elements
echo ${fruits[0]}    # apple
echo ${fruits[1]}    # banana
echo ${#fruits[@]}   # 4 (length)

# Add element
fruits+=(elderberry)

# Iterate array
for fruit in "${fruits[@]}"; do
    echo $fruit
done

# Iterate with index
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done

# Associative array (map/dict)
declare -A person
person[name]="John"
person[age]=30
person[city]="NYC"

# Access associative array
echo ${person[name]}

# Iterate associative array
for key in "${!person[@]}"; do
    echo "$key: ${person[$key]}"
done
```

## File Operations

```bash
#!/bin/bash

# Check file type
if [[ -f "$file" ]]; then
    echo "Regular file"
elif [[ -d "$file" ]]; then
    echo "Directory"
elif [[ -L "$file" ]]; then
    echo "Symbolic link"
fi

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < "file.txt"

# Read file into array
mapfile -t lines < "file.txt"
for line in "${lines[@]}"; do
    echo "$line"
done

# Write to file
echo "Content" > "output.txt"          # Overwrite
echo "More content" >> "output.txt"    # Append

# Create directory
mkdir -p "dir1/dir2/dir3"  # -p creates parent dirs

# Remove files/directories
rm "file.txt"              # Remove file
rm -r "directory"          # Remove directory recursively
rm -f "file.txt"           # Force remove
```

## Error Handling

### Exit Codes and Error Checking

```bash
#!/bin/bash

# Strict mode
set -euo pipefail
IFS=$'\n\t'

# Check exit code
if command; then
    echo "Success"
else
    echo "Failed with code: $?"
fi

# Explicit error handling
function safe_command() {
    if ! some_command; then
        echo "Error: Command failed" >&2
        return 1
    fi
    return 0
}

# Error trap
trap 'echo "Error on line $LINENO"' ERR

# Cleanup trap
cleanup() {
    rm -f /tmp/tempfile
}
trap cleanup EXIT
```

## Common Patterns

### Script Template

```bash
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

# Global variables
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
LOG_FILE="/var/log/myscript.log"

# Logging function
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Error handler
error_exit() {
    echo "Error: $1" >&2
    exit 1
}

# Main function
main() {
    log "Script started"

    # Your code here

    log "Script completed"
}

# Cleanup
trap 'echo "Script interrupted"' INT TERM
trap 'cleanup' EXIT

cleanup() {
    log "Cleaning up..."
}

# Run main
main "$@"
```

### Backup Script Example

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backups"
SOURCE_DIR="/var/www"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${TIMESTAMP}.tar.gz"

# Create backup
tar -czf "$BACKUP_DIR/$BACKUP_FILE" -C "$(dirname "$SOURCE_DIR")" \
    "$(basename "$SOURCE_DIR")"

# Keep only last 7 backups
cd "$BACKUP_DIR"
ls -t backup_*.tar.gz | tail -n +8 | xargs rm -f

echo "Backup completed: $BACKUP_FILE"
```

## Conclusion

Bash scripting essentials:
- Use strict mode (`set -euo pipefail`)
- Quote all variables (`"$var"`)
- Use `local` variables in functions
- Check exit codes explicitly
- Use `[[ ]]` instead of `[ ]`
- Implement proper error handling
- Write reusable functions
- Add logging to production scripts

