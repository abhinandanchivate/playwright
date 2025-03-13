### ✅ **Case Study: Automating Employee Data Processing using Python Basics and Collections**

---

## **📌 Scenario**  
You have been hired to automate the processing of employee data for a mid-sized company. The company stores employee information (name, age, department, and salary) in a text file.  

### **Objectives:**
1. Read employee data from a file.  
2. Store data using Python collections (like `list`, `dictionary`).  
3. Perform data validation and handling using control structures (`if-else`, `loops`).  
4. Implement functions to calculate the following:
   - Average salary of employees in each department.
   - Total salary for each department.
   - Highest and lowest salary in each department.  
5. Use exception handling to manage invalid data and file errors.  
6. Write the output back to a file.  

---

## **📁 Sample Employee Data File (`employees.txt`)**
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

## **🚀 Step 1: Read and Validate Data**
- Read the data from the file.
- Split and clean the data.
- Validate age and salary (should be positive).
- Store data in a dictionary.

---

### ✅ **Code:**
```python
def read_employee_data(file_name):
    employees = []
    try:
        with open(file_name, 'r') as file:
            for line in file:
                try:
                    name, age, department, salary = line.strip().split(", ")
                    age = int(age)
                    salary = int(salary)
                    
                    # Validation
                    if age < 0 or salary < 0:
                        raise ValueError(f"Invalid data for {name}: Age or Salary cannot be negative.")
                    
                    # Store in list of dictionaries
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

### ✅ **Explanation:**
✅ The data is read line-by-line.  
✅ `strip()` removes leading/trailing spaces.  
✅ `split()` separates data fields using commas.  
✅ `ValueError` handles invalid age/salary values.  
✅ Data is stored as a dictionary inside a list.  

---

## **🚀 Step 2: Process Data (Using Collections and Functions)**
- Create functions to compute:
   - Average salary per department.
   - Total salary per department.
   - Highest and lowest salary in each department.

---

### ✅ **Code:**
```python
from collections import defaultdict

def process_employee_data(employees):
    department_data = defaultdict(list)

    # Group employees by department
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

### ✅ **Explanation:**
✅ `defaultdict(list)` automatically creates a list for each department.  
✅ `append()` adds salaries to the list.  
✅ `sum()` calculates total and average salary.  
✅ `max()` and `min()` find highest and lowest salaries.  

---

## **🚀 Step 3: Display Results**
- Display output in a structured format.

---

### ✅ **Code:**
```python
def display_results(results):
    print("\nDepartment-wise Salary Data:")
    print("-" * 40)
    for department, data in results.items():
        print(f"Department: {department}")
        print(f" - Average Salary: ${data['average_salary']:.2f}")
        print(f" - Total Salary: ${data['total_salary']:.2f}")
        print(f" - Highest Salary: ${data['highest_salary']:.2f}")
        print(f" - Lowest Salary: ${data['lowest_salary']:.2f}")
        print("-" * 40)
```

---

## **🚀 Step 4: Write Output to a File**
- Write the structured output to a file.

---

### ✅ **Code:**
```python
def write_results_to_file(results, output_file):
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
    except Exception as e:
        print(f"Error writing to file: {e}")
```

---

## **🚀 Step 5: Main Function**
- Read -> Process -> Display -> Write

---

### ✅ **Code:**
```python
def main():
    input_file = 'employees.txt'
    output_file = 'salary_report.txt'

    # Read data
    employees = read_employee_data(input_file)

    if employees:
        # Process data
        results = process_employee_data(employees)

        # Display results
        display_results(results)

        # Write results to file
        write_results_to_file(results, output_file)

if __name__ == "__main__":
    main()
```

---

## **✅ Final Directory Structure**
```
📁 employee_salary_project
├── employees.txt
├── salary_report.txt
├── main.py
```

---

## **✅ Sample Output**
**Console Output:**
```
Department-wise Salary Data:
----------------------------------------
Department: IT
 - Average Salary: $52500.00
 - Total Salary: $105000.00
 - Highest Salary: $55000.00
 - Lowest Salary: $50000.00
----------------------------------------
Department: HR
 - Average Salary: $65000.00
 - Total Salary: $130000.00
 - Highest Salary: $70000.00
 - Lowest Salary: $60000.00
----------------------------------------
Department: Finance
 - Average Salary: $47500.00
 - Total Salary: $95000.00
 - Highest Salary: $50000.00
 - Lowest Salary: $45000.00
----------------------------------------
```

**File Output (`salary_report.txt`):**
```
Department-wise Salary Data:
----------------------------------------
Department: IT
 - Average Salary: $52500.00
 - Total Salary: $105000.00
 - Highest Salary: $55000.00
 - Lowest Salary: $50000.00
----------------------------------------
Department: HR
 - Average Salary: $65000.00
 - Total Salary: $130000.00
 - Highest Salary: $70000.00
 - Lowest Salary: $60000.00
----------------------------------------
Department: Finance
 - Average Salary: $47500.00
 - Total Salary: $95000.00
 - Highest Salary: $50000.00
 - Lowest Salary: $45000.00
----------------------------------------
```

---

## ✅ **Concepts Covered:**
✔️ Basic Python Syntax  
✔️ Data Types and Operators  
✔️ Control Structures (if-else, loops)  
✔️ Functions and Modules  
✔️ Exception Handling  
✔️ File Handling  
✔️ Collection Handling (`defaultdict`)  

---

