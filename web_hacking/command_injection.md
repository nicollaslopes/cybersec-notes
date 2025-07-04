# Command Injection

O command injection é uma vulnerabilidade web que permite que seja executado um comando do sistema operacional no servidor que está rodando a aplicação. Vamos supor que exista uma parte da aplicação que executa um **whois** em determinado site. Se não existir um filtro adequado do lado do backend, isso pode facilmente ser explorado.