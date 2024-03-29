<div>
    <img align="left" height="120" src="./assets/ufop.png">
    <p> 
        Universidade Federal de Ouro Preto
        <br>
        Disciplina: BCC362 - Sistemas Distribuidos
        <br>
        Professor: Carlos Frederico
    </p>
</div>
<hr />

# Trabalho Pratico I

<details>

## Testando o Dockerfile:

``` bash

vagrant up
vagrant ssh

```

- Dentro do ambiente vagrant:

``` bash

docker run -it myimage:1.0 /bin/sh
ls

```

## Criando diretamente no Host:

``` bash

docker build -t <NOME_DA_IMAGEM>:<VERSAO> --build-arg NAME=<NOME> --build-arg CLASS=<TURMA> .
docker run -it <NOME_DA_IMAGEM>:<VERSAO> /bin/sh

```
## Subindo num Registry:

- Lembre-se de criar o repositorio no DockerHub

### Pipeline:

``` bash

name: Docker Image CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:

  build-and-deploy:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Login Docker Hub
      run: echo '${{ secrets.DOCKER_PASSWORD }}' | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
    - name: Build and tag the Docker image
      run: docker build . --file tpi/Dockerfile --tag "${{ secrets.IMAGE_NAME }}":1.0 --build-arg NAME=<NAME> --build-arg CLASS=<CLASS> 
    - name: Push the Docker image to Docker Hub
      run: docker push "${{ secrets.IMAGE_NAME }}":1.0
    - name: Logout from Docker Hub
      run: docker logout

```

### Diretamente:

``` bash

    echo '<PASSWORD>' | docker login -u "<USERNAME>" --password-stdin
    docker build . --file tpi/Dockerfile --tag <IMAGE_NAME>:<VERSAO> --build-arg NAME=<NAME> --build-arg CLASS=<CLASS> 
    docker push <REGISTRY/REPOSITORIE>:<VERSION>
    docker logout

```

</details>

<hr/>

# Trabalho Pratico II

## Enunciado

- Discorra e apresente os conceitos de eleições
de processos apresentados em aula:
  - Problema em pauta:
    - Objetivos
    - Suposições/hipotes
  - Abordagens:
    - Algoritmo do valentão
    - Algoritmo do anel
    - Eleição em ambiente sem fio
      - Eleição em rede ad hoc
    - Eleição em sistemas de grande escala
    - Aborde um novo exemplo
  - Demonstre uma implementação de um dos algoritmos abordados
- Discorra e apresente os conceitos e abordagens do Rabbitmq

## Eleições de Processos

### Introdução

A eleição de processos é um conceito de sistemas distribuidos.
Procesos em sistemas distribuidos não compartilham memoria, 
cada um tendo alocação de recursos particulares e independentes. 

Sendo assim, tais processos se comunicam através de uma rede, 
o que leva a necessidade da existencia de um **coordenador**. 
Um processo dito coordenador realiza operações das quais 
outro processo depende. Essa caracteristica levou a criação 
dos ditos **Algoritmos de eleição**.

Esses algoritmos escolhem um processo num conjunto distribuido
para que este opere como coordenador. Caso esse processo falhe
por qualquer razão, um novo coordenador, então, é eleito dentro
do conjunto.

### Algoritmos

#### Algoritmo do Valentão

Esse algoritmo é aplicado num sistema onde **todos os procesos
podem se comunicar entre si**. Ele opera da seguinte maneira:

<details>

```mermaid
flowchart LR;
subgraph "Sistema Distribuido"

  subgraph " Prioridade inferior a P "
    A; B; C; 
  end

  P;

  subgraph " Prioridade superior a P "
    S; Q; V;
    Cdn[(Coordenador)]
  end

end
```

```mermaid
---
title: Processo P envia uma mensagem ao coordenador.
---
flowchart LR;
subgraph "Sistema Distribuido"

  subgraph " Prioridade inferior a P "
    A; B; C; 
  end

  subgraph " Prioridade superior a P "
    S; Q; V; 
    Cdn[(Coordenador)]
  end

  P-. Mensagem .-> Cdn;
end
```

```mermaid
---
title:  Caso o coordenador não responda num intervalo T, se assume que o coordenador falhou.
---
flowchart LR;
subgraph "Sistema Distribuido"

  subgraph " Prioridade inferior a P "
    A; B; C; 
  end

  P;

  subgraph " Prioridade superior a P "
    S; Q; V; 
    Cdn[(Coordenador)]
  end

  Cdn -. Falha .-x P;
end
```

```mermaid
---
title: Processo P envia uma mensagem de eleição ao todos os processos com maior numero de prioridade que o seu.
---
flowchart LR;
  subgraph " Prioridade inferior a P "
    A; B; C; 
  end

  P;

  subgraph " Prioridade superior a P "
    S; Q; V;
    Cdn[(Coordenador)]
  end
  
  P-. Eleição .->S
  P-. Eleição .->V
  P-. Eleição .->Q
```

```mermaid
---
title:  Caso nenhum processo de maior prioridade o responda num intervalo T, P elege a si mesmo como coordenador.
---
flowchart LR;
  subgraph " Prioridade inferior a P "
    A; B; C; 
  end

  P[(Coordenador)]

  subgraph " Prioridade superior a P "
    S; Q; V;
  end

  S -. Falha .-x P
  Q -. Falha .-x P
  V -. Falha .-x P
```

```mermaid
---
title:  P então envia uma mensagem a todos os processos com prioridade inferior a si mesmo, dizendo que é o novo coordenador.  
---
flowchart LR;
  subgraph " Prioridade inferior a P "
    A; B; C; 
  end

  P[(Coordenador)]

  subgraph " Prioridade superior a P "
    S; Q; V;
  end
  
  P-. Posse .->C
  P-. Posse .->B
  P-. Posse .->A
```

