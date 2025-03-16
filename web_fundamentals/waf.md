# WAF (Web Application Firewall)

Before we talk about WAF, we need to know what the Firewall is used to control a network, a server, a service inside a corporate network, or a cloud or a local network.

WAF helps protect web applications by filtering and monitoring HTTP traffic between the web application and the internet. It generally protect the web applications against attacks such as cross-site request forgery, cross-site scripting (XSS), file inclusions, SQL injection, etc.

By deploying a WAF in front of the web application, a shield is places between the application and the internet. While a proxy server protects the identity of the client machine using an intermediary, WAF is a type of reverse proxy that protects the server from exposure, as requests pass through the WAF before reaching the server. 