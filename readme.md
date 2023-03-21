# README - Déploiement d'une application Flask avec GitHub et GitHub Actions

Ce document présente les étapes pour déployer une application Flask "Hello World" en utilisant GitHub comme dépôt et GitHub Actions pour la CI/CD, en suivant les bonnes pratiques DevOps.

## Table des matières

1. [Prérequis](#prérequis)
2. [Initialisation du projet](#initialisation-du-projet)
3. [Création de l'application Flask](#création-de-lapplication-flask)
4. [Conteneurisation avec Docker](#conteneurisation-avec-docker)
5. [Configuration de GitHub Actions](#configuration-de-github-actions)
6. [Déploiement](#déploiement)

## Prérequis

- Un compte GitHub
- Git installé localement
- Python 3 installé localement
- Docker installé localement

## Initialisation du projet

1. Créez un nouveau dépôt sur GitHub et clonez-le localement :

```bash
git clone https://github.com/<username>/flask-hello-world.git
cd flask-hello-world 
```



2. Créez un environnement virtuel Python et activez-le :

```bash
python3 -m venv venv
source venv/bin/activate
```

## Création de l'application Flask

1. Installez Flask dans l'environnement virtuel :

```bash
pip install flask
```

3. Créez un fichier `app.py` contenant le code suivant :

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

4. Créez un fichier `requirements.txt` pour lister les dépendances du projet :

```bash
pip freeze > requirements.txt
```

5. Testez l'application localement :

```bash
python app.py
```

Ouvrez un navigateur et accédez à `http://localhost:5000` pour vérifier que l'application fonctionne.

6. Ajoutez les fichiers au dépôt Git et effectuez un commit :

```bash
git add .
git commit -m "Initial Flask application"
git push origin main
```

## Conteneurisation avec Docker

1. Créez un fichier `Dockerfile` avec le contenu suivant :

```Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

2. Construisez l'image Docker localement :

```bash
docker build -t flask-hello-world .
```

3. Testez l'image Docker localement :

```bash
docker run -p 5000:5000 flask-hello-world
```

Vérifiez que l'application fonctionne en accédant à `http://localhost:5000`.

4. Ajoutez les fichiers au dépôt Git et effectuez un commit :

```bash
git add .
git commit -m "Add Dockerfile and .dockerignore"
git push origin main
```

## Configuration de GitHub Actions

1. Créez un fichier .github/workflows/ci-cd.yml avec le contenu suivant :

```yaml
name: CI/CD Pipeline

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.9

      - name: Install dependencies
        run: |
          python -m venv venv
          source venv/bin/activate
          pip install -r requirements.txt

      - name: Run tests
        run: |
          source venv/bin/activate
          # Add your test commands here, for example:
          # pytest

  build_and_push_docker:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker
        run: |
          sudo systemctl start docker

      - name: Build Docker image
        run: |
          docker build -t flask-hello-world .

      - name: Log in to Docker registry
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Push Docker image
        run: |
          docker tag flask-hello-world ${{ secrets.DOCKER_USERNAME }}/flask-hello-world:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/flask-hello-world:latest
```

## Déploiement

Le fichier de configuration GitHub Actions fourni construit et pousse l'image Docker de l'application Flask sur Docker Hub. Vous pouvez maintenant déployer cette image sur n'importe quelle plate-forme de votre choix, par exemple, un serveur dédié, un cluster Kubernetes ou un fournisseur de cloud (AWS, GCP, Azure, etc.).

1. Pour déployer l'application sur un serveur, installez Docker sur le serveur et exécutez la commande suivante :

```bash
docker run -d -p 5000:5000 --name flask-hello-world $DOCKER_USERNAME/flask-hello-world:latest
```


2. Pour déployer l'application sur un cluster Kubernetes, créez un fichier deployment.yaml et ajustez les valeurs en fonction de votre configuration :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-hello-world
  template:
    metadata:
      labels:
        app: flask-hello-world
    spec:
      containers:
      - name: flask-hello-world
        image: $DOCKER_USERNAME/flask-hello-world:latest
        ports:
        - containerPort: 5000

```


Appliquez la configuration de déploiement avec la commande suivante :

```bash
kubectl apply -f deployment.yaml
```


## Conclusion

En suivant ces étapes, vous avez créé et déployé une application Flask "Hello World" en utilisant GitHub pour le versionnement du code et GitHub Actions pour automatiser le processus de construction, de test et de déploiement de l'application. Ceci est un exemple de l'adoption des bonnes pratiques DevOps pour améliorer la collaboration entre les équipes et accélérer le processus de livraison de l'application.

N'oubliez pas que l'adoption de la culture DevOps nécessite également la mise en place d'indicateurs de suivi, la formation des équipes sur les outils et les méthodologies, et l'amélioration continue des processus pour optimiser les performances.



