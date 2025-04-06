[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/_-JrKZbN)

Question 1 ( 1. Build a program to perform CRUD operations (Create, Read, Update, Delete) on a database table Product with columns:
ProductID, ProductName, Price, and Quantity.
The program should include:
Menu-driven options for each operation.
Transaction handling to ensure data integrity.)

import java.sql.*;
import java.util.Scanner;

public class ProductCRUD {

    // JDBC connection parameters
    private static final String DB_URL = "jdbc:mysql://localhost:3306/your_database";
    private static final String DB_USER = "your_username";
    private static final String DB_PASSWORD = "your_password";

    public static void main(String[] args) {
        try (Connection connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD)) {
            connection.setAutoCommit(false); // Enable transaction handling

            Scanner scanner = new Scanner(System.in);
            boolean exit = false;

            while (!exit) {
                System.out.println("\nMenu:");
                System.out.println("1. Create Product");
                System.out.println("2. Read Products");
                System.out.println("3. Update Product");
                System.out.println("4. Delete Product");
                System.out.println("5. Exit");
                System.out.print("Choose an option: ");
                int choice = scanner.nextInt();

                switch (choice) {
                    case 1 -> createProduct(connection, scanner);
                    case 2 -> readProducts(connection);
                    case 3 -> updateProduct(connection, scanner);
                    case 4 -> deleteProduct(connection, scanner);
                    case 5 -> exit = true;
                    default -> System.out.println("Invalid option. Please try again.");
                }
            }

        } catch (SQLException e) {
            System.out.println("Database error: " + e.getMessage());
        }
    }

    private static void createProduct(Connection connection, Scanner scanner) {
        try {
            System.out.print("Enter Product Name: ");
            String name = scanner.next();
            System.out.print("Enter Price: ");
            double price = scanner.nextDouble();
            System.out.print("Enter Quantity: ");
            int quantity = scanner.nextInt();

            String query = "INSERT INTO Product (ProductName, Price, Quantity) VALUES (?, ?, ?)";
            try (PreparedStatement stmt = connection.prepareStatement(query)) {
                stmt.setString(1, name);
                stmt.setDouble(2, price);
                stmt.setInt(3, quantity);
                stmt.executeUpdate();
            }

            connection.commit();
            System.out.println("Product created successfully!");

        } catch (SQLException e) {
            System.out.println("Error creating product: " + e.getMessage());
            try {
                connection.rollback();
            } catch (SQLException rollbackEx) {
                System.out.println("Error rolling back transaction: " + rollbackEx.getMessage());
            }
        }
    }

    private static void readProducts(Connection connection) {
        try {
            String query = "SELECT * FROM Product";
            try (Statement stmt = connection.createStatement();
                 ResultSet rs = stmt.executeQuery(query)) {

                System.out.println("\nProducts:");
                while (rs.next()) {
                    System.out.printf("ID: %d, Name: %s, Price: %.2f, Quantity: %d%n",
                            rs.getInt("ProductID"), rs.getString("ProductName"),
                            rs.getDouble("Price"), rs.getInt("Quantity"));
                }
            }

        } catch (SQLException e) {
            System.out.println("Error reading products: " + e.getMessage());
        }
    }

    private static void updateProduct(Connection connection, Scanner scanner) {
        try {
            System.out.print("Enter Product ID to update: ");
            int id = scanner.nextInt();
            System.out.print("Enter new Price: ");
            double price = scanner.nextDouble();

            String query = "UPDATE Product SET Price = ? WHERE ProductID = ?";
            try (PreparedStatement stmt = connection.prepareStatement(query)) {
                stmt.setDouble(1, price);
                stmt.setInt(2, id);
                int rows = stmt.executeUpdate();
                if (rows > 0) {
                    connection.commit();
                    System.out.println("Product updated successfully!");
                } else {
                    System.out.println("Product not found.");
                }
            }

        } catch (SQLException e) {
            System.out.println("Error updating product: " + e.getMessage());
            try {
                connection.rollback();
            } catch (SQLException rollbackEx) {
                System.out.println("Error rolling back transaction: " + rollbackEx.getMessage());
            }
        }
    }

    private static void deleteProduct(Connection connection, Scanner scanner) {
        try {
            System.out.print("Enter Product ID to delete: ");
            int id = scanner.nextInt();

            String query = "DELETE FROM Product WHERE ProductID = ?";
            try (PreparedStatement stmt = connection.prepareStatement(query)) {
                stmt.setInt(1, id);
                int rows = stmt.executeUpdate();
                if (rows > 0) {
                    connection.commit();
                    System.out.println("Product deleted successfully!");
                } else {
                    System.out.println("Product not found.");
                }
            }

        } catch (SQLException e) {
            System.out.println("Error deleting product: " + e.getMessage());
            try {
                connection.rollback();
            } catch (SQLException rollbackEx) {
                System.out.println("Error rolling back transaction: " + rollbackEx.getMessage());
            }
        }
    }
}


