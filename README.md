# DVWA Security Lab Report

## Student Information
- Name: Fowzi Ali
- Course: Cybersecurity: Theory, Tools
- Assignment: Application Security Testing
- Date: March 8th, 2026

---

# Environment Setup

## Docker Installation

Command used:

```
docker --version
```

Result:
Docker installed successfully.

Screenshot:

![Docker Version](screenshots/docker-version.png)

---

## DVWA Deployment

Command used:

```
docker pull vulnerables/web-dvwa
docker run -d -p 8080:80 --name dvwa vulnerables/web-dvwa
```

Verification:

```
docker ps
```

Result:
The DVWA container is running successfully.

Screenshot:

![Docker PS Output](screenshots/docker-ps.png)

DVWA was accessible at:

http://localhost:8080

Screenshot:

![DVWA Homepage](screenshots/dvwa-homepage.png)

---

# Vulnerability Testing

## Brute Force Authentication

Description:
The brute force vulnerability allows an attacker to attempt multiple username/password combinations until the correct credentials are discovered.

Testing Tool:
Burp Suite Intruder

---

### Security Level: Low

Payload Used:

username: admin  
password list:
123456
admin
letmein
karachi123
password

Steps Performed:

1. Intercept login request using Burp Suite.
2. Send request to Intruder.
3. Configure password field as payload position.
4. Load password wordlist.
5. Start attack.

Result:

Burp Suite successfully identified the correct password **password** for the admin account.

Observation:

The application allowed unlimited login attempts with no rate limiting or CAPTCHA protection.

Screenshot:

![Burp Intruder Attack](screenshots/bruteforce-low-intruder.png)

Explanation (Why it Worked):

The application does not implement any protection mechanisms such as:

- account lockout
- request throttling
- CAPTCHA
- IP blocking

Because of this, automated password guessing attacks succeed easily.

---

### Security Level: Medium

Payload Used:

Same password list as above.

Result:

The attack still works but is slower because the application introduces a delay between login attempts.

Screenshot:

![Medium Brute Force](screenshots/bruteforce-medium.png)

Explanation (Why It Is Harder):

Medium security introduces basic protections such as:

- request validation
- delay between login attempts

This slows down automated brute force attempts but does not fully prevent them.

---

### Security Level: High

Payload Used:

Password wordlist:

123456
admin
letmein
karachi123
password

Steps Performed:

1. Open **Burp Suite** and enable Proxy Intercept.
2. Submit a login request in DVWA to capture the request.
3. Send the request to **Burp Intruder**.
4. Identify the **password parameter** as the payload position.
5. Extract the **user_token value** from the request.
6. Configure Intruder to update the CSRF token dynamically for each request.
7. Launch the brute force attack using the password wordlist.

Result:

Burp Suite successfully identified the correct password **password** even at High security level.

Screenshot:

![High Brute Force](screenshots/bruteforce-high.png)

Explanation (Why it Worked):

At High security level, DVWA introduces additional protections including:

- CSRF tokens
- request validation

Each login attempt requires a valid `user_token` parameter. However, by intercepting the request using Burp Suite and dynamically updating the CSRF token during the attack, the protection can be bypassed. This allows automated brute force attempts to continue until the correct password is discovered.

---

### Security Comparison

| Security Level | Attack Success | Reason |
|----------------|---------------|-------|
| Low | Successful | No protection |
| Medium | Successful | Basic validation and delay |
| High | Successful | CSRF token bypassed using Burp Suite |

---

### OWASP Top 10 Mapping

Category:  
Broken Authentication (OWASP A2)

## Command Injection

Description:
Command Injection occurs when an application passes unsanitized user input to the operating system shell. An attacker can inject additional commands to execute arbitrary system operations on the server. This vulnerability is commonly found in poorly validated system utilities such as ping or network tools.

---

### Security Level: Low

Payload Used:

```
127.0.0.1; ls
```

Other payloads:

```
127.0.0.1 && ls
127.0.0.1 | ls
127.0.0.1; whoami
```

Steps Performed:

1. Navigate to **DVWA → Command Injection**.
2. Set DVWA Security Level to **Low**.
3. Enter the payload in the **IP address field**.
4. Click **Submit**.

Result:

The application executed both the **ping command** and the injected command. The server returned the directory listing or system information.

Screenshot:

![Command Injection Low](screenshots/command-injection-low.png)

Explanation (Why it Worked):

At Low security level, DVWA directly inserts the user input into a system command:

