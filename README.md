# Upgrade-Grading-System
It is a program of Upgrading Grading System
import csv
from prettytable import PrettyTable
import pandas as pd
import os

# Initialize global variables
students = []
subject_names = []


def get_subjects():
    """Prompt user for number of subjects and their names."""
    global subject_names
    while True:
        try:
            num_subjects = int(input("Enter the number of subjects: "))
            if num_subjects <= 0:
                print("Please enter a positive number.")
                continue
            break
        except ValueError:
            print("Invalid input. Please enter a valid number.")

    subject_names = [input(f"Enter the name of subject {i + 1}: ") for i in range(num_subjects)]


def add_student():
    """Add students and their grades."""
    while True:
        name = input("Enter the student's name (or type 'exit' to finish): ")
        if name.lower() == 'exit':
            break

        grades = {}
        for subject in subject_names:
            while True:
                try:
                    score = int(input(f"Enter score for {subject}: "))
                    if 0 <= score <= 100:
                        grades[subject] = score
                        break
                    else:
                        print("Score must be between 0 and 100.")
                except ValueError:
                    print("Invalid input. Please enter a valid score.")

        result = calculate_grade(grades)
        student = {
            "name": name,
            "grades": grades,
            "average_score": f"{result['average']:.2f}",
            "grade": result["grade"],
            "improvement": result["improvement"]
        }
        students.append(student)


def calculate_grade(grades):
    """Calculate the average and final grade based on scores."""
    scores = list(grades.values())
    average_score = sum(scores) / len(scores)

    if average_score >= 90:
        grade = 'A'
        improvement = "Excellent! Keep up the great work."
    elif 80 <= average_score < 90:
        grade = 'B'
        improvement = "Very good! Consider maintaining consistency."
    elif 70 <= average_score < 80:
        grade = 'C'
        improvement = "Good! Focus on improving weak areas."
    elif 60 <= average_score < 70:
        grade = 'D'
        improvement = "Fair! Dedicate more time to studies."
    else:
        grade = 'F'
        improvement = "Fail. Consider seeking help and studying harder."

    return {
        "average": average_score,
        "grade": grade,
        "improvement": improvement
    }


def display_students():
    """Display all student records in a table."""
    if not students:
        print("No student records available.\n")
        return

    table = PrettyTable()
    table.field_names = ["Name"] + subject_names + ["Average Score", "Grade", "Improvement Comment"]

    for student in students:
        row = [student["name"]]
        row.extend([student["grades"][subject] for subject in subject_names])
        row.extend([student["average_score"], student["grade"], student["improvement"]])
        table.add_row(row)

    print("\nStudent Grades and Feedback")
    print(table)


def export_records():
    """Export student records to CSV and Excel."""
    if not students:
        print("No student records available to export.\n")
        return

    data = []
    for student in students:
        row = {
            "Name": student["name"],
            "Average Score": student["average_score"],
            "Grade": student["grade"],
            "Improvement Comment": student["improvement"],
        }
        for subject in subject_names:
            row[subject] = student["grades"][subject]
        data.append(row)

    # Export to CSV
    csv_filename = "student_grades.csv"
    with open(csv_filename, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.DictWriter(file, fieldnames=data[0].keys())
        writer.writeheader()
        writer.writerows(data)
    print(f"Results successfully exported to {csv_filename}.")

    # Export to Excel
    excel_filename = "student_grades.xlsx"
    df = pd.DataFrame(data)
    df.to_excel(excel_filename, index=False)
    print(f"Results successfully exported to {excel_filename}.")

    return csv_filename, excel_filename


def student_records_system():
    """Student Records System main menu"""
    while True:
        print("\n--- Student Records System ---")
        print("1. Add New Student")
        print("2. Record Grades")
        print("3. Display All Student Records")
        print("4. Export Records")
        print("5. Exit")

        choice = input("Enter your choice: ")

        if choice == "1":
            get_subjects()
            add_student()

        elif choice == "2":
            if not subject_names:
                print("Please define subjects first by adding new students.\n")
                continue
            add_student()

        elif choice == "3":
            display_students()

        elif choice == "4":
            # Export options
            export_choice = input("Would you like to export the results to CSV or Excel? (csv/excel/both): ").lower()
            if export_choice in ['csv', 'excel', 'both']:
                csv_filename, excel_filename = None, None
                if export_choice == 'csv' or export_choice == 'both':
                    csv_filename = export_records()[0]
                if export_choice == 'excel' or export_choice == 'both':
                    excel_filename = export_records()[1]

                # Offer file download option
                download_choice = input("Would you like to download the file? (yes/no): ").lower()
                if download_choice == 'yes':
                    print("\nYou can now download the files from this directory:")
                    if csv_filename:
                        print(f"- {os.path.abspath(csv_filename)}")
                    if excel_filename:
                        print(f"- {os.path.abspath(excel_filename)}")
                else:
                    print("Export complete. Files are saved locally.")
            else:
                print("Invalid input. Please choose from 'csv', 'excel', or 'both'.")

        elif choice == "5":
            print("Exiting the system...")
            break

        else:
            print("Invalid choice. Please select from the options.")

if __name__ == "__main__":
    student_records_system()
