import java.awt.*;
import java.awt.event.*;
import javax.swing.*;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.security.NoSuchAlgorithmException;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.KeySpec;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.PBEKeySpec;
import java.util.Base64;
import java.util.Random;
import java.awt.datatransfer.StringSelection;
import java.awt.datatransfer.Clipboard;
import java.awt.Toolkit;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.PreparedStatement;
import java.sql.ResultSet;


public class PasswordManagerGUI {

	public static String generatePassword(String name, String birthYear, String color) {
	    String passwordString = name + birthYear + color;
	    passwordString = passwordString.replace("a", "@")
	            .replace("e", "3")
	            .replace("i", "1")
	            .replace("o", "0");

	    // Create a string of uppercase letters
	    String uppercaseChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
	    
	    char[] passwordArray = passwordString.toCharArray();
	    Random random = new Random();
	    for (int i = 0; i < passwordArray.length; i++) {
	        int j = random.nextInt(passwordArray.length);
	        char temp = passwordArray[i];
	        passwordArray[i] = passwordArray[j];
	        passwordArray[j] = temp;
	    }

	    // Insert uppercase characters at random positions in the password
	    for (int i = 0; i < 2; i++) {
	        int position = random.nextInt(passwordArray.length);
	        char uppercaseChar = uppercaseChars.charAt(random.nextInt(uppercaseChars.length()));
	        passwordArray[position] = uppercaseChar;
	    }

	    return new String(passwordArray);
	}


    public static String checkPasswordStrength(String password) {
    	    boolean hasUppercase = false;
    	    boolean hasLowercase = false;
    	    boolean hasDigit = false;
    	    boolean hasSpecialCharacter = false;

    	    for (char c : password.toCharArray()) {
    	        if (Character.isUpperCase(c)) {
    	            hasUppercase = true;
    	        }
    	        if (Character.isLowerCase(c)) {
    	            hasLowercase = true;
    	        }
    	        if (Character.isDigit(c)) {
    	            hasDigit = true;
    	        }
    	        if ("!@#$%^&*()_+-={};:'\"<>,.?/|\\\\".indexOf(c) >= 0) {
    	            hasSpecialCharacter = true;
    	        }
    	    }

    	    // Modify the criteria as needed
    	    if (hasUppercase && hasDigit) {
    	        return "Strong";
    	    } else {
    	        return "Weak";
    	    }
    }

