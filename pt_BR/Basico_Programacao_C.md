# Básico da programação em C

## Índice
- Introdução, algoritmos, binário e hexadecimal.
- Ambientes de Desenvolvimento.
- Estrutura básica de um programa em C.
- Operadores e Tipagem de Dados.
- Variáveis, Vetores,Matrizes e constantes.
- Condicionais e Repetições.
- Funções e Parâmetros. 
- Ponteiros, Estruturas.
- Filas, Pilhas.
- Manipulação de arquivos.

## Introdução, algoritmos, binário e hexadecimal

### A máquina de turing
A Máquina de Turing é um dispositivo teórico conhecido como máquina universal, que foi concebido pelo matemático britânico Alan Turing.

Num sentido preciso, é um modelo abstrato de um computador, que se restringe apenas aos aspectos lógicos do seu funcionamento (memória, estados e transições), e não a sua implementação física.

Uma máquina de Turing consiste em:

Uma fita que é dividida em células, uma adjacente à outra. Cada célula contém um símbolo de algum alfabeto finito. O alfabeto contém um símbolo especial branco (aqui escrito como ¬) e um ou mais símbolos adicionais. Assume-se que a fita é arbitrariamente extensível para a esquerda e para a direita, isto é, a máquina de Turing possui tanta fita quanto é necessário para a computação. Assume-se também que células que ainda não foram escritas estão preenchidas com o símbolo branco.

Um cabeçote, que pode ler e escrever símbolos na fita e mover-se para a esquerda ou para a direita.

Um registrador de estados, que armazena o estado da máquina de Turing. O número de estados diferentes é sempre finito e há um estado especial denominado estado inicial com o qual o registrador de estado é inicializado.

Uma tabela de ação (ou função de transição) que diz à máquina que símbolo escrever, como mover o cabeçote ( ← para esquerda e →  para direita) e qual será seu novo estado, dados o símbolo que ele acabou de ler na fita e o estado em que se encontra. Se não houver entrada alguma na tabela para a combinação atual de símbolo e estado então a máquina para.

### E os nossos computadores?
Entre as várias arquiteturas presentes esse curso irá focar na arquitetura x86/x86-64.

Podemos ver em uma desmontagem de um código binário que um programa são uma série de instruções assim como a fita da maquina de Turing, e assim como a máquina de Turing, nós temos uma memória e um estado em que o computador se encontra.

O processador logo que ligado começa a executar código a partir de um local fixo na memória, aonde esta residente um programa chamado BIOS ou UEFI,  esse programa por sua vez faz testes básicos, uma configuração inicial do hardware, e contém instruções de carregamento do sistema operacional.
O sistema operacional então assume o controle.

### Mas qual a relevância para o curso?
A linguagem de programação C é considerada uma linguagem de programação de baixo nível, pois você está próximo do hardware, o seu controle do hardware é maior, e o compilador traduz o seu código em instruções para o processador, resultando num código mais otimizado e rápido.

Portanto o entendimento básico do funcionamento dos computadores é necessário.

### Algoritmos
Ao desenvolver um sistema você precisa de ter em mente a sequencia lógica de como o aplicativo deverá funcionar, as instruções e os passos necessários para realizar o objetivo final.

Exemplo:  Como fazer café: Em uma caneca aquecer 1 litro de água até 97ºC → Colocar filtro de papel no suporte → Colocar suporte na garrafa térmica → Colocar os grãos de café no moedor →Moer o grão de café → Despejar o pó de café no filtro → Despejar a água no filtro → Aguardar que toda a agua passe pelo filtro → Retirar suporte do filtro → FIM

Como podemos ver é necessário que nós tenhamos a “receita” para a solução de nosso problema antes de iniciarmos a programação  do nosso programa.

É como construir uma casa, é necessário que haja o planejamento e a modelagem da planta para que na hora da execução nós saibamos o que será necessário.

Na programação um simples fluxograma pode nos ajudar durante o desenvolvimentos, podemos adotar vários níveis de fluxograma para descrever minuciosamente processos complicados e também de maneira sumária descrever o funcionamento do programa.

### Sistema binário
Com o advento da tecnologia digital, os computadores em sua forma mais básica de processamento são sinais ligados e desligados, ou respectivamente 1 e 0.
Logo o sistema numérico binário surge para representarmos os números e dados para esses computadores.

Como esse sistema só tem dois estados, ligado(1) ou desligado(0) a representação é feita por números na potência de 2, sendo que a cada digito a sobe o expoente.

