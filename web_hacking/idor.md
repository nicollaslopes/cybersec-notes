# IDOR (Insecure Direct Object References)

Insecure Direct Object Reference (IDOR) is a vulnerability that arises when attackers can access or modify objects by manipulating identifiers used in a web application's URLs or parameters. It occurs due to missing access control checks, which fail to verify whether a user should be allowed to access specific data.

Consider a website that uses the following URL to access the customer account page, by retrieving information from the back-end database:

```
https://insecure-website.com/customer_account?customer_number=123
```

The 123 in the URL is a direct reference to the user's record in the database, often represented by the primary key. If an attacker changes this number to 124 and gains access to another user's information, the application is vulnerable to Insecure Direct Object Reference. This happens because the app didn't properly check if the user had permission to view data for user 124 before displaying it.