# HTTPS

O netcat não suporta o SSL para fazer a conexão, com isso podemos utilizar o **openssl**. Temos que informar o método que queremos e o host. Um detalhe, é que nesse caso é importante utilizar o HTTP/1.1 para identificarmos Web Application Firewall ou Proxys, que serão serviços que estão no meio do caminho. O host também é necessário informar, para que o servidor possa nos responder corretamente.

Netcat does not support SSL to establish the connection, so we can utilize `openssl`. We have to inform the method and the host. In this case it's important to use HTTP/1.1 to identify the Web Application Firewall or Proxys, which are services 

```
$ openssl s_client -quiet -connect www.tibiabr.com:443
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R4
verify return:1
depth=1 C = US, O = Google Trust Services, CN = WE1
verify return:1
depth=0 CN = tibiabr.com
verify return:1
HEAD / HTTP/1.1
Host: www.tibiabr.com

HTTP/1.1 200 OK
Date: Tue, 27 May 2025 00:11:08 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Server: cloudflare
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Link: <https://www.tibiabr.com/wp-json/>; rel="https://api.w.org/"
Link: <https://www.tibiabr.com/wp-json/wp/v2/pages/3199>; rel="alternate"; title="JSON"; type="application/json"
Link: <https://www.tibiabr.com?p=3199>; rel=shortlink
X-Tec-Api-Version: v1
X-Tec-Api-Root: https://www.tibiabr.com/wp-json/tribe/events/v1/
X-Tec-Api-Origin: https://www.tibiabr.com
Nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
Cf-Cache-Status: DYNAMIC
Report-To: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=7K5ofxGneOD2GYYa8QOpemf%2FNlG9ABrVKWS1tv304TYBVVD3WUj0eRO5ECu233nM37TEuF5DUtWbbfcNPkPuHt8GK1Rt1rV3WBqH9JtkIrAVmHVLffCQ5tM%3D"}]}
Set-Cookie: PHPSESSID=j6cq0q9bg12tes55m23nuuk84l; Path=/
CF-RAY: 94614af6fe456509-GIG
alt-svc: h3=":443"; ma=86400
```

Quando usamos o HTTP/1.0 e não passamos um host, o servidor não retorna um resultado como esperamos, por isso é necessário utilizar o HTTP/1.1 e o host principalmente se ele estiver passando por um WAF ou proxy. Com isso acaba que lidamos diretamente com o web application firewall ou proxy que a aplicação está utilizando, e não com ela diretamente.

```
$ openssl s_client -quiet -connect www.serasa.com.br:443
depth=2 C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA
verify return:1
depth=1 C = BE, O = GlobalSign nv-sa, CN = GlobalSign Organization Validation CA - SHA256 - G2
verify return:1
depth=0 C = BR, ST = SAO PAULO, L = SAO PAULO, OU = Serasa S.A., O = Serasa S.A., CN = *.serasaexperian.com.br
verify return:1
HEAD / HTTP/1.0

HTTP/1.1 503 Service Unavailable
Content-Type: text/html
Cache-Control: no-cache, no-store
Connection: close
Content-Length: 692
X-Iinfo: 3-131528363-0 0NNN RT(1625925716861 8316) q(3 -1 -1 -1) r(3 -1)

<html style="height:100%"><head><META NAME="ROBOTS" CONTENT="NOINDEX, NOFOLLOW"><meta name="format-detection" content="telephone=no"><meta name="viewport" content="initial-scale=1.0"><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"></head><body style="margin:0px;height:100%"><iframe id="main-iframe" src="/_Incapsula_Resource?CWUDNSAI=27&xinfo=3-131528363-0%200NNN%20RT%281625925716861%208316%29%20q%283%20-1%20-1%20-1%29%20r%283%20-1%29&incident_id=0-483007950490240963&edet=9&cinfo=ffffffff&rpinfo=0&mth=HEAD" frameborder=0 width="100%" height="100%" marginheight="0px" marginwidth="0px">Request unsuccessful. Incapsula incident ID: 0-483007950490240963</iframe></body></html>
```