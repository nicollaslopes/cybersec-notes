# Bypass Proxies And Firewalls

## SSH Remote Forwarding

Let's suppose we have a localhost application which, as its name implies, it's local, meaning it can only be accessed by the same machine. Therefore, we can't expose this application from a port to the internet. If we want this machine to be accessible externally, we need a valid public IP with open ports and no firewall blocking. To do this, we're going to use an ssh feature called `ssh remote forwarding`.

Inside this machine, I'm going to edit `/etc/ssh/sshd_config` file and enable these options `AllowTcpForwarding` and `GatewayPorts` to `yes` and change the port 22 to 80 (this is optional). Now I save and I need to run the command `systemctl restart sshd` to restart the daemon and load the new settings.

If I try the same ssh command to connect to the server, it will fail because ssh works by default on port 22. We need to specify the port using the `-p` flag. In the terminal of the company's local server (where the firewall is located), we can run this following command:

```bash
$ ssh -R 7000:127.0.0.1:3000 -N -f root@137.184.138.74 -p 80
```

Basically every time someone tries to connect to my server address on Digital Ocean on port `7000`, it will forward, that is, redirect the traffic to my local machine `127.0.0.1` on port `3000`, which is where the local application running.

Now we can access the url:

```
http://137.184.138.74:7000
```

## SSH Dynamic Port Forwarding

Let's suppose that we are working in a company and the proxy blocks the traffic to some sites and we want to bypass it, how can we avoid being restricted?

With the same Digital Ocean server that I have, I can setting that sshd daemon not for port `80` but for a port like `50000`. A high port that probably won't be blocked by firewall. The remote forwarding I did, with ssh -R option allows someone from outside to access a website of mine running on a private machine inside a network. But the opposite is also possible. I can run the following command:

```bash
$ ssh -D 1337 -q -C -N root@137.184.138.74 -p 50000
```

Options like `-q` is quiet mode, so it doesn't print anything on terminal, `-C` is for compressing the content `-N` is for not opening a command line to the server. But the most important is `-D 1337`. This will bind to port `1337` on my local machine. It could by any other port number. Now we connect to sshd pointing to port 50000 on the server, which will probably bypass the firewall.

On my operation system or directly on browser, I look for proxy configuration, but instead of setting the company's proxy address, I set localhost and port `1337`. I can now browse any website I want, even if the company thinks it's blocking me. What I did was create a tunnel via SSH, that is exported as a SOCKS5 service, which is a protocol of a web proxy, listening on port `1337` on my local machine. I made a personal proxy, who receives HTTP packets normally, like a web server, except that it forwards, redirect the traffic, to my digital Ocean server via encrypted tunnel via SSH.

imagem

O serviço sshd de lá recebe os pacotes que vieram pelo túnel e navega usando a internet da Digital Ocean, que tá toda aberta. O site que eu quis navegar devolve a resposta HTTP pro servidor, e o ssd redireciona o pacote de volta pra mim pelo mesmo túnel. E assim eu burlo toda tentativa de me restringir de navegar. Posso navegar onde quiser. Mesmo a saída da porta 80 na empresa estando fechada por firewall, mesmo sendo instruído a usar o servidor de proxy da empresa, eu criei o meu próprio servidor de proxy saindo por uma porta alta que provavelmente tá aberta no firewall e passei a navegar sem restrição nenhuma. Isso é um exemplo do que se chama de tunelamento.

https://builtin.com/software-engineering-perspectives/ssh-port-forwarding\
https://iximiuz.com/en/posts/ssh-tunnels/
