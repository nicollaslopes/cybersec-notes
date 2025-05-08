# Web

Para termos sucesso em uma exploração web, a parte de reconhecimento é de suma importância para que isso aconteça. Essa é praticamente a fase mais importante que pode decidir o sucesso ou não das etapas posteriores. Precisamos então coletar as informações de uma forma que possamos levantar o maior número possível para entendermos bem o ambiente.

## Robots and Sitemap

O google possui vários crawlers (robôs que varrem a internet) que servem para indexar páginas. No entendo, você pode dizer para o google para não indexar alguma página de sua escolha. Para isso, é utilizado o arquivo **robots.txt** para essa configuração, colocando dentro dele os diretórios que não devem ser indexados. Quando o crawler acessar a página, ele irá ler primeiro o arquivo e ver quais diretórios que não vão ser indexados. No entendo, nós conseguimos ler o **robots.txt** e isso acaba se tornando uma brecha se o administrador deixar isso visível. Para isso, basta acessar o site junto com um "/robots.txt".

Já o sitemap, é basicamente um mapa do site que mapeia todas as páginas do site. Isso facilita o google saber quais são as páginas do site para indexar mais rápido. Para ler esse arquivo basta acessar o site junto com um "/sitemap.xml".

## Directory Listing

A listagem de diretórios, como o próprio nome já diz, é quando a aplicação possui um diretório onde está sendo listado os arquivos daquela pasta. Isso pode representar uma falha e ao mesmo tempo não, pois irá depender muito do que for encontrado e da criticidade de importância desses arquivos. Isso ocorre quando determinada pasta não possui um arquivo nela de index, então todos os outros arquivos nela ficam expostos.

imagem 1

## Mirror Website

O mirror website é basicamente baixar/clonar uma página, serve tanto para fazer um conhecimento da estrutura como fazer uma página para engenharia social. Podemos utilizar dessa forma:

```
$ wget -m <site>
```

Pode ser que por causa do robots.txt, não seja possível baixar corretamente os arquivos, para isso basta utilizar

```
$ wget -m -e robots=off <site>
```