Por exemplo o número 1 representado em binário é 1, o número 10 representado em binário é 1010 pois temos 2^0 = 1 , 2^1 = 2,2^2 =4, 2^3 = 8, logo o cálculo é (23 * 1) + (22 * 0) + (21*1) + (20 * 0).

Hoje em dia estamos na era do 64bits , sabendo que um bit é 0 ou 1, isso significa que processadores de 64 bits, operam com palavras de 64 bits.

Assim como no sistema decimal nós temos dezenas, centenas, milhares, etc.. no sistema binário nos temos o bit,nibble(4bits) byte(8 bits), word(2 bytes/16 bits), double word(4 bytes/32 bits), quad word(8 bytes / 64 bits), sendo assim o valor máximo que se cabe em cada um é respectivamente 1,16,256,65.535,4.294.967.625.

Isso se torna importante para sabermos o tamanho de cada tipo de dado na programação, acredite, muitos problemas acontecem pela tipagem incorreta de dados.

A calculadora presente no Windows tem o modo “programador” que permite a visualização e conversão fácil entre esses tipos de dados.

### Sistema hexadecimal
O sistema hexadecimal é parecido com o nosso sistema decimal, se em vez de 10 dedos nas mãos tivessemos 15, esse seria o sistema adotado globalmente, nesse sistema nós temos os números 0,1,2,3,4,5,6,7,8,9,A,B,C,D,E,F; e esses números correspondem ao sistema decimal como 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15, sua conversão.

Esse sistema facilita a trabalharmos com números grandes que seriam inconvenientes trabalhar com binário, pois apenas um algarismo em hexadecimal tem o poder de armazenar 4 bits, ou seja em vez de representarmos um número em binário como 1111 1111 nós podemos representar como FF em hexadecimal. 

### Exercícios
- Desenvolva um fluxograma de apenas um nível descrevendo o processo de lavar um carro.

- Desenvolva um fluxograma descrevendo o processo de escovar os dentes e um nível mais detalhado do processo da preparação da escova de dente.

- Converta os números em binário*

a) 255 b) 127 c)23 d)56

- Converta os números do exercício anterior em hexadecimal*

*exercícios 3 e 4 podem ser feitos utilizando a calculadora do windows

## Ambientes de Desenvolvimento
São programas(ou conjunto de programas) destinados a facilitar o desenvolvimento de um sistema, podendo agrupar  códigos-fontes em um projeto, fornecer ferramentas de compilação e depuração e também controle de versão.

Apesar da comodidade, esses ambientes não são obrigatórios, sendo possível programar utilizando um simples editor de texto e um compilador e um depurador(debugger).

### Opções de IDE gratuitas no Windows:
- Dev-C++
- Code::Blocks
- Visual Studio Community
- Visual Studio Code
- Etc..

## Estrutura básica de um programa em C.

```c
#include<stdio.h>

int invertesinal(int numero){
    return numero * -1;
}

int main(int argc, char* argv[]){
    int positivo, negativo;
    printf("Insira um número:");
    scanf("%d",&positivo);
    negativo = invertesinal(positivo);
    printf("%d resulta da função que troca o sinal do número",negativo);

    return 0;
}
```

Esse programa exemplo foi feito para mostrar a estrutura basica de um programa em C, essa estrutura consiste em:
- Inclusão de bibliotecas
- Definições de macros.
- Funções complementares do programa
- Função Principal do programa que dentro de si contém:
- Declaração de Variáveis
- Chamada de funções da biblioteca stdio.h para manipulação de variáveis
- Chamada da função complementar para manipular variáveis
- Retorno de um valor sinalizando a execução bem sucedida para o sistema operacional.

### Bibliotecas
Bibliotecas são arquivos com códigos já escritos ou compilados que executam certas funções, por exemplo a biblioteca “stdio.h” contém funções básicas de entrada e saída de dados, como a função printf() que exibe um texto formatado na saída padrão de dados(stdout), ou a função scanf() que recebe dados formatados da entrada padrão de dados.

Em grandes sistemas é comum que os desenvolvedores criem bibliotecas próprias contendo funções de uso comum no código.

Para incluir uma biblioteca de código utilize #include<nomedoarquivo.h> para incluir bibliotecas padrões presente na pasta do compilador ou utilize #include”nomedoarquivo.h” para incluir uma biblioteca presente no mesmo diretório do arquivo fonte.

### Funções complementares
Por motivos de organização e eficácia, os programadores fazem funções complementares no código quando o mesmo código deve ser executado mais de uma vez em locais distintos, embora no exemplo dado a utilização da função é apenas uma vez, ela foi colocada para a ilustração do conceito.