```
ping -c 4 <user_input>
```

Since no input validation is applied, command separators like `;` allow attackers to execute additional commands.

---

### Security Level: Medium

Payload Used:

```
127.0.0.1 && ls
```

or

```
127.0.0.1 | ls
```

Result:

Some command separators may still work, allowing partial command execution.

Screenshot:

![Command Injection Medium](screenshots/command-injection-medium.png)

Explanation (Why It Is Harder):

Medium security introduces basic input filtering to block dangerous characters like `;`. However, other operators such as `&&` or `|` may still bypass the filter.

---

### Security Level: High

Payload Used:

```
127.0.0.1; ls
```

Result:

The attack fails and only the ping command executes.

Screenshot:

![Command Injection High](screenshots/command-injection-high.png)

Explanation (Why It Failed):

High security performs strict input validation and sanitization, removing dangerous command separators and preventing arbitrary command execution.

---

### Security Comparison

| Security Level | Attack Success | Reason |
|----------------|---------------|-------|
| Low | Successful | No input validation |
| Medium | Successful | Basic filtering |
| High | Failed | Strong input sanitization |

---

### OWASP Top 10 Mapping

Category:

Injection (OWASP A03)

### More PHP Payloads:
https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#python

## Cross-Site Request Forgery (CSRF)

Description:
Cross-Site Request Forgery (CSRF) is a vulnerability that allows an attacker to trick a logged-in user into performing unintended actions on a web application. The attacker crafts a malicious request that is automatically executed by the victim's browser because the user is already authenticated.

---

### Security Level: Low

Payload Used:

```
http://localhost:8080/vulnerabilities/csrf/?password_new=hacked123&password_conf=hacked123&Change=Change
```

Steps Performed:

1. Navigate to **DVWA → CSRF**.
2. Set DVWA Security Level to **Low**.
3. Capture the password change request URL.
4. Modify the URL with a new password value.
5. Open the malicious URL while logged in.

Result:

The password for the logged-in user was successfully changed without any confirmation.

Screenshot:

![CSRF Low](screenshots/csrf-low.png)

Explanation (Why it Worked):

At Low security level, the application does not verify whether the request originated from the legitimate website. Since the user is already authenticated, the browser automatically includes the session cookie, allowing the request to execute successfully.

---

### Security Level: Medium

Payload Used:

```
http://localhost:8080/vulnerabilities/csrf/?password_new=hacked123&password_conf=hacked123&Change=Change
```

Result:

The attack becomes harder because the application checks the HTTP referrer header.

Screenshot:

![CSRF Medium](screenshots/csrf-medium.png)
![CSRF Medium](screenshots/csrf-medium1.png)
![CSRF Medium](screenshots/csrf-medium2.png)

Explanation (Why It Is Harder):

Medium security introduces a basic validation check that verifies whether the request originated from the same application using the HTTP referrer header. However, referrer headers can sometimes be bypassed or manipulated.

---

### Security Level: High

Payload Used:

Request captured using Burp Suite containing a valid CSRF token.

Example request:

http://localhost:8080/vulnerabilities/csrf/?password_new=hacked123&password_conf=hacked123&user_token=<VALID_TOKEN>&Change=Change

Steps Performed:

1. Navigate to **DVWA → CSRF**.
2. Set DVWA Security Level to **High**.
3. Open **Burp Suite** and enable the **Proxy Intercept** feature.
4. Change the password in DVWA to capture the request.
5. Send the captured request to **Repeater**.
6. Extract the **user_token value** from the request.
7. Reuse the token inside a crafted malicious request.

Result:

The password was successfully changed after replaying the request with a valid CSRF token.

Screenshot:

![CSRF High](screenshots/csrf-high.png)
![CSRF High](screenshots/csrf-high1.png)
![CSRF High](screenshots/csrf-high2.png)

Explanation (Why it Worked):

At High security level, DVWA introduces a CSRF token (`user_token`) to protect the request.

However, the token can still be captured from a legitimate request using **Burp Suite** and reused in another request before it expires.

Since the server only checks whether the token exists and is valid, replaying the request with a captured token allows the attacker to bypass the CSRF protection.

---

### Security Comparison

| Security Level | Attack Success | Reason |
|----------------|---------------|-------|
| Low | Successful | No request validation |
| Medium | Successful | Referrer header check can be bypassed |
| High | Successful | CSRF token captured and replayed using Burp Suite |

---

### OWASP Top 10 Mapping

Category:

Broken Access Control (OWASP A01)
