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
