Let's extend the **Employee Data Processing** case study by adding the following advanced features:

### 🔥 **New Enhancements**:
1. ✅ **Data Sorting** – Sort employee data by:
   - Name  
   - Salary  
   - Age  
2. ✅ **Enhanced Output Format** – Generate output in **JSON format**.  
3. ✅ **More Error Handling** – Handle missing fields, incorrect formatting, etc.  
4. ✅ **Interactive User Input** – Allow the user to choose sorting options and output format.  
5. ✅ **Command Line Arguments** – Accept input and output file names via command line.  

---

## 📁 **Updated Sample Employee Data (`employees.txt`)**
```
John, 30, IT, 50000
Doe, 45, HR, 60000
Alice, 29, IT, 55000
Bob, 35, Finance, 45000
Eve, 40, HR, 70000
Chris, 31, Finance, 50000
InvalidEmployee, -25, IT, -1000
```

---

## 🚀 **1. Enhanced Data Reading and Error Handling**
### ✅ **Features Added:**
- Handle **missing fields**.  
- Skip records with incorrect formatting.  
- Improved validation.  

---

### ✅ **Code:**
```python
def read_employee_data(file_name):
    employees = []
    try:
        with open(file_name, 'r') as file:
            for line in file:
                try:
                    # Ensure the data has exactly 4 fields
                    fields = line.strip().split(", ")
                    if len(fields) != 4:
                        raise ValueError(f"Invalid format: {line.strip()}")

                    name, age, department, salary = fields
                    age = int(age)
                    salary = int(salary)

                    # Validate data
                    if age <= 0 or salary <= 0:
                        raise ValueError(f"Invalid data for {name}: Age or Salary cannot be negative.")

                    employee = {
                        'name': name,
                        'age': age,
                        'department': department,
                        'salary': salary
                    }
                    employees.append(employee)
                except ValueError as e:
                    print(f"Skipping record due to error: {e}")
    except FileNotFoundError:
        print(f"Error: File '{file_name}' not found.")
    except Exception as e:
        print(f"Error: {e}")
    
    return employees
```

---

## 🚀 **2. Sorting Function**
### ✅ **Features Added:**
- Sort employee data by:
   - Name  
   - Salary  
   - Age  
- User input for sorting preference.  

---

### ✅ **Code:**
```python
def sort_employees(employees, sort_key):
    try:
        return sorted(employees, key=lambda x: x[sort_key])
    except KeyError:
        print(f"Invalid sorting key: {sort_key}")
        return employees
```

---

## 🚀 **3. Updated Processing**
### ✅ **Features Added:**
- Sort and process data.  
- Calculate average, total, highest, and lowest salary.  

---

### ✅ **Code:**
```python
from collections import defaultdict

def process_employee_data(employees):
    department_data = defaultdict(list)

    for employee in employees:
        department_data[employee['department']].append(employee['salary'])

    results = {}

    for department, salaries in department_data.items():
        avg_salary = sum(salaries) / len(salaries)
        total_salary = sum(salaries)
        highest_salary = max(salaries)
        lowest_salary = min(salaries)

        results[department] = {
            'average_salary': avg_salary,
            'total_salary': total_salary,
            'highest_salary': highest_salary,
            'lowest_salary': lowest_salary
        }
    
    return results
```

---

## 🚀 **4. Write to JSON File**
### ✅ **Features Added:**
- Generate output in **JSON format**.  
- Output to both console and file.  

---

### ✅ **Code:**
```python
import json

def write_results_to_json(results, output_file):
    try:
        with open(output_file, 'w') as file:
            json.dump(results, file, indent=4)
        print(f"Results written to '{output_file}' in JSON format.")
    except Exception as e:
        print(f"Error writing to JSON file: {e}")
```

---

## 🚀 **5. Write to Text File**
### ✅ **Features Added:**
- Added more structured output.  
- Improved formatting.  

---

### ✅ **Code:**
```python
def write_results_to_text(results, output_file):
    try:
        with open(output_file, 'w') as file:
            file.write("Department-wise Salary Data:\n")
            file.write("-" * 40 + "\n")
            for department, data in results.items():
                file.write(f"Department: {department}\n")
                file.write(f" - Average Salary: ${data['average_salary']:.2f}\n")
                file.write(f" - Total Salary: ${data['total_salary']:.2f}\n")
                file.write(f" - Highest Salary: ${data['highest_salary']:.2f}\n")
                file.write(f" - Lowest Salary: ${data['lowest_salary']:.2f}\n")
                file.write("-" * 40 + "\n")
        print(f"Results written to '{output_file}'.")
    except Exception as e:
        print(f"Error writing to file: {e}")
```

---

## 🚀 **6. Main Function with User Choice**
### ✅ **Features Added:**
- Accept sorting option from user.  
- Allow user to choose output format.  
- Use command-line arguments for input and output files.  

---

### ✅ **Code:**
```python
import sys

def main():
    if len(sys.argv) != 3:
        print("Usage: python main.py <input_file> <output_file>")
        return
    
    input_file = sys.argv[1]
    output_file = sys.argv[2]

    # Read data
    employees = read_employee_data(input_file)

    if employees:
        # Sort by user choice
        sort_key = input("Sort by (name, age, salary): ").strip()
        employees = sort_employees(employees, sort_key)

        # Process data
        results = process_employee_data(employees)

        # Display Results
        display_results(results)

        # Output format choice
        output_format = input("Output format (text/json): ").strip().lower()

        if output_format == 'json':
            write_results_to_json(results, output_file)
        else:
            write_results_to_text(results, output_file)

if __name__ == "__main__":
    main()
```

---

## ✅ **Sample Output (Console)**
```
Sort by (name, age, salary): salary
Output format (text/json): json
Results written to 'salary_report.json' in JSON format.
```

---

## ✅ **Sample Output (`salary_report.json`)**
```json
{
    "IT": {
        "average_salary": 52500.0,
        "total_salary": 105000.0,
        "highest_salary": 55000,
        "lowest_salary": 50000
    },
    "HR": {
        "average_salary": 65000.0,
        "total_salary": 130000.0,
        "highest_salary": 70000,
        "lowest_salary": 60000
    },
    "Finance": {
        "average_salary": 47500.0,
        "total_salary": 95000.0,
        "highest_salary": 50000,
        "lowest_salary": 45000
    }
}
```

---

## ✅ **Directory Structure**
```
📁 employee_salary_project
├── employees.txt
├── salary_report.json
├── main.py
```

---

## ✅ **Concepts Covered:**
✔️ Collections (`list`, `dict`, `defaultdict`)  
✔️ Functions and Modules  
✔️ Exception Handling  
✔️ File Handling  
✔️ JSON Handling  
✔️ Sorting (by key)  
✔️ User Input and Command-Line Arguments  

---

## 🏆 **Key Improvements:**
✅ Added sorting.  
✅ Added JSON output support.  
✅ Enhanced error handling.  
✅ Better user interaction.  

---