```mermaid
---
title: Porém, existe o caso em que um processo Q responde P dentro do tempo T
---
flowchart LR;
  subgraph " Prioridade inferior a P "
    A; B; C; 
  end

  P;

  subgraph " Prioridade superior a P "
    S; Q; V;
  end

  Q -. Ok .->P
```

```mermaid
---
title: Então o processo P aguarda novamente um intervalo de tempo T para receber uma nova mensagem de Q, dizendo que é o novo coordenador.
---
flowchart LR;
  subgraph " Prioridade inferior a P "
    A; B; C; 
  end

  P;

  subgraph " Prioridade superior a P "
    S; Q[(Coordenador)]; V;
  end

  Q -. Posse .->P
```

```mermaid
---
title: Caso Q não responda dentro do intervalo, P assume que houve alguma falha e o processo reinicia. 
---
flowchart LR;
  subgraph " Prioridade inferior a P "
    A; B; C; 
  end

  P;

  subgraph " Prioridade superior a P "
    S; Q; V;
  end    
  
  Q -. Falha .-x P
  P-. Eleição .->S
  P-. Eleição .->Q
  P-. Eleição .->V
```
</details>

#### Algoritmo do Anel

Esse algoritmo é aplicado num sistema onde organizado
numa forma circular. Um proceso se comunica apenas com
o proximo a direita, de maneira **unidiecional**. 

<details>

```mermaid
flowchart LR;
    Pa --> Pb --> Pc --> Pd --> Pf --> Pa 
```

```mermaid
---
title: Proceso Pf falha 
---
flowchart LR;
    Pa -->Pb --> Pc --> Pd --x Pf --> Pa 
```

```mermaid
---
title: Proceso Pb inicializa a eleição 
---
flowchart LR;
    Pa --> Pb -. Eleição .-> Pc --> Pd --> Pf --> Pa 
```


```mermaid
---
title: Proceso Pb inicializa a eleição 
---
flowchart LR;
    Pa -->Pb -. Eleição .-> Pc --> Pd --> Pf -->Pa 
```

```mermaid
---
title: A eleição é passada adiante até que se encontre o processo de hierarquia mais alta.   
---
flowchart LR;
    Pa --> Pb -. Eleição .-> Pc -. Eleição .-> Pd -. Eleição .-x Pf --> Pa;
    Pd -. Eleição .-> Pa; 
```

```mermaid
---
title: O processo eleito então comunica sua posse ao sistema.   
---
flowchart LR;
    Pa -. Eleição .-> Pb -. Eleição .-> ... -. Eleição .-> Pd -. Eleito .-> Pa; 
```

```mermaid
---
title: O processo eleito então comunica sua posse ao sistema.   
---
flowchart LR;
    Pa -. Eleito .-> Pb -. Eleito .-> ... -. Eleito .-> Pd -. Eleito .-> Pa; 
```

```mermaid
---
title: Fim da eleição.   
---
flowchart LR;
    Pa --> Pb --> Pc --> Pd -->Pa; 
```
</details>

#### Eleição em rede ad hoc

Uma rede é considerada ad hoc em sistemas distribuidos quando um par de 
nodulos se comunicam através de uma fonte direta ou atraves de um ou mais
nodulos intermediarios. Os pares que estão proximos uns dos outros, dentro
do limite de transmissão, são os unicos capazes de se comunicarem entre si.

Uma falha de rede então ocorre quando algum elemento do par se move e não 
se encontra mais dentro do raio de transmissão de seu par. Da mesma maneira,
uma conexâo ocorre quando um nodulo antes distante, encontra-se então dentro
do raio de algum outro nodulo.

Essa dinamica estabelece um espaço cuja topologia está em constante mudança,
e rompe com o paradigma de redes estãticas e seus processos de eleição. A
diferença consiste no fato de que por vezes, existirões partições separadas,
tanto como um conjunto de nodulos assim como nodulos completamente isolados.
Isso detona que dentro de um conjuto distribuido movel, haverão momentos 
em que existirão multiplos lideres para diferentes partições e até mesmo
nodulos desprovidos de lideres. As eleições nesse paradigma devem então 
garantir que num conjunto de nodulos, apenas um lider exista, ou sobreviva
dentre outros. 

O grafo desenhado abaixo foi extraido da segunda referencia deste documento,
que aborta o algoritmo TORA para a escolha de lideres em redes Ad Hoc.
A diferença principal será a explicação cuidadosa dos relacionamentos
dos nodulos e suas transformassões, abstraindo o funciomaneto do algoritmo.


```mermaid
---
title: Estado inicial da Rede Ad Hoc    
---
flowchart LR;
    A(("(A,-1,-1,-1,0,A)"))
    B; D; E; F; H; G;
  
    B-->A
    B-->F
    B-->D
    D-->A
    D-->F

    E-->F
    H-->F
    G-->H

```

```mermaid
---
title: Estado inicial da Rede Ad Hoc    
---
flowchart LR;
  subgraph " Partição 1 "
    A(("(A,-1,-1,-1,0,A)"))
    B; D; 
  end

  subgraph " Partição 2 "
    E; F; H; G;
  end
  
    B-->A
    B-->D
    D-->A

    E-->F
    H-->F
    G-->H

```

## Rabbitmq

## Referencias

- [https://www.geeksforgeeks.org/election-algorithm-and-distributed-processing/](https://www.geeksforgeeks.org/election-algorithm-and-distributed-processing/)
- [https://people.engr.tamu.edu/j-welch/papers/dialm00.pdf](https://people.engr.tamu.edu/j-welch/papers/dialm00.pdf)
