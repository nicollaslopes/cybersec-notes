# CSRF - Cross-site Request Forgery


Essa vulnerabilidade ocorre quando é possível fazer uma requisição de um site para outro site.

Cross-site request forgery (also known as CSRF) is a web security vulnerability that allows an attacker to induce users to perform actions that they do not intend to perform. It allows an attacker to partly circumvent the same origin policy, which is designed to prevent different websites from interfering with each other.

For a CSRF attack to be possible, three key conditions must be in place:

- **A relevant action.** There is an action within the application that the attacker has a reason to induce. This might be a privileged action (such as modifying permissions for other users) or any action on user-specific data (such as changing the user's own password).
- **Cookie-based session handling.** Performing the action involves issuing one or more HTTP requests, and the application relies solely on session cookies to identify the user who has made the requests. There is no other mechanism in place for tracking sessions or validating user requests.
- **No unpredictable request parameters.** The requests that perform the action do not contain any parameters whose values the attacker cannot determine or guess. For example, when causing a user to change their password, the function is not vulnerable if an attacker needs to know the value of the existing password.

With these conditions in place, the attacker can construct a web page containing the following HTML:

```html
<html>
<body>
    <form action="http://site-target.com/change/password" method="POST">
        <input type="hidden" name="password" value="newpassw0rd">
        <input type="hidden" name="email" value="admin@admin.com">
    </form>

    <script>
        document.forms[0].submit();
    </script>
</body>
</html>
```

If a victim user visits the attacker's web page, the following will happen:

- The attacker's page will trigger an HTTP request to the vulnerable website.
- If the user is logged in to the vulnerable website, their browser will automatically include their session cookie in the request (assuming [SameSite cookies](https://portswigger.net/web-security/csrf#common-defences-against-csrf) are not being used).
- The vulnerable website will process the request in the normal way, treat it as having been made by the victim user, and change their email address.