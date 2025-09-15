# SSRF (Server-side Request Forgery)

Server Side Request Forgery (SSRF) occurs when an attacker can create requests from the vulnerable server to the internet. Typically, the vulnerable server has a functionality that reads data from a URL, publishes data to a URL, or imports data from a URL. An attacker could abuse this functionality to read or update internal resources, or bypass access controls like firewalls that prevent the attackers from accessing them directly.