    public static boolean isPasswordBreached(String password) {
        try {
            String sha1Hash = getSHA1Hash(password);
            String firstFiveChars = sha1Hash.substring(0, 5);
            String restOfHash = sha1Hash.substring(5);

            URL url = new URL("https://api.pwnedpasswords.com/range/" + firstFiveChars);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();

            connection.setRequestMethod("GET");
            connection.setRequestProperty("User-Agent", "PasswordManager");

            int responseCode = connection.getResponseCode();
            if (responseCode == 200) {
                BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
                String line;
                while ((line = reader.readLine()) != null) {
                    if (line.startsWith(restOfHash)) {
                        return true;
                    }
                }
                reader.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        return false;
    }

    private static String getSHA1Hash(String password) throws NoSuchAlgorithmException, InvalidKeySpecException {
        byte[] salt = new byte[16];
        Random random = new Random();
        random.nextBytes(salt);

        KeySpec spec = new PBEKeySpec(password.toCharArray(), salt, 65536, 128);
        SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");

        byte[] hash = factory.generateSecret(spec).getEncoded();
        byte[] hashWithSalt = new byte[hash.length + salt.length];
        System.arraycopy(hash, 0, hashWithSalt, 0, hash.length);
        System.arraycopy(salt, 0, hashWithSalt, hash.length, salt.length);

        return Base64.getEncoder().encodeToString(hashWithSalt);
    }
    

    public class DatabaseConnection {
        private static Connection connection = null;

        public static Connection getConnection() {
            if (connection != null) {
                return connection;
            }

            try {
                String url = "jdbc:mysql://localhost:3306/passwords";
                String user = "//your mysql workbench root name";
                String password = "//your password";
                connection = DriverManager.getConnection(url, user, password);
            } catch (SQLException e) {
                e.printStackTrace();
            }
            return connection;
        }
    }


    public static void main(String[] args) {
        JFrame frame = new JFrame("Password Manager");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 350);

        JPanel panel = new JPanel(new GridBagLayout());
        frame.add(panel);

        Font largeFont = new Font("SansSerif", Font.PLAIN, 16);

        GridBagConstraints c = new GridBagConstraints();
        c.fill = GridBagConstraints.HORIZONTAL;
        c.insets = new Insets(5, 5, 5, 5);

        JLabel nameLabel = new JLabel("Name:");
        nameLabel.setFont(largeFont);
        c.gridx = 0;
        c.gridy = 0;
        panel.add(nameLabel, c);

        JTextField nameField = new JTextField(20);
        c.gridx = 1;
        c.gridy = 0;
        panel.add(nameField, c);

        JLabel birthYearLabel = new JLabel("Birth Year:");
        birthYearLabel.setFont(largeFont);
        c.gridx = 0;
        c.gridy = 1;
        panel.add(birthYearLabel, c);

        JTextField birthYearField = new JTextField(20);
        c.gridx = 1;
        c.gridy = 1;
        panel.add(birthYearField, c);

        JLabel colorLabel = new JLabel("Favorite Color:");
        colorLabel.setFont(largeFont);
        c.gridx = 0;
        c.gridy = 2;
        panel.add(colorLabel, c);

        JTextField colorField = new JTextField(20);
        c.gridx = 1;
        c.gridy = 2;
        panel.add(colorField, c);

        JButton generateButton = new JButton("Generate Password");
        generateButton.setFont(largeFont);
        generateButton.setBackground(new Color(50, 205, 50));
        c.gridx = 0;
        c.gridy = 3;
        c.gridwidth = 2;
        panel.add(generateButton, c);

        JButton checkStrengthButton = new JButton("Check Strength");
        checkStrengthButton.setFont(largeFont);
        checkStrengthButton.setBackground(new Color(255, 69, 0));
        c.gridx = 0;
        c.gridy = 4;
        c.gridwidth = 2;
        panel.add(checkStrengthButton, c);

        JPasswordField passwordField = new JPasswordField();
        passwordField.setFont(largeFont);
        c.gridx = 0;
        c.gridy = 5;
        c.gridwidth = 2;
        panel.add(passwordField, c);

        JLabel passwordStrengthLabel = new JLabel("Password Strength: ");
        passwordStrengthLabel.setFont(largeFont);
        c.gridx = 0;
        c.gridy = 6;
        c.gridwidth = 2;
        panel.add(passwordStrengthLabel, c);

        JLabel passwordBreachLabel = new JLabel("Password Breach: ");
        passwordBreachLabel.setFont(largeFont);
        c.gridx = 0;
        c.gridy = 7;
        c.gridwidth = 2;
        panel.add(passwordBreachLabel, c);

        JButton copyButton = new JButton("Copy Password");
        copyButton.setFont(largeFont);
        c.gridx = 0;
        c.gridy = 8;
        c.gridwidth = 2;
        panel.add(copyButton, c);
        
        generateButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String name = nameField.getText();
                String birthYear = birthYearField.getText();
                String color = colorField.getText();
                String password = generatePassword(name, birthYear, color);
                passwordField.setText(password);
            }
        });

        checkStrengthButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                // Retrieve data from the database
                try {
                    Connection dbConnection = DatabaseConnection.getConnection();
                    String enteredPassword = new String(passwordField.getPassword());

                    if (enteredPassword.isEmpty()) {
                        JOptionPane.showMessageDialog(null, "Please generate a password or enter one.");
                        return;
                    }

                    String strength = checkPasswordStrength(enteredPassword);

                    // Add code to retrieve data from the database and display it
                    String query = "SELECT * FROM passwords WHERE generatedPassword = ?";
                    PreparedStatement preparedStatement = dbConnection.prepareStatement(query);
                    preparedStatement.setString(1, enteredPassword);
                    ResultSet resultSet = preparedStatement.executeQuery();

                    if (resultSet.next()) {
                        nameField.setText(resultSet.getString("name"));
                        birthYearField.setText(resultSet.getString("birthYear"));
                        colorField.setText(resultSet.getString("color"));
                    }

                    passwordStrengthLabel.setText("Password Strength: " + strength);
                    passwordBreachLabel.setText("Password Breach: " + (isPasswordBreached(enteredPassword) ? "Yes" : "No"));
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
        });


        copyButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String passwordToCopy = new String(passwordField.getPassword());
                StringSelection selection = new StringSelection(passwordToCopy);
                Clipboard clipboard = Toolkit.getDefaultToolkit().getSystemClipboard();
                clipboard.setContents(selection, null);
            }
        });
        
        generateButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                String name = nameField.getText();
                String birthYear = birthYearField.getText();
                String color = colorField.getText();
                String password = generatePassword(name, birthYear, color);
                passwordField.setText(password);

                // Insert data into the database
                try {
                    Connection dbConnection = DatabaseConnection.getConnection();
                    String insertQuery = "INSERT INTO passwords (name, birthYear, color, generatedPassword) VALUES (?, ?, ?, ?)";
                    PreparedStatement preparedStatement = dbConnection.prepareStatement(insertQuery);
                    preparedStatement.setString(1, name);
                    preparedStatement.setString(2, birthYear);
                    preparedStatement.setString(3, color);
                    preparedStatement.setString(4, password);
                    preparedStatement.executeUpdate();
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
        });
        

        frame.setVisible(true);
    }
    
}
