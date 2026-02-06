# Partie 1 : DevOps (10h TD en 5 séances de 2h)

## Pré-requis pour tous les TD DevOps

1. **Java (JDK) 25 ou supérieur**  
   - [Installation OpenJDK 25](https://openjdk.org/projects/jdk/25/)  
   - Vérifier l’installation :

     ```bash
     java -version
     ```

2. **Maven**  
   - [Téléchargement et installation Maven](https://maven.apache.org/download.cgi)  
   - Vérifier l’installation :  

     ```bash
     mvn -v
     ```

3. **Git**  
   - [Installation de Git](https://git-scm.com/downloads)  
   - Vérifier l’installation :  

     ```bash
     git --version
     ```

4. **Docker**  
   - [Installer Docker Desktop (Windows/Mac)](https://www.docker.com/products/docker-desktop/) ou [Docker Engine (Linux)](https://docs.docker.com/engine/install/)  
   - Vérifier l’installation :  

     ```bash
     docker --version
     ```

5. **Compte GitHub** (ou GitLab)  
   - [Créer un compte GitHub](https://github.com/join)

    *(Les étapes ci-après sont illustrées pour GitHub et GitHub Actions, mais vous pouvez adapter pour GitLab et GitLab CI.)*

---

## TD DevOps 1 (2h)  

### Thème : Prise en main de Git & Mise en place d’un pipeline CI basique

**Objectifs :**  

- Créer un dépôt Git pour un projet Java/Maven.  
- Mettre en place une première GitHub Action (CI) qui se déclenche à chaque push.

---

### Étapes détaillées Git

1. **Installer/Configurer Git**  
   - Suivre le lien d’installation plus haut.  
   - Configurer votre nom d’utilisateur et e-mail (pour les commits) :  

     ```bash
     git config --global user.name "VotreNom"
     git config --global user.email "votre_email@example.com"
     ```

2. **Créer un compte GitHub** *(si ce n’est pas déjà fait)*  
   - Allez sur [https://github.com/join](https://github.com/join)  
   - Remplissez le formulaire pour vous inscrire.

3. **Création d’un nouveau dépôt sur GitHub**  
   - Connectez-vous à votre compte.  
   - Cliquez sur **New** ou accédez à [https://github.com/new](https://github.com/new).  
   - Donnez un nom à votre dépôt, ex. `demo-devops-java`.  
   - Laissez-le en **Public** ou **Privé** (selon votre préférence).  
   - Cochez l’option “Add a README file” si vous le souhaitez.

4. **Clonage du dépôt en local**  
   - Copiez l’URL HTTPS de votre dépôt (ex. `https://github.com/VotreUser/demo-devops-java.git`).  
   - Dans un terminal, exécutez :  

     ```bash
     git clone https://github.com/VotreUser/demo-devops-java.git
     cd demo-devops-java
     ```

5. **Initialisation d’un projet Maven**  
   - Dans le dossier cloné, exécutez la commande Maven pour générer un squelette de projet :  

     ```bash
     mvn archetype:generate \
       -DgroupId=com.example \
       -DartifactId=demo-app \
       -DarchetypeArtifactId=maven-archetype-quickstart \
       -DarchetypeVersion=1.0.0 \
       -DinteractiveMode=false
     ```

   - Cette commande crée un dossier `demo-app` contenant `pom.xml`, `src/main/java/...`, etc.

6. **Premier commit & push**  
   - Ajoutez tous les fichiers :  

     ```bash
     git add .
     git commit -m "Initial Maven project"
     git push origin main
     ```

   - Vérifiez sur GitHub que votre code est bien en ligne.

7. **Mise en place d’un pipeline CI basique (GitHub Actions)**  
   - Dans votre dépôt GitHub, cliquez sur l’onglet **Actions**.  
   - Choisissez **Set up a workflow yourself** (ou un template “Java with Maven”).  
   - Créez un fichier (par exemple) `.github/workflows/ci.yml` avec le contenu minimal suivant :

     ```yaml
     name: CI

     on:
       push:
         branches: [ "main" ]
       pull_request:
         branches: [ "main" ]

     jobs:
       build:
         runs-on: ubuntu-latest
         steps:
           - uses: actions/checkout@v2
           - name: Set up JDK 25
             uses: actions/setup-java@v5
             with:
               java-version: '25'
           - name: Build with Maven
             run: mvn -B package --file demo-app/pom.xml
     ```

   - Commitez ce fichier.  
   - Rendez-vous dans l’onglet **Actions**, vous verrez votre workflow se lancer.

**À la fin de cette séance**, vous avez un dépôt Java/Maven avec un pipeline CI qui build le projet à chaque push.

---

<!-- TODO send students  cours 3 -->

## TD DevOps 2 (2h)  

### Thème : Intégration Continue avancée (Tests JUnit, Qualité du code)

**Objectifs :**  

- Ajouter des tests unitaires JUnit.  
- Automatiser leur exécution dans GitHub Actions.  
- Intégrer un outil de qualité de code (Checkstyle ou SpotBugs).

#### Étapes détaillées JUnit

1. **Ajouter des tests JUnit 5**  
  Dans `demo-app/src/test/java/com/example/AppTest.java`, ajoutez :

     ```java
     package com.example;

     import org.junit.jupiter.api.Test;
     import static org.junit.jupiter.api.Assertions.*;

     public class AppTest {

       @Test
       public void testSum() {
         int result = App.sum(2, 3);
         assertEquals(5, result, "2 + 3 should be 5");
       }
     }

  Dans `App.java` (dans `src/main/java/com/example`), créez la méthode sum :
        ```java
          public static int sum(int a, int b) {
            return a + b;
          }
          ```
2. **Test en local**  

- À la racine du projet, exécutez :

     ```bash
     mvn test -f demo-app/pom.xml
     ```

- Vous devriez voir un test réussi.

3. **Mettre à jour le workflow CI pour exécuter les tests**  
   - Dans `.github/workflows/ci.yml`, remplacez la commande de build par :

     ```yaml
     - name: Build and Test with Maven
       run: mvn clean install --file demo-app/pom.xml
     ```

   - Commitez, poussez.  
   - Dans l’onglet Actions, vérifiez que le job exécute bien les tests.

4. **Intégrer Checkstyle (exemple)**  
    - Dans `demo-app/pom.xml`, ajoutez la dépendance Checkstyle dans la section `<build>` -> `<plugins>` :

      ```xml
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-checkstyle-plugin</artifactId>
        <version>3.6.0</version>
        <configuration>
          <configLocation>google_checks.xml</configLocation>
          <encoding>UTF-8</encoding>
          <consoleOutput>true</consoleOutput>
          <failsOnError>true</failsOnError>
        </configuration>
        <executions>
          <execution>
            <phase>verify</phase>
            <goals>
              <goal>check</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      ```

  Téléchargez [google_checks.xml](https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml) et placez-le dans `demo-app/`.  
  Lancer les commandes :
    ```bash
     mvn clean verify -f demo-app/pom.xml
     ```
  Si vous avez des soucis de style, le build échouera.

5. **Automatisation avec GitHub Actions**  
   - Les étapes `mvn clean install` incluront la phase `verify`, donc Checkstyle sera exécuté.  
   - Commitez, poussez, observez le résultat.

**À la fin de cette séance**, la CI exécute tests + checkstyle, et échoue si la qualité du code n’est pas respectée.

---

## TD DevOps 3 (2h)  

### Thème : Déploiement Continu (CD) et Conteneurisation avec Docker

**Objectifs :**  

- Créer et pousser une image Docker de l’application Java.  
- Mettre en place la publication automatique (CD) via GitHub Actions.

---

### Étapes détaillées JAR

1. **Créer un jar exécutable**  
    - Dans `demo-app/pom.xml`, assurez-vous d’avoir le plugin `maven-assembly-plugin` ou `maven-shade-plugin` pour créer un fat jar (toutes dépendances incluses).  

    > Exemple pour `maven-assembly-plugin` :
        ```xml
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>3.8.0</version>
          <configuration>
            <archive>
              <manifest>
                <mainClass>com.example.App</mainClass>
              </manifest>
            </archive>
            <descriptorRefs>
              <descriptorRef>jar-with-dependencies</descriptorRef>
            </descriptorRefs>
          </configuration>
          <executions>
            <execution>
              <id>make-assembly</id>
              <phase>package</phase>
              <goals>
                <goal>single</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
        ```
    > Lancez :  
        ```bash
        mvn clean package -f demo-app/pom.xml
        ```
    > Vous obtiendrez un fichier `demo-app-1.0-SNAPSHOT-jar-with-dependencies.jar` dans `target/`.

2. **Créer un Dockerfile**  
   - Dans la racine du projet (ou dans `demo-app/`), créez un fichier `Dockerfile` :

     ```dockerfile
     FROM amazoncorretto:25-alpine
     COPY target/demo-app-1.0-SNAPSHOT-jar-with-dependencies.jar /app/demo-app.jar
     WORKDIR /app
     ENTRYPOINT ["java", "-jar", "demo-app.jar"]
     ```

3. **Build local de l’image**  
    - Assurez-vous d’avoir Docker en service (`docker --version`).  
    - Dans le dossier contenant le Dockerfile :

      ```bash
      docker build -t votreuser/demo-app:latest .
      ```

    > Testez l’image :
        ```bash
        docker run --rm votreuser/demo-app:latest
        ```
    Vous devriez voir la sortie “Hello World!” ou autre, selon votre `App.java`.

4. **Pousser l’image vers Docker Hub ou GitHub Container Registry**  
   - [Créer un compte Docker Hub](https://hub.docker.com/) si besoin.  
   - Connectez-vous via CLI :  

     ```bash
     docker login
     ```

   - Poussez l’image :

     ```bash
     docker push votreuser/demo-app:latest
     ```

5. **Automatiser dans GitHub Actions**  
   - Mettez à jour `.github/workflows/ci.yml` pour ajouter un job de build/push Docker **après** le succès des tests.  
   - Exemple (étapes clés) :

     ```yaml
     - name: Build JAR
       run: mvn clean package -f demo-app/pom.xml
     - name: Build Docker Image
       run: docker build -t votreuser/demo-app:latest .
     - name: Login to Docker Hub
       run: echo "${{ secrets.DOCKERHUB_PASSWORD }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin
     - name: Push Docker Image
       run: docker push votreuser/demo-app:latest
     ```

   - Dans **Settings > Secrets** de GitHub, ajoutez `DOCKERHUB_USERNAME` et `DOCKERHUB_PASSWORD` (cf. [doc GitHub sur les Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)).

**À la fin de cette séance**, vous avez un pipeline qui build, teste, et publie l’image Docker de votre app Java.

---

## TD DevOps 4 (2h)  

### Thème : Orchestration Kubernetes & Infrastructure as Code

**Objectifs :**  

- Déployer l’application dans un cluster Kubernetes local.  
- Découvrir l’approche Infrastructure as Code (IaC).

---

### Étapes détaillées K8S

1. **Installer minikube ou kind**  
   - [Installer minikube](https://minikube.sigs.k8s.io/docs/start/) ou [kind](https://kind.sigs.k8s.io/).  
   - Exemple (Linux) pour kind :  

     ```bash
     curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
     chmod +x kind
     sudo mv kind /usr/local/bin/
     ```

   - Lancez un cluster kind :  

     ```bash
     kind create cluster
     ```

2. **Écrire un Deployment et un Service**  
   - `deployment.yaml` :

     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: demo-app
     spec:
       replicas: 1
       selector:
         matchLabels:
           app: demo-app
       template:
         metadata:
           labels:
             app: demo-app
         spec:
           containers:
             - name: demo-app
               image: votreuser/demo-app:latest
               ports:
                 - containerPort: 8080
     ```

    `service.yaml` :
        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: demo-app-service
        spec:
          type: NodePort
          selector:
            app: demo-app
          ports:
            - protocol: TCP
              port: 80
              targetPort: 8080
        ```

    Ajustez `containerPort`/`targetPort` selon le code Java.

3. **Déploiement sur le cluster**  
   - Dans le terminal :  

     ```bash
     kubectl apply -f deployment.yaml
     kubectl apply -f service.yaml
     kubectl get pods
     kubectl get svc
     ```

   - Vérifiez que le pod est en état `Running`.  
   - Avec `kind`, le `NodePort` n’est accessible que par `kubectl port-forward` ou config spécifique.  
   - Exemple :  

     ```bash
     kubectl port-forward deployment/demo-app 8080:8080
     ```

   - Allez sur <http://localhost:8080> pour tester.

4. **Infrastructure as Code (Terraform) - optionnel**  
   - [Installer Terraform](https://developer.hashicorp.com/terraform/downloads).  
   - Exemple d’un fichier `main.tf` minimal pour Kubernetes :  

     ```hcl
     provider "kubernetes" {
       config_path = "~/.kube/config"
     }

     resource "kubernetes_deployment" "example" {
       metadata {
         name = "demo-app-terraform"
         labels = {
           App = "demo-app"
         }
       }
       spec {
         replicas = 1
         selector {
           match_labels = {
             App = "demo-app"
           }
         }
         template {
           metadata {
             labels = {
               App = "demo-app"
             }
           }
           spec {
             container {
               image = "votreuser/demo-app:latest"
               name  = "demo-app"
             }
           }
         }
       }
     }
     ```

- Lancer :  

     ```bash
     terraform init
     terraform plan
     terraform apply
     ```

**À la fin de cette séance**, vous savez déployer votre container Java sur Kubernetes local, et vous avez un aperçu d’IaC.

---

## TD DevOps 5 (2h)  

### Thème : Sécurité, Monitoring & Pipeline Complet

**Objectifs :**  

- Intégrer un scan de vulnérabilités.  
- Ajouter un composant de monitoring/logging.  
- Vérifier l’ensemble de la chaîne CI/CD.

---

### Étapes détaillées Sécurité

1. **Scan de vulnérabilités (Trivy ou Snyk)**  
   - [Installer Trivy](https://trivy.dev/docs/latest/getting-started/installation/) ou [Snyk CLI](https://docs.snyk.io/snyk-cli/install-the-snyk-cli).  
   - Exemple Trivy :  

     ```bash
     trivy image votreuser/demo-app:latest
     ```

   - Intégrez-le dans GitHub Actions :

     ```yaml
     - name: Scan Docker Image with Trivy
       uses: aquasecurity/trivy-action@master
       with:
         image-ref: votreuser/demo-app:latest
     ```

   - Configurez l’étape pour échouer si des vulnérabilités sont critiques.

2. **Monitoring**  
   - Dans l’application Java, vous pouvez ajouter [Micrometer](https://micrometer.io/) + [Prometheus Registry](https://micrometer.io/docs/registry/prometheus) pour exposer des métriques sur un endpoint `/actuator/prometheus` (Spring Boot) ou `/metrics`.  
   - Installer [Prometheus](https://prometheus.io/docs/introduction/first_steps/) localement :  

     ```bash
     docker run -p 9090:9090 prom/prometheus
     ```

   - Configurez `prometheus.yml` pour scrapper votre pod sur `http://demo-app-service:80/metrics`.  
   - Lancez `kubectl port-forward ...` si besoin.

3. **Pipeline complet**  
   - Modifiez le code, push => le pipeline :  
     1. Compile le code Java & exécute les tests (mvn).  
     2. Lance Checkstyle pour la qualité du code.  
     3. Construit l’image Docker et la pousse sur le registre.  
     4. Scanne l’image pour vulnérabilités (Trivy).  
     5. Met à jour Kubernetes (optionnellement via `kubectl set image` ou Terraform).  
   - Surveillez la CI sur GitHub Actions, vérifiez la réussite de chaque étape.

---

**À la fin de cette séance**, vous avez un pipeline *complet*, incluant sécurité, conteneurisation, orchestration, monitoring.

---

## Conclusion & Évaluation

- **Évaluation continue** :
  - QCM au début de certains cours pour valider les acquis théoriques.  
  - Suivi des TD (qualité des commits, pipelines fonctionnels, Power Apps complètes).
