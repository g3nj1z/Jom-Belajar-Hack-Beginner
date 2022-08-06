# OS Command Injection

## OS command injection, simple case
1. Use Burp Suite to intercept and modify a request that checks the stock level.
2. Modify the storeID parameter, giving it the value 1|whoami.
3. Observe that the response contains the name of the current user.

## Blind OS command injection with time delays
1. Use Burp Suite to intercept and modify the request that submits feedback.
2. Modify the email parameter, changing it to:
    email=x||ping+-c+10+127.0.0.1||
3. Observe that the response takes 10 seconds to return.



