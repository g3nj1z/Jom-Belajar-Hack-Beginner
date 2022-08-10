*REFLECTED XSS*

Reflected XSS into HTML context with nothing encoded

1. Put this script in the search box:

<script>alert(1)</script>

*Graphic solution available in module on Reflected XSS.

*STORED XSS*

Stored XSS into HTML context with nothing encoded

1. Enter the following into the comment box:

<script>alert(1)</script>
2. Enter a name, email and website.
3. Click "Post comment".
4. Go back to the blog.

*Graphic solution available in module on Reflected XSS.

*DOM-BASED XSS*

DOM XSS in document.write sink using source location.search

1. Enter a random alphanumeric string into the search box.
2. Right-click and inspect the element, and observe that your random string has been placed inside an img src attribute.
3. Break out of the img attribute by searching for:

  {"><svg onload=alert(1)>}

DOM XSS in document.write sink using source location.search inside a select element

1. On the product pages, notice that the dangerous JavaScript extracts a storeId parameter from the location.search source. It then uses document.write to create a new option in the select element for the stock checker functionality.
2. Add a storeId query parameter to the URL and enter a random alphanumeric string as its value. Request this modified URL.
3. In the browser, notice that your random string is now listed as one of the options in the drop-down list.
4. Right-click and inspect the drop-down list to confirm that the value of your storeId parameter has been placed inside a select element.
5. Change the URL to include a suitable XSS payload inside the storeId parameter as follows:

  {product?productId=1&storeId="></select><img%20src=1%20onerror=alert(1)>}

DOM XSS in innerHTML sink using source location.search

1. Enter the following into the into the search box:

  {<img src=1 onerror=alert(1)>}
2. Click "Search".

The value of the src attribute is invalid and throws an error. This triggers the onerror event handler, which then calls the alert() function. As a result, the payload is executed whenever the user's browser attempts to load the page containing your malicious post.

DOM XSS in jQuery anchor href attribute sink using location.search source

1. On the Submit feedback page, change the query parameter returnPath to / followed by a random alphanumeric string.
2. Right-click and inspect the element, and observe that your random string has been placed inside an a href attribute.
3. Change returnPath to:

  {javascript:alert(document.cookie)}
Hit enter and click "back".

DOM XSS in jQuery selector sink using a hashchange event

1. Notice the vulnerable code on the home page using Burp or the browser's DevTools.
2. From the lab banner, open the exploit server.
3. In the Body section, add the following malicious iframe:

  {<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>}
4. Store the exploit, then click View exploit to confirm that the print() function is called.
5. Go back to the exploit server and click Deliver to victim to solve the lab.

Reflected DOM XSS

1. In Burp Suite, go to the Proxy tool and make sure that the Intercept feature is switched on.
2. Back in the lab, go to the target website and use the search bar to search for a random test string, such as "XSS".
3. Return to the Proxy tool in Burp Suite and forward the request.
4. On the Intercept tab, notice that the string is reflected in a JSON response called search-results.
5. From the Site Map, open the searchResults.js file and notice that the JSON response is used with an eval() function call.
6. By experimenting with different search strings, you can identify that the JSON response is escaping quotation marks. However, backslash is not being escaped.
7. To solve this lab, enter the following search term:

  \"-alert(1)}//

As you have injected a backslash and the site isn't escaping them, when the JSON response attempts to escape the opening double-quotes character, it adds a second backslash. The resulting double-backslash causes the escaping to be effectively canceled out. This means that the double-quotes are processed unescaped, which closes the string that should contain the search term.

An arithmetic operator (in this case the subtraction operator) is then used to separate the expressions before the alert() function is called. Finally, a closing curly bracket and two forward slashes close the JSON object early and comment out what would have been the rest of the object. As a result, the response is generated as follows:

  {"searchTerm":"\\"-alert(1)}//", "results":[]}

Stored DOM XSS

1. Post a comment containing the following vector:

  {<><img src=1 onerror=alert(1)>}

In an attempt to prevent XSS, the website uses the JavaScript replace() function to encode angle brackets. However, when the first argument is a string, the function only replaces the first occurrence. We exploit this vulnerability by simply including an extra set of angle brackets at the beginning of the comment. These angle brackets will be encoded, but any subsequent angle brackets will be unaffected, enabling us to effectively bypass the filter and inject HTML.
