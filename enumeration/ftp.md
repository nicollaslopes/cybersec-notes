# FTP

To interact with FTP service, we can use netcat to connect or also use the FTP utility (we don't need to pass the port if it is the default port 21).

```
$ nc -vn <host> 21
```

```
$ ftp <host>
```

It will then ask you for your username and password.

We can enumerate trying to log in with `anonymous` or `ftp` users. Some administrators configure it to accept to loggin with these users, which have low privileges, it's not a good practice to use it. If these users are allowed, we can try to log in using `anonymous` or `ftp` as the password.

If we manage to connect successfully and execute a command, for example "ls", it will not respond, because there is a firewall blocking, the default FTP configuration works actively, so we first need to enter passive mode to interact with this service. 

We can use `help` command to check the options. To enter passive mode, we can use `passive` or `pasv` command. Now we can execute the commands without any problems.
