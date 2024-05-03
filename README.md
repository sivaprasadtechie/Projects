Password Manager GUI

This Java application provides a graphical user interface (GUI) for managing passwords. It allows users to generate strong passwords based on their name, birth year, and favorite color, and provides features to check the strength and breach status of passwords. The application integrates with a MySQL database for storing and retrieving password-related data.

Key Features:

--> Generate strong passwords using a combination of user-provided information.
--> Check password strength based on criteria such as uppercase letters, digits, and special characters.
--> Verify if a password has been breached using the Have I Been Pwned API.
--> Store password-related data (name, birth year, color, and generated password) in a MySQL database.
--> Retrieve stored data from the database and display it in the GUI.

Technologies Used:

--> Java Swing for building the graphical user interface.
--> JDBC for connecting to and interacting with the MySQL database.
--> Have I Been Pwned API for checking if passwords have been breached.
--> MySQL for storing password-related data.

Usage:

--> Clone the repository.
--> Set up a MySQL database with the required schema (passwords table).
--> Update the database connection details in the DatabaseConnection class.
--> Compile and run the PasswordManagerGUI class to launch the application.
--> Use the GUI to generate passwords, check their strength and breach status, and manage password-related data.