As funções podem ser escritas antes da função principal, ou podem ter sua “assinatura” descrita antes da função principal e sua implementação pode ser feita após a função principal.

### Função principal
A função principal é a função que é executada automaticamente quando o seu programa é carregado pelo sistema operacional, é o ponto de entrada do seu programa.

Nessa funções temos declarações de variáveis da função, e chamadas de outras funções e manipulação de variáveis com operadores, ou seja, tudo que o seu programa tem que fazer.

Por fim é uma boa pratica retornar para o sistema operacional o valor 0 caso o programa tenha sido executado com sucesso e esteja pronto para ser encerrado sem problemas.

A assinatura da função principal é :
```c
int main(int argc, char* argv[])
```
Na linguagem de programação C os símbolos de chave “{” e “}” definem blocos de código, no caso da função main, o seu programa deverá estar contido em um bloco de código, por isso podemos ver no exemplo após a declaração da função nós temos um “{” e após a ultima linha de código da função main temos um “}”, e nos finais de linha devemos colocar o  token ;

### Declaração de variáveis
Na linguagem de programação C as variáveis podem ser declaradas em qualquer lugar do código, porém é uma boa prática declarar todas as variáveis que serão utilizadas no inicio da função.

O escopo das variáveis locais(declaradas dentro de uma função) são apenas dentro da função, ao contrário de variáveis globais(declaradas fora de uma função) , mas muito cuidado com variáveis globais.

Os nomes de variáveis são “case sensitive”, ou seja a variável x é diferente da variável X.

No C as variáveis obrigatoriamente tem um tipo, na próxima aula nos aprofundaremos nos tipos.

Vale ressaltar que ao declarar uma variável, a mesma não é inicializada automaticamente(contém lixo em seu conteúdo), mas você pode inicializar a variável.

A declaração de uma variável no código é feita `tipo nome`, por exemplo `int x`, podemos criar mais de uma variável na mesma declaração, por exemplo `int x,y,z`.

Opcionalmente podemos inicializar a variável em sua declaração, por exemplo `int x=1,y=2,z`.

Observe que a variável x foi inicializada com o valor 1, a variável y foi inicializada com o valor 2 e a variável z não foi inicializada.

### Chamada de função
Na linguagem de programação C as funções são declaradas com sua assinatura, as funções são declaradas opcionalmente com as propriedades da função, obrigatoriamente com o tipo de retorno, o nome e seus parâmetros entre parênteses com seu respectivo tipo , por exemplo  int invertesinal(int numero)
A palavra int define o tipo de retorno, que no caso é um número inteiro, “invertesinal” é o nome da função, entre parênteses temos o parâmetro chamado “numero” que é do tipo int.

### Uso de operadores
Como o nome já deixa a entender, os operadores fazem alguma operação, na linguagem de programação C temos operadores unários, binários e ternários, sendo respectivamente um operador que tem um operando, dois operandos ou três operandos.

No nosso exemplo temos `negativo = invertesinal(positivo);`
Aonde o operador “=” atribui o valor de retorno da função “invertesinal” para a variável “negativo”.

Em C ao atribuir algum valor com apenas números, é considerado uma atribuição de um valor decimal(x = 255), se atribuirmos precedendo o valor com 0x, estamos atribuindo um valor em hexadecimal(x = 0xFF), e ao atribuir com a letra b precedendo o valor, estamos atribuindo um valor em binário(x = b11111111), em ambos os casos ilustrados aqui, estamos atribuindo o mesmo valor para x.

## Operadores e Tipagem de Dados

### Tipagem de dados
Na linguagem de programação C todos os dados tem um tipo, mesmo, seja ele um número inteiro, um número real, uma letra, um conjunto de letras, um valor lógico, ou mesmo um tipo que não tem tipo.

Cada tipo tem seu tamanho definido, e esse tamanho deve ser sempre levado em consideração, pois caso o limite desse tamanho seja atingido e uma tentativa de adicionar algo a esse valor acontecer, um underflow ou overflow ocorrerá.

No caso do overflow é fácil de ilustrar com o exemplo real de um copo, se tivermos um copo de 250ml e tentarmos adicionar 300ml, 50 ml transbordarão para locais indesejáveis.

No caso de tipagem numéricas em C, um overflow pode redefinir esse valor máximo + 1 para o valor mínimo, fazendo uma analogia seria em um relógio ao chegarmos as 23:59 de um dia e ao adicionar um minuto voltaríamos para 00:00, e no caso de underflow, subtrairíamos 1 minuto de 00:00 e chegaríamos a 23:59.

