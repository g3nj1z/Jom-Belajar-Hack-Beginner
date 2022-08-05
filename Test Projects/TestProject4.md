# OS command injection

## [OS command injection, simple case](https://portswigger.net/web-security/os-command-injection/lab-simple)

This lab contains an OS command injection vulnerability in the product stock checker.

The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.

To solve the lab, execute the whoami command to determine the name of the current user.
