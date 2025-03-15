# Cookies

Cookies are small files of information that a web server generates and sends to a web browser. Web browsers store the cookies they receive for a predetermined period of time, or for the length of a user's session on a website. They attach the relevant cookies to any future requests the user makes of the web server.

## Types of cookies

* Session cookies
* Persistent cookies
* Third-party Cookies

## Properties

#### 1. **name**

- **Description**: The name is the cookie key, that is, the unique identification that will be used to access the cookie value.
- **Example**: `name=user`

#### 2. **value**

- **Description**: The cookie value is the data that the website wants to store. It can contain any information, such as login data or language preferences.
- **Example**: `value=john_doe`

#### 3. **path**

- **Description**: The `path` property determines the URL scope in which the cookie is valid. That is, it will only be sent in requests for URLs that match the specified path.
- **Example**: If the path is `/app`, the cookie will only be sent in requests that begin with `/app`.

#### 4. **domain**

- **Description**: The `domain` property specifies the domain for which the cookie will be valid. If not specified, the cookie will only be sent to the domain of the website that created it. It can be set to a higher domain, allowing the cookie to be accessed by subdomains.
- **Example**: If `domain=example.com`, the cookie will be valid for both `www.example.com` and `blog.example.com`.

#### 5. **expires**

- **Description**: The `expires` property sets the date and time at which the cookie expires. If the cookie does not have the `expires` property, it is considered a session cookie, which is deleted when the browser is closed. Otherwise, the cookie persists until the specified date.
- **Example**: `expires=Sat, 01 Jan 2026 12:00:00 GMT`

#### 6. **http-only**

- **Description**: When the `HttpOnly` property is set, the cookie cannot be accessed via client-side JavaScript. This increases security by making it more difficult for the cookie to be stolen by malicious scripts, such as those that can be used in Cross-Site Scripting (XSS) attacks.
- **Example**: If `HttpOnly` is set, the cookie will not be accessible via `document.cookie` in JavaScript.

#### 7. **secure**

- **Description**: The `secure` property indicates that the cookie will only be sent over secure connections (HTTPS). This ensures that the cookie will not be exposed to man-in-the-middle attacks over unsecured networks.
- **Example**: When `secure` is set, the cookie will only be sent when the communication between the browser and the server is encrypted.

#### 8. **samesite**

- **Description**: The `SameSite` property controls when the cookie is sent in requests from other sites, helping to protect against Cross-Site Request Forgery (CSRF) attacks. There are three possible values:
    - `Strict`: The cookie is only sent to the same origin site.
    - `Lax`: The cookie is sent in top-level navigation requests, such as when clicking on a link, but not in third-party requests such as images or iframes.
    - `None`: The cookie is sent in any request, including from other sites. When `None` is used, the `Secure` property must also be set.


We can create a session cookie using PHP. With this, the cookie will not be persistent, that is, as soon as the browser is closed, the cookie will disappear.

```php
<?php

setcookie('mycookie', 'somevalue');
```

To create a persistent cookie, we can pass the expiration time as a parameter. This way, even if you close the browser, the cookie will exist. It will only not exist if you access it through the incognito tab.

```php
<?php

setcookie('mypersistentcookie', 'somevalue', time() + 60);
```
