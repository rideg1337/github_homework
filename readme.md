# Github Homework

[![Build and Push Docker image](https://github.com/rideg1337/github_homework/actions/workflows/docker-build.yml/badge.svg)](https://github.com/rideg1337/github_homework/actions/workflows/docker-build.yml)
[![Docker Hub](https://img.shields.io/badge/docker-repo-blue?logo=docker&logoColor=white)](https://hub.docker.com/r/rideg1337/devops-homework)


##### Tartalomjegyzek
* [A feladat](#a-feladat)
* [Dockerfile](#dockerfile)
* [index.html](#indexhtml)
* [Github Workflows](#github-workflows)
    * [Mi tortenik itt valojaban?](#mi-tortenik-itt-valojaban)
* [Hogy futtasd a containert?](#hogy-futtasd-a-containert)
* [Ellenorzes](#ellenorzes)

##### A Feladat:

GitHub actions létrehozása ami egy nginx webszerver docker image-t fog építeni a Dockerfile-ból a saját
Docker Hub fiókod alá, mint "homework:latest" (kép:tag).


---


##### Dockerfile:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html


EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

```

* Itt egy nagyon egyszeru ```nginx``` image-bol letrehozzuk a webszerverunket ```alpine``` alapon.
* Majd utana felmasoljuk az elore definialt ```index.html``` fajlunkat az ```nginx``` root konyvtaraba: ```/usr/share/nginx/html/index.html``` ez fog szamunkra szolgalni mint default page.
* Kinyitjuk a 80-as portot.
* Majd elinditjuk az nginx webszervert:
```CMD ["nginx", "-g", "daemon off;"]```

##### index.html:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DevOps Homework</title>
</head>
<body>
    <h1>DevOps Homework by: Rideg Zsolt</h1>
</body>
</html>
```

##### Github workflows:

```yml
name: Build and Push Docker image

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
    - name: Check out the repo
      uses: actions/checkout@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v2
      with:
        context: .
        file: ./Dockerfile
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/devops-homework:latest
```

##### Mi tortenik itt valojaban?

A workflow ```push```-ra aktivalodik de csak a ```main``` branchen
```yaml
on:
  push:
    branches:
      - main
```

A jobs-nal definialjuk eloszor a runnert ami esetunkben egy ```ubuntu-latest```
```yml
jobs:
  build-and-push:
    runs-on: ubuntu-latest
```
Majd kovetkeznek a lepesek:
```yml
steps:
```
Letolti az aktualis repot, esetunkben az utolso commit-ot
```yml
name: Check out the repo
      uses: actions/checkout@v2
```
Bejelentkezik ```dockerhub```-ra megadott ```secrets```-ek segitsegevel amit a repo beallitasainal tudunk hozzaadni majd elmenti egy valtozoba.

```yml
- name: Log in to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
```

Majd felepiti az image-t es felpusholja a ```dockerhub``` fiokunk-ra
```yml
 - name: Build and push Docker image
      uses: docker/build-push-action@v2
      with:
        context: .
        file: ./Dockerfile
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/devops-homework:latest
```

##### Hogy futtasd a containert?

Mivel a pipeline elvegezte a feladatot, igy ez az image mar fent a sajat dockerhub fiokunkon. (A dockerhub reponknak ez esetben publikusnak kell lennie!)

Letoltjuk az image-t
```bash
docker pull rideg1337/devops-homework:latest
```
Majd futtatjuk peldaul:
```bash
docker run --name hazi-feladat -d -p 8080:80 rideg1337/devops-homework:latest
```

##### Ellenorzes:

```bash
$ curl localhost:8080
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DevOps Homework</title>
</head>
<body>
    <h1>DevOps Homework by: Rideg Zsolt</h1>
</body>
</html>
```
