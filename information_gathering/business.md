# Business

Some sites we can look for important information are:

* **Linkedin**: Both to search for information about employees and detailed vacancies about the technologies that this company uses (also collect information from the developers' github)
* **Hunter.io**: Site to collect email addresses (you need to create an account)
* **Have I been pwned**: Site to check if any email is in the leaked email database.
* **Pastebin**: Site to upload content.

## Information Gathering methods

### Searching website caches

Another very essential tool is Web Archive. This site basically maps the cache of a domain's pages since its existence. We can use it to search for files that existed a while ago and gather important information.

{% embed url="https://web.archive.org" %}

### Google Hacking

Google hacking is basically using the google search engine to collect sensitive information. Some operators are:

* **site**: search on the specified site
* **inurl**: search on the url
* **intitle**: search on the title
* **intext**: search on the text
* **filetype**: search by file type
* **ext**: search by file extension
* **cache**: search in the cache
* **" "** : exact search
* **-** : (just a dash) Omit in the search 

Type of files and extensions: 

* php, asp, aspx, do, js, phps, txt, doc, docx, pdf, xls, xlsx, ppt, ovpn, sql, bak, old...

Titles
* "index of", "login", "restricted access", "admin", "adm"...

Keywords
* "config", "password", "passwords", "users", "access", "ftp", "bkp", "backup", "data"...  

Examples:
* site:businesscorp.com.br -www ext:php
* site:businesscorp.com.br intitle:"Admin"
* site:businesscorp.com.br "index of" backup

A well-known site that we can use to consult dorks is GHDB.

{% embed url="https://www.exploit-db.com/google-hacking-database" %}

### NDN - Non-Delivery-Notification

NDN (non-delivery notification) is when the server is configured to reply to an e-mail that has been sent to it but does not exist. This way, the server can reply with some important information, for example, subdomain, versions of the software used, local network IP address, among others.

### Metadata 

When we manage to collect a certain file from the target, we can use a tool called `exiftool` to analyze the metadata of that file. The metadata may contain some interesting information that we can analyze. After downloading the document, we can use it as follows:

```bash
$ exiftool <file>
```