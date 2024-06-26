# INFO 4345 Final Project - Website Security Enhancement

# Table of Contents

- [Group Members](#group-members)
- [Web Application Overview](#webapp)
- [Objectives](#objectives)
- [Database Setup](#db)
- [Enhancements](#enhancements)
  - [Input Validation](#input-validation)
  - [Authentication](#authentication)
  - [Authorization](#authorization)
  - [XSS and CSRF Prevention](#xss-and-csrf-prevention)
  - [Database Security](#database-security)
  - [File Security](#file-security)
- [Reference](#reference)

## Group Members
Wan Hamzah Iyad bin Wan Adlan (2115449) - Leader
- Input Validation
- XSS & CSRF Defence

Muhammad bin Abas (2113201)
- Authentication
- Authorization

Muhammad Arif Faisal bin Zahari (2117277)
- Database Security
- File Security

## <a name="webapp"/>Web Application Overview
**Name of Website:** SweatFactory

### Introduction

<img src="https://github.com/iyadadalan/Final_Assessment/assets/92302244/79bd08a3-340b-4f27-a9df-3605a943e9e1" width="35%" alt="SWEATFACTORY">

SweatFactory is an engaging website designed specifically for a mock organization focused on gym services and promoting a healthy lifestyle. The website offers a variety of features ranging from exercise tutorials, dietary guidance, lifestyle management tips, and a user-friendly BMI calculator. This platform stands out by providing a comprehensive fitness experience that is both informative and engaging.

Key features of the SweatFactory website include:
- **Gym Registration:** Prospective gym members can navigate from the "Register Now" button on the Home page to the Registration page, where various membership plans are detailed, allowing users to sign up and access the gym facilities located in Kuala Lumpur.
- **Educational Content:** Extensive information is available on home exercises and detailed diet plans, including various nutritional strategies.
- **Interactive Tools:** Features like a BMI calculator help users actively engage with their health, offering valuable tools to assess and maintain their wellness.

## Objectives
1. **Improving Data Integrity and Security:** Implement robust security measures to protect personal data of users from unauthorized access, alteration, or destruction.
2. **Enhancing User Privacy:** Strengthen privacy protocols to ensure that all user data is handled confidentially and in compliance with applicable privacy laws and regulations.
3. **Preventing Web Attacks:** Introduce advanced safeguards to defend against common web vulnerabilities such as Cross-Site Scripting (XSS), Cross-Site Request Forgery (CSRF), and SQL injection.
4. **Ensuring Secure User Interactions:** Deploy mechanisms like Content Security Policies (CSP) to secure user interactions with the website.

##  <a name="db"/>Database Setup

Step 1: Create a new database:
```sql
CREATE DATABASE final_assessment;
```
Step 2: Switch to the new database:
```sql
USE final_assessment;
```
Step 3: Create the users table:
```sql
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    gender ENUM('male', 'female', 'Other') NOT NULL,
    user_type ENUM('user', 'admin') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
Step 4:  Create a stored procedure to fetch user details by email:
```sql
-- Create the stored procedure to get user by email
DELIMITER //

CREATE PROCEDURE GetUserByEmail(IN p_email VARCHAR(255))
BEGIN
    SELECT * FROM users WHERE email = p_email LIMIT 1;
END //

DELIMITER ;

```
```sql
-- Create the stored procedure to register a new user
DELIMITER //

CREATE PROCEDURE RegisterUser(
    IN p_user_id VARCHAR(255),
    IN p_email VARCHAR(255),
    IN p_password VARCHAR(255),
    IN p_username VARCHAR(255),
    IN p_gender ENUM('male', 'female'),
    IN p_user_type ENUM('admin', 'user')
)
BEGIN
    INSERT INTO users (user_id, email, password, username, gender, user_type)
    VALUES (p_user_id, p_email, p_password, p_username, p_gender, p_user_type);
END //

DELIMITER ;
```
Step 5: Insert admin account
```sql
-- Username: admin 
-- Password: Admin123

INSERT INTO users (username, email, password, gender, user_type)
VALUES ('admin', 'admin@example.com', '$2y$10$rqOZP1q4uLVgu/9TfZ6rZesQ8y9iynGs1UGfOMAV5brG58cMyOS72', 'Male', 'admin');
```


## Enhancements

### <a name="input-validation"/>Input Validation - Iyad
For input validation, extensive use of both client-side and server-side strategies was implemented across all forms. Each input field in forms such as signup and login uses regex patterns that enforce specific formatting rules. These rules ensure that the input matches expected patterns such as valid email formats, strong password requirements (including upper and lowercase letters, numbers, and minimum length), and valid full names. On the client-side, JavaScript regex validation is used to provide immediate feedback before form submission. On the server-side, PHP scripts sanitize and validate data to prevent any malicious input from being processed or stored.

- Client-side validation (lifestyle.js)
```js
function validateForm() {
    var age = document.getElementById("age").value;
    var height = document.getElementById("height").value;
    var weight = document.getElementById("weight").value;

    if (!age || age < 0 || age > 120) {
        alert("Please enter a valid age.");
        return false;
    }

    if (!height || height <= 0) {
        alert("Please enter a valid height.");
        return false;
    }

    if (!weight || weight <= 0) {
        alert("Please enter a valid weight.");
        return false;
    }

    return true; // Proceed with form submission if validations pass
}
```
 
- Client-side Input Validation using RegEx (signin_validation.js)
```js
document.addEventListener("DOMContentLoaded", function () {
    document.getElementById("form").onsubmit = function () {
        var email = document.getElementById("email").value;
        var password = document.getElementById("password").value;

        var emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
        var passwordRegex = /^(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}$/;

        if (!emailRegex.test(email)) {
            alert("Please enter a valid email address.");
            return false;
        }

        if (!passwordRegex.test(password)) {
            alert("Password must contain at least one number, one uppercase and one lowercase letter, and at least 8 or more characters");
            return false;
        }

        return true;
    };
});
```
- Server-side Input Validation (signin.php)
```php
       // Regex patterns for validation
       $regex_email = "/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/";
       $regex_password = "/^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])(?=.*[a-zA-Z]).{8,}$/";
   
       // Validate input
       if (!preg_match($regex_email, $email) || !preg_match($regex_password, $password)) {
           echo '<script>
               alert("Invalid email or password format!");
               window.location.href = "signin.php";
           </script>';
           exit;
       }
```

### <a name="authentication"/>Authentication - Muhammad
Details on the authentication mechanisms implemented.

The login section will handle user authentication by validating the user's email and password during login. The authentication process involves verifying the supplied credentials against stored user data in a MySQL database. When the user want to sign in, there will be a comaprison login in the application with the Hashed Password.

[signup.php](Enhanced/register_user/signup.php)

## Form Handling
```php
if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $email = htmlspecialchars($_POST['email']);
    $password = htmlspecialchars($_POST['password']);
    $user_name = htmlspecialchars($_POST['user_name']);
    $gender = htmlspecialchars($_POST['gender']);
    $user_type = 'user';
```

- Checks if the request method is POST, indicating that the form has been submitted.

## Password Hashing
```php
    if (!empty($email) && !empty($password)) {
        $password_hash = password_hash($password, PASSWORD_DEFAULT);
        $user_id = random_num(5);
```

- Ensures that both email and password fields are not empty.
- Hashes the password using password_hash with the default algorithm (currently bcrypt).
- Generates a random user ID using the random_num function.

[signin.php](Enhanced/register_user/signin.php)

## Credential Verification
```php
if ($result && $result->num_rows > 0) {
    $user_data = $result->fetch_assoc();
    
    if (password_verify($password, $user_data['password'])) {
        $_SESSION['user_id'] = $user_data['user_id'];
        $_SESSION['username'] = $user_data['username'];
        $_SESSION['email'] = $user_data['email'];
        $_SESSION['user_type'] = $user_data['user_type'];
```

- Checks if the result set contains any rows, indicating that a user with the provided email exists.
- Fetches the user data and verifies the supplied password against the hashed password stored in the database using password_verify.

## Session Management and Redirection
```php
if ($user_data['user_type'] == 'admin') {
    header('Location: ../adminpage/admin_index.php');
} elseif ($user_data['user_type'] == 'user') {
    header('Location: ../au_userpage/user_index.php');
}
exit;
```

- If password verification is successful, sets session variables with user details.
- Redirects the user to the appropriate page based on their user type (admin or user).

## Protecting Website with .htaccess file

This method uses the .htaccess and .htpasswd files to restrict access to specific areas of a website by requiring user authentication. Below is a detailed explanation of how this method works and the purpose of each line in the provided code.

.htaccess File
The .htaccess file is a configuration file used by Apache web servers to control access to directories and to manage URL rewriting, redirection, and other server settings.

Contents of the [.htaccess](Enhanced/.htaccess) File
```
AuthUserFile C:\xampp\htdocs\Final_Assessment\Enhanced\.htpasswd
AuthName "Password Protected Area"
AuthType Basic

<limit GET POST>
require valid-user
</limit>
```
.htpasswd File
The .htpasswd file contains the usernames and hashed passwords for users authorized to access the protected area. Each line in the file represents a single user in the format username:password.

Contents of the [.htpaswwd](Enhanced/.htpasswd) File
```
Hackers4Flat:IyadMadArif
```

### <a name="authorization"/>Authorization - Muhammad
Explanation of the authorization processes enhanced.

Authorization is the process of determining what actions a user is allowed to perform and what resources they can access within a system. It follows the authentication process, which verifies the user's identity.

## Key Concepts:

1. Roles: Groups of permissions assigned to users (e.g., Admin, User).
2. Permissions: Specific rights to perform actions or access resources (e.g., read, write).
3. Access Control Models:
  - Role-Based Access Control (RBAC): Users are assigned roles, and roles have permissions.
  - Attribute-Based Access Control (ABAC): Access is granted based on attributes (user, resource, environment).

## Process:
User Authentication: The user provides credentials and is authenticated.
Assigning Roles/Permissions: The system assigns roles/permissions to the user.
Access Request: The user attempts to access a resource.
Policy Evaluation: The system evaluates the request against access policies.
Decision Making: The system allows or denies access based on the evaluation.
Enforcement: The system enforces the decision, granting or denying access.

## Session Management
Session Start

Starting a session is essential for maintaining state between the user's interactions with a web application. This is commonly done using session_start() in PHP.
```php
session_start();
```

- This function starts a new session or resumes an existing one.
- It enables storing user information across different pages.
- Typically used at the beginning of a script to access session variables.

Session Destroy

Ending a session ensures that all session data is cleared and the user is effectively logged out. This involves unsetting session variables, deleting the session cookie, and destroying the session.
Unset All Session Variables

```php
$_SESSION = array();
```
- Clears the $_SESSION array, removing all session data.

Delete the Session Cookies
```php
if (ini_get("session.use_cookies")) {
    $params = session_get_cookie_params();
    setcookie(session_name(), '', time() - 42000,
        $params["path"], $params["domain"],
        $params["secure"], $params["httponly"]
    );
}
```
- Checks if sessions are using cookies and deletes the session cookie by setting its expiration time to the past.

Destroy the Session
```php
session_destroy();
```
- Destroys the session data on the server.

## HttpOnly and Secure Cookies

Description of Apache Configuration:

Loading the Headers Module
![Screenshot 2024-06-25 145035](https://github.com/iyadadalan/Final_Assessment/assets/122088412/85632af3-e9bf-43bb-8642-3939e9b0b3fa)

- This line loads the mod_headers module, which allows for the manipulation of HTTP headers.

Setting HttpOnly and Secure Attribute
![Screenshot 2024-06-25 161641](https://github.com/iyadadalan/Final_Assessment/assets/122088412/4cdb8ff5-fa50-4cb8-80d4-d1c59f50da8a)

- This modified directive ensures that both HttpOnly and Secure attributes are added to all Set-Cookie headers, providing additional security by ensuring the cookies are protected both from client-side access and during transmission.

## SSL certificate for localhost

SSL (Secure Sockets Layer) certificates are essential for encrypting data transmitted between a server and a client, ensuring secure communication over the internet. Belwo is the process of configuration:

Download important file
![Screenshot 2024-06-25 162923](https://github.com/iyadadalan/Final_Assessment/assets/122088412/79ae513e-94fe-45d4-8815-975265ce8e07)

Chnage the name of DNS.1
![Screenshot 2024-06-25 164051](https://github.com/iyadadalan/Final_Assessment/assets/122088412/c5ad2146-fc99-411e-aabe-b2e284960490)

Restart the apache and the the website is secured with https://
![Screenshot 2024-06-25 150314](https://github.com/iyadadalan/Final_Assessment/assets/122088412/07767ac3-9306-4adb-b2d6-2ebfd1b322ef)

### <a name="xss-and-csrf-prevention"/>XSS and CSRF Prevention - Iyad
Strategies used to prevent XSS and CSRF vulnerabilities.

To combat XSS and CSRF vulnerabilities, several measures have been taken:

- **Content Security Policy (CSP):** A strict CSP is implemented to control the sources of content that can be loaded on the website. This includes restrictions on scripts, styles, and images to trusted sources and disallows inline scripts and styles to mitigate XSS attacks. A CSP has been added for each page, taking into account the specific resources and scripts utilized on that page.  By customizing the CSP directives to match the unique requirements of each page, the security policy provides precise control over resource loading and script execution, thereby enhancing the overall security posture of the website.
  - join_us.php:
```html
<meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    script-src 'self' https://code.jquery.com https://cdn.jsdelivr.net https://ajax.googleapis.com 'unsafe-inline';
    style-src 'self' https://fonts.googleapis.com 'unsafe-inline';
    img-src 'self' https://images.pexels.com data:;
    font-src 'self' https://fonts.gstatic.com;
    frame-src 'self' https://www.google.com;
    connect-src 'self';
    media-src 'self';
    object-src 'none';
    child-src 'none';">
  ```
- **CSRF Tokens:** CSRF tokens are used in all forms to ensure that requests are originated from the website's forms and not from external sources. This token is generated on the server-side, embedded in forms, and verified on each form submission.
  - CSRF Token generation and verification (signin.php)
```php
// CSRF token generation and verification
if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = generate_csrf_token();
}

if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!verify_csrf_token($_POST['csrf_token'])) {
        die('CSRF token validation failed.');
    }
}
```
CSRF token in hidden ``<form>`` tag:
```html
<input type="hidden" name="csrf_token" value="<?php echo htmlspecialchars($csrf_token); ?>">
```
- **Sanitization:** All inputs from users are sanitized using PHP to encode or strip out potentially malicious characters and scripts, reducing the risk of XSS attacks.
  - join_us.php
```php
// Sanitize and validate inputs
$fullName = sanitize_input($_POST['fullName']);
$email = filter_var(sanitize_input($_POST['email']), FILTER_VALIDATE_EMAIL);
$phone = sanitize_input($_POST['phone']); // Additional validation can be applied if needed
```
- **Update 3rd-Party Libraries:** The jQuery library has been updated to the latest version to mitigate any known vulnerabilities associated with older versions. Also included additional security attributes within the <script> tag. Example below is from lifestyle.php
```html
<script src="https://code.jquery.com/jquery-3.7.1.min.js" integrity="sha256-/JqT3SQfawRcv/BIHPThkBvs0OEvtFFmqPF/lYI/Cxo=" crossorigin="anonymous"></script>
<script src="https://code.jquery.com/ui/1.13.3/jquery-ui.js" integrity="sha256-J8ay84czFazJ9wcTuSDLpPmwpMXOm573OUtZHPQqpEU=" crossorigin="anonymous"></script>
```

These strategies collectively enhance the security of the website by validating input data rigorously and defending against common web vulnerabilities like XSS and CSRF.

### <a name="database-security"/>Database Security - Arif

Security Principles Implemented
1. Prepared Statements <br>
Both the login and registration scripts use prepared statements to interact with the database. Prepared statements are a powerful way to mitigate SQL injection attacks. By using prepared statements, SQL queries are precompiled by the database, and user inputs are treated as data rather than executable code.

#### Vulnerable SQL Query (Without Prepared Statement) ####
- login.php
```php
    $query = "SELECT * FROM users WHERE email = '$email'";
    $result = $conn->query($query);
```
- register.php
```php
    // Vulnerable to SQL injection
    $query = "INSERT INTO users (email, password) VALUES ('$email', '$hashed_password')";
    if ($conn->query($query) === TRUE) {
        // Registration successful, redirect to login page
        header("Location: login.php");
        exit;
    } else {
        $error = "Email already exists";
    }
}
```
Secure SQL Query (With Prepared Statement)

- login.php 
```php
if ($stmt = $conn->prepare("CALL GetUserByEmail(?)")) {
    $stmt->bind_param("s", $email);
    $stmt->execute();
    $result = $stmt->get_result();
}
```
- register.php
```php
if ($stmt = $conn->prepare("CALL RegisterUser(?, ?)")) {
    $stmt->bind_param("ss", $email, $hashed_password);
    $stmt->execute();
}
```

2. Stored Procedures <br>
Stored procedures encapsulate SQL queries and logic within the database and reduces the risk of SQL injection. They are stored in the database and can be executed by calling them from applications. The provided code already uses stored procedures (GetUserByEmail and RegisterUser), which is a good practice.

Vulnerable SQL Query (Without Stored Procedure)
```php
$email = $_POST['email'];
$query = "SELECT * FROM users WHERE email = '$email'";
$result = $conn->query($query);
```
Secure SQL Query (With Stored Procedure)
Stored Procedure Definition:
```sql
DELIMITER $$
CREATE PROCEDURE GetUserByEmail(IN userEmail VARCHAR(255))
BEGIN
    SELECT * FROM users WHERE email = userEmail;
END $$
DELIMITER ;
```
PHP Code to Call Stored Procedure:
```php
if ($stmt = $conn->prepare("CALL GetUserByEmail(?)")) {
    $stmt->bind_param("s", $email);
    $stmt->execute();
    $result = $stmt->get_result();
    // process result
    $stmt->close();
}
```
### <a name="file-security"/>File Security - Arif
Methods used to secure file management and storage.
1. Restrict Diretory Listing <br>
Directory listing in a web server like Apache (which XAMPP uses) can expose the contents of directories to users, potentially revealing sensitive information or files that should not be publicly accessible. Restricting directory listing is an important security measure.

  Step 1: Navigate to C:/xampp/apache/conf/. <br>
  ![image](https://github.com/iyadadalan/Final_Assessment/assets/59950029/e4fe23de-daa5-47b4-92c7-db734321f654)
  <br><br>
  Step 2: Open the httpd.conf file with a text editor like Notepad++ or any other text editor. <br>
  ![image](https://github.com/iyadadalan/Final_Assessment/assets/59950029/255cff2c-27a3-410b-a186-8c535dfc8928)
  <br><br>
  Step 3: Change the Options directive to remove Indexes <br>
```
<Directory "C:/xampp/htdocs">
    Options FollowSymLinks Includes ExecCGI
    AllowOverride All
    Require all granted
</Directory>
```
- Options: This directive controls which server features are enabled or disabled in that directory.
- Indexes: When enabled, if a user requests a directory and no index file (like index.html or index.php) is present, Apache will display a listing of the directory's contents.
- FollowSymLinks: Allows Apache to follow symbolic links.
- Includes: Allows server-side includes (SSI).
- ExecCGI: Allows execution of CGI scripts.

### <a name="reference"/>Reference
- https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation
- https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- https://portswigger.net/web-security/cross-site-scripting/preventing
