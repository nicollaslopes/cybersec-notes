# DNS

DNS (Domain Name System) is the registry that transforms letters into IP addresses, so we have a name, a DNS address and it is transformed into an IP address.

Before DNS existed, when we needed to access a resource such as a website on the internet, we would have to know the IP address. Imagine how difficult it would be to remember hundreds of IP address numbers every time we accessed a website.

If DNS didn't exist, to access Facebook you would have to remember the address 157.240.222.35, then if you wanted to use Google you would have to know the address 172.217.30.78. The Domain Name System or DNS emerged precisely to solve this problem, basically it translates the name into an IP address.

## How does a domain work?

When we access a website, to know which IP address the entry represents, it is done in a few steps. Our computer communicates with the root servers, and these root servers have a list of top-level domains. DNS resolution starts backwards, first the country of origin is consulted (which TLD is responsible for that domain), then the authoritative server is consulted, and then the subdomain is consulted.

#### The 8 steps in a DNS lookup:

<figure><img src="../.gitbook/assets/dns-1 (1).png" alt=""><figcaption></figcaption></figure>

1. A user types ‘example.com’ into a web browser and the query travels into the Internet and is received by a DNS recursive resolver.
2. The resolver then queries a DNS root nameserver (.).
3. The root server then responds to the resolver with the address of a Top Level Domain (TLD) DNS server (such as .com or .net), which stores the information for its domains. When searching for example.com, our request is pointed toward the .com TLD.
4. The resolver then makes a request to the .com TLD.
5. The TLD server then responds with the IP address of the domain’s nameserver, example.com.
6. Lastly, the recursive resolver sends a query to the domain’s nameserver.
7. The IP address for example.com is then returned to the resolver from the nameserver.
8. The DNS resolver then responds to the web browser with the IP address of the domain requested initially.

## What are DNS records?

DNS records (also known as zone files) are instructions that reside on authoritative DNS servers and provide information about a domain, including which IP addresses are associated with that domain and how to handle requests for that domain. These records consist of a series of text files written in what is known as DNS syntax. DNS syntax is simply a string of characters used as commands that tell the DNS server what to do.

#### What are the most common types of DNS records?

**A** — This is the record that contains the IP address of a domain. **AAAA** — This is a record that contains the IPv6 address for a domain (as opposed to A records, which list the IPv4 address). **CNAME** — This points a domain or subdomain to another domain; it does NOT provide an IP address. **MX** — This directs email to a mail server. **TXT** — This allows an administrator to store text notes in the record. These records are often used for email security. **NS** — This stores the nameserver for a DNS entry. **SOA** — This stores administrator information about a domain.
