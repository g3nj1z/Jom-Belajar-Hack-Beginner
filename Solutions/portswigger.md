# OS Command Injection

## OS command injection, simple case
1. Use Burp Suite to intercept and modify a request that checks the stock level.
2. Modify the storeID parameter, giving it the value 1|whoami.
3. Observe that the response contains the name of the current user.

## Blind OS command injection with time delays
1. Use Burp Suite to intercept and modify the request that submits feedback.
2. Modify the email parameter, changing it to:
    `email=x||ping+-c+10+127.0.0.1||`
3. Observe that the response takes 10 seconds to return.

## Blind OS command injection with output redirection
1. Use Burp Suite to intercept and modify the request that submits feedback.
2. Modify the email parameter, changing it to:
    `email=||whoami>/var/www/images/output.txt||`
3. Now use Burp Suite to intercept and modify the request that loads an image of a product.
4. Modify the filename parameter, changing the value to the name of the file you specified for the output of the injected command:
    `filename=output.txt`
5. Observe that the response contains the output from the injected command.