Question 2 (2. Develop a Java application using JDBC and MVC architecture to manage student data. The application should:
Use a Student class as the model with fields like StudentID, Name, Department, and Marks.
Include a database table to store student data.
Allow the user to perform CRUD operations through a simple menu-driven view.
Implement database operations in a separate controller class.)



import java.sql.*;
import java.util.Scanner;

// Model: Student Class
class Student {
    private int studentID;
    private String name;
    private String department;
    private double marks;

    // Getters and Setters
    public int getStudentID() {
        return studentID;
    }

    public void setStudentID(int studentID) {
        this.studentID = studentID;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    public double getMarks() {
        return marks;
    }

    public void setMarks(double marks) {
        this.marks = marks;
    }
}

// Controller: Handles database operations
class StudentController {
    private static final String DB_URL = "jdbc:mysql://localhost:3306/your_database";
    private static final String DB_USER = "your_username";
    private static final String DB_PASSWORD = "your_password";

    public void createStudent(Student student) {
        try (Connection connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
             PreparedStatement stmt = connection.prepareStatement(
                     "INSERT INTO Student (StudentID, Name, Department, Marks) VALUES (?, ?, ?, ?)")) {
            stmt.setInt(1, student.getStudentID());
            stmt.setString(2, student.getName());
            stmt.setString(3, student.getDepartment());
            stmt.setDouble(4, student.getMarks());
            stmt.executeUpdate();
            System.out.println("Student added successfully!");
        } catch (SQLException e) {
            System.out.println("Error adding student: " + e.getMessage());
        }
    }

    public void readStudents() {
        try (Connection connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
             Statement stmt = connection.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM Student")) {
            System.out.println("\nStudents List:");
            while (rs.next()) {
                System.out.printf("ID: %d, Name: %s, Department: %s, Marks: %.2f%n",
                        rs.getInt("StudentID"), rs.getString("Name"),
                        rs.getString("Department"), rs.getDouble("Marks"));
            }
        } catch (SQLException e) {
            System.out.println("Error retrieving students: " + e.getMessage());
        }
    }

    public void updateStudentMarks(int studentID, double newMarks) {
        try (Connection connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
             PreparedStatement stmt = connection.prepareStatement(
                     "UPDATE Student SET Marks = ? WHERE StudentID = ?")) {
            stmt.setDouble(1, newMarks);
            stmt.setInt(2, studentID);
            int rows = stmt.executeUpdate();
            if (rows > 0) {
                System.out.println("Student marks updated successfully!");
            } else {
                System.out.println("Student not found.");
            }
        } catch (SQLException e) {
            System.out.println("Error updating marks: " + e.getMessage());
        }
    }

    public void deleteStudent(int studentID) {
        try (Connection connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
             PreparedStatement stmt = connection.prepareStatement(
                     "DELETE FROM Student WHERE StudentID = ?")) {
            stmt.setInt(1, studentID);
            int rows = stmt.executeUpdate();
            if (rows > 0) {
                System.out.println("Student deleted successfully!");
            } else {
                System.out.println("Student not found.");
            }
        } catch (SQLException e) {
            System.out.println("Error deleting student: " + e.getMessage());
        }
    }
}

// View: Main Application
public class StudentManagementApp {
    public static void main(String[] args) {
        StudentController controller = new StudentController();
        Scanner scanner = new Scanner(System.in);
        boolean exit = false;

        while (!exit) {
            System.out.println("\nMenu:");
            System.out.println("1. Add Student");
            System.out.println("2. View Students");
            System.out.println("3. Update Student Marks");
            System.out.println("4. Delete Student");
            System.out.println("5. Exit");
            System.out.print("Choose an option: ");
            int choice = scanner.nextInt();

            switch (choice) {
                case 1 -> {
                    Student student = new Student();
                    System.out.print("Enter Student ID: ");
                    student.setStudentID(scanner.nextInt());
                    System.out.print("Enter Name: ");
                    student.setName(scanner.next());
                    System.out.print("Enter Department: ");
                    student.setDepartment(scanner.next());
                    System.out.print("Enter Marks: ");
                    student.setMarks(scanner.nextDouble());
                    controller.createStudent(student);
                }
                case 2 -> controller.readStudents();
                case 3 -> {
                    System.out.print("Enter Student ID: ");
                    int studentID = scanner.nextInt();
                    System.out.print("Enter new Marks: ");
                    double marks = scanner.nextDouble();
                    controller.updateStudentMarks(studentID, marks);
                }
                case 4 -> {
                    System.out.print("Enter Student ID: ");
                    int studentID = scanner.nextInt();
                    controller.deleteStudent(studentID);
                }
                case 5 -> exit = true;
                default -> System.out.println("Invalid option. Please try again.");
            }
        }
        scanner.close();
    }
}
