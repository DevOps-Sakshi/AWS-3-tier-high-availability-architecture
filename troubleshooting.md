# Troubleshooting

This document records the major issues encountered while deploying and
validating the AWS 3-tier application and the steps taken to resolve them.

## 1. Private Application Tier Access

### Problem
The Application tier was deployed in private subnets without public IP
addresses, so direct SSH access was not available.

### Investigation
Instead of exposing the private instances to the internet, AWS Systems
Manager (SSM) Session Manager was evaluated for secure instance access.

### Resolution
An IAM role was created for the EC2 instances and the
`AmazonSSMManagedInstanceCore` policy was attached.

The application instances could then be accessed through SSM Session
Manager without exposing SSH port 22.

### Result
Private application instances remained private while still allowing
secure administrative access.

---

## 2. RDS MySQL 8.4 Application Connectivity

### Problem
The Node.js application was unable to connect correctly to the RDS MySQL
database.

### Investigation
Network connectivity from the Application tier to RDS on port `3306` was
verified successfully. The RDS instance was running MySQL `8.4.9`.

The issue was traced to the Node.js MySQL client being incompatible with
the authentication used by the MySQL 8.4 environment.

### Resolution
The Node.js database client was changed from `mysql` to `mysql2`, and the
application was restarted using PM2.

### Result
The application successfully connected to RDS and was able to retrieve
database records.

---

## 3. Nginx HTTP/HTTPS Redirect

### Problem
The application was initially redirecting HTTP requests to HTTPS.

### Investigation
The Nginx configuration contained a redirect based on the
`X-Forwarded-Proto` header:

```nginx
if ($http_x_forwarded_proto = 'http') {
    return 301 https://$host$request_uri;
}