# Server Side Template Injection

Server-side template injection (SSTI) happens when an attacker can insert malicious code using the template engine’s native syntax, which then gets executed on the server.

Template engines are meant to create web pages by merging static templates with dynamic data. SSTI occurs when user input is directly embedded into a template instead of being treated as data. This gives attackers the ability to inject arbitrary template commands, potentially letting them manipulate the engine and even gain full control of the server. Unlike client-side template injection, SSTI executes on the server, making it significantly more dangerous.

## Twig