### Tipos de dados básicos em C
Os principais tipos de dados na linguagem C são:
- int: inteiro de 32bits suporta números de −2.147.483.647 até +2.147.483.647 observe que o tipo int pode ter tamanho diferente em diferentes plataformas
- char: 8 bits, representa caracteres da tabela ASCII
- float: numero real, tem casa decimal.
- void: um tipo de dado sem tamanho, se você tentar fazer sizeof(void) um erro na hora de compilar ocorrerá, pois void não tem tamanho.

### Características adicionais dos tipos 
Tipos numéricos podem ser do tipo longe terem o dobro de tamanho.

Tipos numéricos podem ser unsigned, ou seja sem sinal, por exemplo enquanto um int pode ter números de −2.147.483.647 até +2.147.483.647, um unsigned int pode ter números de 0 até 4.294.967.295, que pode ser conveniente para quando você precisa de um alcance maior e tem certeza de que os valores não serão negativos.

### Operadores unários
- +: Operador unário, pode ser utilizado para definir valores positivos(ex : int x = +4)
- -: Operador unário, utilizado para definir valores negativos(ex: int x = -4)
- ++: Operador unário de incremento, utilizado para incrementar em um o valor, podendo ser utilizado como um pré incremento(++x) ou pós incremento(x++)
- --: Operador unário de decremento, utilizado para decrementar em um o valor, podendo ser utilizado como um pré decremento(--x) ou pós decremento(x--)
- &: Operador de endereço, utilizado para se ter o endereço do operando na memória.
sizeof(): Operador de tamanho, utilizado para se ter o tamanho do operando na memória
- *: Operando de dereferencia, utilizado para se ter o valor do operando na memória(assim como a multiplicação é o inverso da divisão, o operador * é o inverso do &)
- !: Operador de negação lógica, ele inverte o conteúdo lógico do operando(ex: bool x = !true ; o valor de x será false)
- ~: Operador de negação de bit, por exemplo x = b10101010 então ~x = b01010101

### Operadores aritméticos
- + : Operador de adição, soma dois operandos.
- - : Operador de substração, subtrai dois operandos.
- / : Operador de divisão, divide dois operandos.
- %: operador de módulo, resto da divisão entre dois operandos
- *: operador de multiplicação, multiplica dois operandos 

### Operadores relacionais 
- ==: operador de igualdade, verifica se o operando da esquerda é igual o da direita (2 ==2).
- >: operador de maioridade,verifica se o operando da esquerda é maior que o da direita(3 > 2).
- >=:operador de maioridade ou igualdade, verifica se o operando da esquerda é maior ou igual o da direita(3>=2).
- <=: operador de minoridade ou igualdade, verifica se o operando da esquerda é menor ou igual o da direita(2<= 3).
- <: operador de minoridade, verifica se o operando da esquerda é menor que o operando da direita(2<3).

### Operadores lógicos
- &&: Operador  “E” , é verdade quando o operando da direita e o operando da esquerda são verdadeiros
- ||: Operador “Ou”, é verdade quando o operando da direita é verdade ou o operando da esquerda é verdade, ambos operando verdadeiros também resulta em verdade.
- !:Operador “não”, visto em um slide passado. 

### Operadores lógicos bitwise
- &: Operador E a nível de bits(ex: 11110011 & 11000001 = 11000001)
- |: operador OU a nível de bits(ex:00010010 | 11101100 = 11111110)
- ^:operador “OU EXCLUSIVO” a nível de bits, Só é verdadeiro se um dos operandos for verdade, se ambos forem verdade é falso (ex: 10010001 ^11000011 = 01010010)
- ~: operador “não” bitwise, inverte os bits(ex: ~11100111 = 00011000)
- <<: operador shift-left, empurra os bits para direita(ex:<< 10001111 = 0001110)
- >>:operador shift-right, empurra os bits para a esquerda(ex: >>11110000 = 01111000) 

### Operadores de atribuição
- = : Operador de atribuição simples(ex: a = 10 significa que o valor de 10 esta sendo atribuído a variável “a”)
- +=: Atribuição e soma (ex: a+=10 é o mesmo que a = a + 10)
- -: Atribuição e subtração (ex: a-=10 é o mesmo que a = a - 10)
- /=: Atribuição e divisão (ex: a/=10 é o mesmo que a = a / 10)
- *=: Atribuição e multiplicação(ex: a*=10 é o mesmo que a = a * 10)
- %=:Atribuição e módulo (ex: a%=10 é o mesmo que a = a % 10)
- <<=: Atribuição e shift-left(ex: a<<=10 é o mesmo que a = a << 10)
- >>=: Atribuição e shift-right(ex: a>>=10 é o mesmo que a = a >> 10)