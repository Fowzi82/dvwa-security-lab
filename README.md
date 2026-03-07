# DVWA Security Lab Report

## Student Information
Name:
Course:
Assignment:
Date:

---

# Environment Setup

## Docker Installation

Command used:

```
docker --version
```

Result:
Docker installed successfully.

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

DVWA was accessible at:

http://localhost:8080

---

# Vulnerability Testing

## SQL Injection

### Security Level: Low
