# Pointers

Precisamos entender que a a variável está associada a um valor, que tem um tipo e está em um endereço. (o 4 em %4zu e 5 em %5d só está especificando a quantidade de coluna para ser impresso no terminal)

```c
#include <stdio.h>

int main(void) {

    int a = 17;
    int b = 25;

    puts("Var   Address         Size Value");

    printf("a -> %p %4zu %5d\n", &a, sizeof(a), a);
    printf("b -> %p %4zu %5d\n", &b, sizeof(b), b);

    return 0;
}
```


```
$ ./main            
Var   Address         Size  Value
a -> 0x7ffd79725750    4     17
b -> 0x7ffd79725754    4     25
```

## O que é um ponteiro?

Um ponteiro é simplesmente uma variavel especializada em receber e manipular endereços de memória. 

