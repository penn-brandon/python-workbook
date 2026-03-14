Thought about this
Python Cheatsheet for Beginners
# Python Cheatsheet for Beginners

## 🔹 Basic Syntax

```python
# Comments start with hash
print("Hello, World!")  # Output text

# Variables (no declaration needed)
name = "Alice"          # String
age = 25                # Integer
height = 5.7            # Float
is_student = True       # Boolean
🔹 Data Types
Type	Example	Description
str	"hello"	Text
int	42	Whole numbers
float	3.14	Decimal numbers
bool	True/False	True or False
list	[1, 2, 3]	Ordered, mutable collection
tuple	(1, 2, 3)	Ordered, immutable collection
dict	{"key": "value"}	Key-value pairs
set	{1, 2, 3}	Unique unordered collection
🔹 Strings
text = "Python"

# Common operations
len(text)           # Length: 6
text.upper()        # "PYTHON"
text.lower()        # "python"
text[0]             # First char: "P"
text[0:3]           # Slice: "Pyt"
text + "!"          # Concatenation: "Python!"
f"Hi {name}"        # f-string formatting

# Methods
text.strip()        # Remove whitespace
text.replace("Py", "Ja")  # Replace
text.split(",")     # Split into list
🔹 Lists
fruits = ["apple", "banana", "cherry"]

# Access
fruits[0]           # "apple"
fruits[-1]          # "cherry" (last item)

# Modify
fruits.append("orange")     # Add to end
fruits.insert(1, "grape")   # Insert at index
fruits.remove("banana")     # Remove by value
fruits.pop()                # Remove last item

# Operations
len(fruits)         # Count items
"apple" in fruits   # Check membership: True
sorted(fruits)      # Return sorted copy
🔹 Dictionaries
person = {"name": "Alice", "age": 25, "city": "NYC"}

# Access
person["name"]              # "Alice"
person.get("email", "")     # Safe access with default

# Modify
person["age"] = 26          # Update value
person["job"] = "Engineer"  # Add new key
del person["city"]          # Delete key

# Methods
person.keys()               # Get all keys
person.values()             # Get all values
person.items()              # Get key-value pairs
🔹 Conditionals
age = 18

if age < 13:
    print("Child")
elif age < 20:
    print("Teenager")
else:
    print("Adult")

# Comparison operators
==  # Equal
!=  # Not equal
>   # Greater than
<   # Less than
>=  # Greater than or equal
<=  # Less than or equal

# Logical operators
and, or, not
🔹 Loops
# For loop
for i in range(5):        # 0, 1, 2, 3, 4
    print(i)

for fruit in fruits:      # Iterate list
    print(fruit)

for key, value in person.items():  # Dict
    print(f"{key}: {value}")

# While loop
count = 0
while count < 5:
    print(count)
    count += 1

# Loop control
break       # Exit loop
continue    # Skip to next iteration
🔹 Functions
# Define function
def greet(name):
    return f"Hello, {name}!"

# Call function
message = greet("Alice")

# Default parameters
def greet(name="Guest"):
    return f"Hello, {name}!"

# Multiple return values
def get_stats(numbers):
    return min(numbers), max(numbers), sum(numbers)/len(numbers)

min_val, max_val, avg = get_stats([1, 2, 3, 4, 5])
🔹 Error Handling
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Can't divide by zero!")
except Exception as e:
    print(f"Error: {e}")
finally:
    print("This always runs")

# Raise custom error
if age < 0:
    raise ValueError("Age can't be negative")
🔹 File Operations
# Read file
with open("file.txt", "r") as f:
    content = f.read()
    lines = f.readlines()

# Write file
with open("output.txt", "w") as f:
    f.write("Hello, file!")

# Append file
with open("log.txt", "a") as f:
    f.write("New entry\n")
🔹 Common Modules
import math
math.sqrt(16)           # 4.0
math.pi                 # 3.14159...

import random
random.randint(1, 10)   # Random number
random.choice(items)    # Random item from list

import datetime
now = datetime.datetime.now()

import json
json.loads('{"key": "value"}')  # Parse JSON
json.dumps(data)                # Convert to JSON
🔹 List Comprehensions
# Create list in one line
squares = [x**2 for x in range(10)]
even_squares = [x**2 for x in range(10) if x % 2 == 0]

# Dictionary comprehension
squares_dict = {x: x**2 for x in range(5)}
🔹 Useful Built-in Functions
Function	Purpose
len()	Get length
range()	Generate number sequence
type()	Check data type
int(), str(), float()	Type conversion
input()	Get user input
sum(), min(), max()	Math operations
enumerate()	Get index + value in loop
zip()	Combine two lists
🔹 Quick Tips
# Swap variables
a, b = b, a

# Check if empty
if not my_list:
    print("List is empty")

# Unpack list
first, second, *rest = [1, 2, 3, 4, 5]

# Ternary operator
status = "adult" if age >= 18 else "minor"
Save this as python_cheatsheet.md and keep it handy for quick reference!


---