# PHP Owasp Top 10

### A01 - Broken Access Control
- Users can only perform actions or access resources in their authorized scope.
- Check if the endpoints are responding to objects IDs Insecure direct object reference (IDOR).
- Use object uuid always when possible.
##### **Correct code apporach:**
```php
$sth = $dbh->prepare("SELECT * FROM orders WHERE order_id = ? AND user_id = ?");
$sth->execute([$orderId, $userId]);
```
### A02 - Cryptographic Failures
- Use strong hash functions like password_hash.
- Use strong algorithms like bcrypt, sha256 only with salt.
- All passwords and secrete tokens must be saved with strong hash.
- If nginx config, must use https with ssl.
##### **Correct code apporach:**
```php
$stored_hash = password_hash($_POST['password'], PASSWORD_BCRYPT, ['cost'=>13]);

// verify
if (password_verify($_POST['password'], $stored_hash)) {
    echo 'Password is valid!';
}
```
### A03 - Injection (SQL, Command)
- Always use sql query prepared statements, never bind input string to the sql query.
- Sanitize user input before storing.
##### **Correct code apporach:**
```php
$sth = $dbh->prepare("SELECT * FROM users WHERE email = ? AND password = ?");
$sth->execute([$email, $password]);
```
### A04 - Insecure Design
- Check for rate limiting attempts for login, should be only 5 attempts.
- Tokens should be cryptographically random and at least 128 bits longer.
- Sensitive operations like email change or payment, must require user to login again.
##### **Correct code apporach:**
```php
$token = bin2hex(random_bytes(32));
$expireDate = (new \DateTime())->format('Y-m-d');
$sth = $dbh->prepare("INSERT INTO password_resets (email, token, expirer_date) VALUES (?, ?, ?)");
$sth->execute([$emailLogged, $token, $expireDate]);
```
