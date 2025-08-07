# CRLF Injection

A [CRLF injection](https://www.invicti.com/learn/crlf-injection/) attack is one of several types of injection attacks. It can be used to escalate to more malicious attacks such as Cross-site Scripting (XSS), page injection, web cache poisoning, cache-based defacement, and more. A CRLF injection vulnerability exists if an attacker can inject the CRLF characters into a web application, for example using a user input form or an HTTP request.

The CRLF abbreviation refers to `Carriage Return` and `Line Feed`. CR and LF are special characters (ASCII 13 and 10 respectively, also referred to as \r\n) that are used to signify the _End of Line_ (EOL). The CRLF sequence is used in operating systems including Windows (but not Linux/UNIX) and Internet protocols including HTTP.

There are two most common uses of CRLF injection attacks: log poisoning and HTTP response splitting. In the first case, the attacker falsifies log file entries by inserting an end of a line and an extra line. This can be used to hide other attacks or to confuse system administrators. In the second case, CRLF injection is used to add HTTP headers to the HTTP response and, for example, perform an XSS attack that leads to information disclosure. A similar technique, called [Email Header Injection](https://www.acunetix.com/blog/articles/email-header-injection/), may be used to add SMTP headers to emails.

## What Is HTTP Response Splitting

The HTTP protocol uses the CRLF character sequence to signify where one header ends and another begins. It also uses it to signify where headers end and the website content begins.

If the attacker inserts a single CRLF, they can add a new header. If it is, for example, a `Location` header, the attacker can redirect the user to a different website. Criminals may use this technique for phishing or defacing. This technique is often called HTTP header injection.

If the attacker inserts a double CRLF, they can prematurely terminate HTTP headers and inject content before the actual website content. The injected content can contain JavaScript code. It can also be formulated so that the actual website content coming from the web server is ignored by the web browser. This is how HTTP response splitting is used in combination with Coss-site Scripting (XSS).

An example:

```http
htttps://www.example.com/somepage.php?page=%0d%0aSet-Cookie:%20malicious-cookie
```

We can notice that `%0d%0a` is `\r\n`. With this, we were able to use a line break and add a new header `Set-Cookie` with value of `malicious-cookie`.

_(Many web frameworks nowadays prevent HTTP response splitting through CRLF injection by not allowing CRLF sequences to be included in HTTP headers.)_
