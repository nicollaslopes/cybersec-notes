# IANA

**IANA** (Internet Assigned Numbers Authority) é uma organização responsável pela **coordenação global de identificadores únicos da Internet**. Ela não vende domínios, mas garante que os sistemas de nomes, endereços IP e protocolos da Internet sejam únicos e funcionem corretamente no mundo todo.
#### Funções principais da IANA:

* **Gerenciamento de domínios de topo (TLDs)** como `.com`, `.org`, `.br`, etc.
* **Atribuição de blocos de endereços IP** a entidades regionais (como LACNIC na América Latina).
* **Gestão de protocolos e portas padrão**, como as usadas por HTTP, FTP, etc.

A IANA é operada pela **ICANN** (Internet Corporation for Assigned Names and Numbers), uma entidade sem fins lucrativos.

## **RIR (Regional Internet Registry)**

Um **RIR** é um **Registro Regional da Internet**. Ele é responsável por:

- **Distribuir e gerenciar blocos de endereços IP (IPv4 e IPv6)** e **números de sistemas autônomos (ASN)** em uma **região geográfica grande**.
- **Avaliar pedidos técnicos e justificar o uso de IPs**.
- Manter **bases de dados públicas** (como o WHOIS) com as alocações feitas.

### Os 5 RIRs no mundo:

|RIR|Região atendida|Sede|
|---|---|---|
|**LACNIC**|América Latina e Caribe|Montevidéu, Uruguai|
|**ARIN**|EUA, Canadá, partes do Caribe|Virginia, EUA|
|**RIPE NCC**|Europa, Oriente Médio, Ásia Central|Amsterdã, Holanda|
|**APNIC**|Ásia-Pacífico|Brisbane, Austrália|
|**AFRINIC**|África|Ebene, Maurício|

RIRs recebem blocos de IP da **IANA** e os repassam para ISPs, empresas e NIRs.

## **NIR (National Internet Registry)**?

Um **NIR** é um **Registro Nacional da Internet**. Ele atua como uma **delegação do RIR dentro de um país específico**, com as seguintes funções:

- Atende diretamente os ISPs e organizações **dentro do país**.
- Gerencia alocações de **endereços IP** e **ASNs** nacionais.
- Avalia justificativas técnicas e mantém os registros.

Nem todo país tem um NIR — ele **só existe quando o RIR delega essa função a uma organização local**.

### Exemplo:

 No **Brasil**, o NIR é o **NIC.br**, que:
 * Administra os domínios `.br` (através do Registro.br).
 * Distribui blocos de IP e ASNs no país.
 * Opera o sistema de DNS raiz `.br`, NTP.br, etc.


### Estrutura de atribuição de endereços IP:

```
IANA
└── RIRs (ex: LACNIC, ARIN, RIPE, APNIC, AFRINIC)
    └── NIRs (ex: NIC.br no Brasil)
        └── ISPs e organizações locais
```