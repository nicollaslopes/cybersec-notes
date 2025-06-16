# Path Traversal

Essa vulnerabilidade ocorre quando a aplicação que te permite ler arquivos no servidor. Geralmente quando há um parâmetro que te permite acessar algum arquivo que está salvo no servidor, como imagem, por exemplo, se não estiver tratado corretamente pode resultar nessa vulnerabilidade. Se caso isso acontecer, você pode ir navegando pelas pastas no servidor, utilizando "../../../../" para voltar as pastas anteriores e acessando os arquivos.

*References:* 

{% embed url="https://portswigger.net/web-security/file-path-traversal" %}
