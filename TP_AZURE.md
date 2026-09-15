# TP - Déploiement d’une application Full Stack Java sur Azure avec Terraform et Ansible

## Objectif

L’objectif de ce **TP** est de déployer une application **Full Stack** composée de :

- un **frontend** ;
- un **backend Java / Spring Boot** ;
- une **base de données MySQL** ;
- une infrastructure **Azure** créée avec **Terraform** ;
- une configuration et un déploiement automatisés avec **Ansible** ;
- un **rapport avec captures d’écran** permettant de justifier toutes les étapes réalisées.

---

## Dépôt du TP sur Google Classroom

> **Lien de dépôt : [Déposer votre TP sur Google Classroom](https://classroom.google.com/)**

Le lien ci-dessus permet d’accéder à Google Classroom.
Le lien direct du devoir pourra être ajouté ici par l’enseignant.

---

# 1. Application à utiliser

Vous devez choisir une application **Java Full Stack** disponible sur **GitHub**.

Application proposée :

**jhordyess/dockerized-spring-react-mysql**

**GitHub** :

https://github.com/jhordyess/dockerized-spring-react-mysql

**Cette application contient notamment** :

- un frontend **React** ;
- un backend **Java / Spring Boot** ;
- une **API REST** ;
- une base de données **MySQL**.

**Vous pouvez utiliser une autre application Java Full Stack à condition qu’elle contienne au minimum** :

- un frontend ;
- un backend **Java** ;
- une base de données relationnelle ;
- des fonctionnalités permettant de lire et écrire des données.

---

# 2. Architecture attendue

**L’architecture cible est la suivante** :

```text
Utilisateur
    |
    | HTTPS
    v
Azure App Service
    |
    |-- Frontend React
    |
    |-- Backend Spring Boot
              |
              | JDBC
              v
      Azure Database for MySQL
      Flexible Server
```

Dans cette architecture, le frontend **React** est compilé puis intégré dans les ressources statiques de **Spring Boot**.

**L’application finale pourra donc être exposée avec une seule URL** :

```text
https://nom-application.azurewebsites.net
```

**L’API REST pourra être accessible par exemple avec** :

```text
https://nom-application.azurewebsites.net/api/users
```

---

# 3. Répartition des rôles

## Terraform

**Terraform doit être utilisé pour créer l’infrastructure Azure** :

- **Resource Group** ;
- **App Service Plan** ;
- **Azure App Service** ;
- **Azure Database for MySQL Flexible Server** ;
- **base de données** ;
- **règles réseau nécessaires** ;
- **variables** ;
- **outputs**.

## Ansible

**Ansible doit être utilisé pour** :

- récupérer l’application depuis **GitHub** ;
- installer les dépendances ;
- compiler le frontend ;
- compiler le backend **Java** ;
- configurer l’**App Service** ;
- configurer les variables d’environnement ;
- déployer le fichier **JAR** ;
- vérifier que l’application répond correctement.

---

# 4. Structure du projet

**Organisation recommandée** :

```text
tp-fullstack-azure/
|
|-- terraform/
|   |-- providers.tf
|   |-- variables.tf
|   |-- main.tf
|   |-- outputs.tf
|   `-- terraform.tfvars
|
|-- ansible/
|   |-- inventory.ini
|   |-- deploy.yml
|   `-- vars.yml
|
|-- application/
|
|-- screenshots/
|
|-- README.md
|
`-- rapport.pdf
```

---

# 5. Prérequis

**Vous devez disposer des outils suivants** :

- **Azure CLI** ;
- **Terraform** ;
- **Ansible** ;
- **Git** ;
- **Java** ;
- **Maven** ;
- **Node.js** ;
- **npm** ;
- **un abonnement Microsoft Azure**.

**Vérifiez les installations** :

```bash
az version
terraform version
ansible --version
git --version
java --version
mvn --version
node --version
npm --version
```

---

# 6. Connexion à Azure

**Connectez-vous** :

```bash
az login
```

**Vérifiez l’abonnement utilisé** :

```bash
az account show
```

**Si vous disposez de plusieurs abonnements** :

```bash
az account list --output table
```

**Puis sélectionnez celui à utiliser** :

```bash
az account set --subscription "ID_OU_NOM_ABONNEMENT"
```

---

# 7. Partie Terraform

**Créez le dossier** :

```bash
mkdir -p tp-fullstack-azure/terraform
cd tp-fullstack-azure/terraform
```

---

## 7.1 providers.tf

**Créez** :

```text
providers.tf
```

**Exemple** :

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

---

# 8. Variables Terraform

**Créez** :

```text
variables.tf
```

**Exemple** :

```hcl
variable "location" {
  description = "Région Azure"
  type        = string
  default     = "West Europe"
}

variable "db_admin" {
  description = "Administrateur MySQL"
  type        = string
  default     = "adminazure"
}

variable "db_password" {
  description = "Mot de passe MySQL"
  type        = string
  sensitive   = true
}
```

Le mot de passe ne doit pas apparaître directement dans le dépôt **Git**.

**Vous pouvez par exemple utiliser** :

```bash
export TF_VAR_db_password='VotreMotDePasseTresFort!'
```

---

# 9. Resource Group

**Dans** :

```text
main.tf
```

**ajoutez** :

```hcl
resource "azurerm_resource_group" "rg" {
  name     = "rg-fullstack-tp"
  location = var.location

  tags = {
    project    = "fullstack-java"
    managed_by = "terraform"
  }
}
```

---

# 10. App Service Plan

**Ajoutez** :

```hcl
resource "azurerm_service_plan" "plan" {
  name                = "plan-fullstack-tp"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location

  os_type  = "Linux"
  sku_name = "B1"
}
```

> Adaptez le **SKU** en fonction des limitations de votre abonnement **Azure**.

---

# 11. Azure App Service

**Ajoutez** :

```hcl
resource "azurerm_linux_web_app" "app" {
  name                = "tp-java-fullstack-12345"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  service_plan_id     = azurerm_service_plan.plan.id

  https_only = true

  site_config {
    application_stack {
      java_version        = "17"
      java_server         = "JAVA"
      java_server_version = "17"
    }
  }

  tags = {
    managed_by = "terraform"
  }
}
```

Le nom de l’App Service doit être globalement unique.

**Modifiez donc** :

```text
tp-java-fullstack-12345
```

avec un nom qui vous est propre.

---

# 12. Azure Database for MySQL

**Ajoutez** :

```hcl
resource "azurerm_mysql_flexible_server" "mysql" {
  name                = "mysql-fullstack-tp-12345"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location

  administrator_login    = var.db_admin
  administrator_password = var.db_password

  sku_name = "B_Standard_B1ms"

  version = "8.0.21"
}
```

Le nom du serveur **MySQL** doit également être unique.

---

# 13. Création de la base

**Ajoutez** :

```hcl
resource "azurerm_mysql_flexible_database" "database" {
  name                = "fulldb"
  resource_group_name = azurerm_resource_group.rg.name
  server_name         = azurerm_mysql_flexible_server.mysql.name

  charset   = "utf8"
  collation = "utf8_unicode_ci"
}
```

---

# 14. Règle réseau

**Pour ce TP, vous pouvez autoriser les services Azure à contacter MySQL** :

```hcl
resource "azurerm_mysql_flexible_server_firewall_rule" "azure" {
  name                = "AllowAzureServices"
  resource_group_name = azurerm_resource_group.rg.name
  server_name         = azurerm_mysql_flexible_server.mysql.name

  start_ip_address = "0.0.0.0"
  end_ip_address   = "0.0.0.0"
}
```

Dans le rapport, vous devez expliquer pourquoi cette configuration serait à renforcer pour une application de production.

---

# 15. Outputs Terraform

**Créez** :

```text
outputs.tf
```

**Puis ajoutez** :

```hcl
output "resource_group" {
  value = azurerm_resource_group.rg.name
}

output "app_name" {
  value = azurerm_linux_web_app.app.name
}

output "app_url" {
  value = "https://${azurerm_linux_web_app.app.default_hostname}"
}

output "mysql_host" {
  value = azurerm_mysql_flexible_server.mysql.fqdn
}

output "database_name" {
  value = azurerm_mysql_flexible_database.database.name
}
```

---

# 16. Initialisation Terraform

**Exécutez** :

```bash
terraform init
```

**Puis** :

```bash
terraform fmt
```

**Vérifiez la configuration** :

```bash
terraform validate
```

---

# 17. Terraform Plan

**Exécutez** :

```bash
terraform plan
```

Analysez les ressources qui seront créées.

Vous devez comprendre les informations affichées avant de continuer.

---

# 18. Création de l’infrastructure

**Exécutez** :

```bash
terraform apply
```

**Validez avec** :

```text
yes
```

**À la fin, vous devez obtenir un message similaire à** :

```text
Apply complete!
```

**Affichez les informations utiles** :

```bash
terraform output
```

---

# 19. Vérification dans Azure

**Depuis le portail Azure, vérifiez la présence de** :

```text
Resource Group
|
|-- App Service Plan
|
|-- App Service
|
`-- MySQL Flexible Server
```

**Vérifiez également que la base** :

```text
fulldb
```

a bien été créée.

---

# 20. Partie Ansible

**Créez** :

```bash
cd ..
mkdir ansible
cd ansible
```

**Installez si nécessaire Ansible** :

```bash
sudo apt update
sudo apt install ansible
```

**Installez la collection Azure** :

```bash
ansible-galaxy collection install azure.azcollection
```

---

# 21. Inventory Ansible

**Créez** :

```text
inventory.ini
```

**Contenu** :

```ini
[local]
localhost ansible_connection=local
```

**Vérifiez** :

```bash
ansible all -i inventory.ini -m ping
```

**Résultat attendu** :

```text
localhost | SUCCESS
```

---

# 22. Playbook Ansible

**Créez** :

```text
deploy.yml
```

Le playbook devra automatiser les opérations suivantes.

---

## 22.1 Cloner l’application

**Exemple** :

```yaml
---
- name: Déployer l'application Full Stack sur Azure
  hosts: local
  connection: local

  vars:
    resource_group: "rg-fullstack-tp"
    app_name: "tp-java-fullstack-12345"
    project_dir: "../application"

  tasks:

    - name: Cloner l'application GitHub
      ansible.builtin.git:
        repo: "https://github.com/jhordyess/dockerized-spring-react-mysql.git"
        dest: "{{ project_dir }}"
        version: main
```

**Adaptez éventuellement** :

```text
version: main
```

en fonction de la branche utilisée par le dépôt.

---

# 23. Build du frontend

**Ajoutez** :

```yaml
    - name: Installer les dépendances frontend
      ansible.builtin.command:
        cmd: npm install
        chdir: "{{ project_dir }}/frontend"

    - name: Compiler le frontend
      ansible.builtin.command:
        cmd: npm run build
        chdir: "{{ project_dir }}/frontend"
```

Vérifiez le répertoire généré par votre application.

**Selon le framework, il pourra s’appeler** :

```text
dist/
```

**ou** :

```text
build/
```

---

# 24. Intégration du frontend dans Spring Boot

**Créez le répertoire statique** :

```yaml
    - name: Créer le répertoire static
      ansible.builtin.file:
        path: "{{ project_dir }}/backend/src/main/resources/static"
        state: directory
```

**Puis copiez le frontend compilé** :

```yaml
    - name: Copier le frontend dans Spring Boot
      ansible.builtin.copy:
        src: "{{ project_dir }}/frontend/dist/"
        dest: "{{ project_dir }}/backend/src/main/resources/static/"
```

**Adaptez le chemin si votre application génère un répertoire** :

```text
build/
```

**à la place de** :

```text
dist/
```

---

# 25. Configuration Spring Boot

La configuration **MySQL** ne doit pas contenir de mot de passe codé en dur.

**Utilisez par exemple** :

```properties
spring.datasource.url=${SPRING_DATASOURCE_URL}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD}

spring.jpa.hibernate.ddl-auto=update
```

Les valeurs seront fournies à l’application via **Azure App Service**.

---

# 26. Build du backend

**Ajoutez** :

```yaml
    - name: Compiler l'application Spring Boot
      ansible.builtin.command:
        cmd: mvn clean package -DskipTests
        chdir: "{{ project_dir }}/backend"
```

À la fin, vous devez obtenir un fichier :

```text
backend/target/*.jar
```

---

# 27. Variables Azure App Service

Les informations **MySQL** devront être configurées dans **App Service**.

**Exemple de valeurs** :

```text
SPRING_DATASOURCE_URL
SPRING_DATASOURCE_USERNAME
SPRING_DATASOURCE_PASSWORD
SPRING_JPA_HIBERNATE_DDL_AUTO
```

Vous pouvez les définir avec **Ansible**.

**Exemple** :

```yaml
    - name: Configurer App Service
      azure.azcollection.azure_rm_webapp:
        resource_group: "{{ resource_group }}"
        name: "{{ app_name }}"

        app_settings:
          SPRING_DATASOURCE_URL: "jdbc:mysql://mysql-fullstack-tp-12345.mysql.database.azure.com:3306/fulldb?useSSL=true"
          SPRING_DATASOURCE_USERNAME: "{{ db_username }}"
          SPRING_DATASOURCE_PASSWORD: "{{ db_password }}"
          SPRING_JPA_HIBERNATE_DDL_AUTO: "update"

        purge_app_settings: false
```

Ne stockez pas votre vrai mot de passe directement dans **Git**.

---

# 28. Déploiement du JAR

Après compilation, déployez le fichier **JAR** sur **Azure App Service**.

**Exemple** :

```yaml
    - name: Déployer le JAR sur Azure
      ansible.builtin.command:
        cmd: >
          az webapp deploy
          --resource-group {{ resource_group }}
          --name {{ app_name }}
          --src-path {{ project_dir }}/backend/target/application.jar
          --type jar
```

**Vous devez adapter** :

```text
application.jar
```

au nom réellement généré par **Maven**.

---

# 29. Vérification automatique

**Ajoutez une vérification HTTP** :

```yaml
    - name: Vérifier que l'application répond
      ansible.builtin.uri:
        url: "https://{{ app_name }}.azurewebsites.net"
        method: GET
        status_code: 200
      register: app_test
      retries: 10
      delay: 10
      until: app_test.status == 200
```

**Puis affichez l’URL** :

```yaml
    - name: Afficher l'URL
      ansible.builtin.debug:
        msg: "Application disponible sur https://{{ app_name }}.azurewebsites.net"
```

---

# 30. Exécution Ansible

**Exécutez** :

```bash
ansible-playbook -i inventory.ini deploy.yml
```

**Le résultat final doit contenir** :

```text
failed=0
```

**Exemple** :

```text
PLAY RECAP
localhost : ok=8 changed=6 unreachable=0 failed=0
```

---

# 31. Tests fonctionnels

Vous devez vérifier que toute la chaîne fonctionne.

## Test 1 - Frontend

**Ouvrez** :

```text
https://nom-application.azurewebsites.net
```

Le frontend doit s’afficher.

---

## Test 2 - Backend

Utilisez une fonctionnalité nécessitant le backend.

**Par exemple** :

```text
Nom : Alice
Email : alice@test.fr
```

Enregistrez les données.

---

## Test 3 - Base de données

Rechargez l’application.

La donnée enregistrée doit toujours être présente.

**Cela permet de valider la chaîne** :

```text
Frontend
   |
   v
Spring Boot
   |
   v
API REST
   |
   v
MySQL Azure
```

---

# 32. Vérifications supplémentaires

**Vous devez également vérifier** :

```bash
az webapp show \
  --resource-group rg-fullstack-tp \
  --name NOM_APP
```

**Puis** :

```bash
az webapp config appsettings list \
  --resource-group rg-fullstack-tp \
  --name NOM_APP
```

Attention à ne pas faire apparaître un mot de passe dans vos captures d’écran.

---

# 33. Rapport demandé

Vous devez remettre un rapport **PDF** expliquant votre travail.

**Le rapport devra contenir au minimum les sections suivantes** :

```text
1. Introduction

2. Présentation de l'application GitHub

3. Architecture retenue

4. Infrastructure Terraform

5. Création des ressources Azure

6. Présentation du playbook Ansible

7. Build du frontend

8. Build du backend

9. Déploiement sur Azure

10. Configuration de MySQL

11. Tests fonctionnels

12. Sécurité

13. Difficultés rencontrées

14. Solutions apportées

15. Conclusion
```

---

# 34. Captures d’écran obligatoires

Votre rapport doit contenir des captures d’écran permettant de prouver le travail réalisé.

**Au minimum** :

### Capture 1

Dépôt **GitHub** sélectionné.

### Capture 2

**Structure du projet avec** :

```text
frontend/
backend/
```

### Capture 3

**Commande** :

```bash
terraform init
```

**et** :

```bash
terraform validate
```

### Capture 4

**Commande** :

```bash
terraform plan
```

### Capture 5

**Résultat** :

```text
Apply complete!
```

### Capture 6

**Resource Group** dans le portail **Azure**.

### Capture 7

**Azure App Service**.

### Capture 8

**Azure Database** for **MySQL Flexible Server**.

### Capture 9

Exécution du playbook **Ansible**.

**Le résultat doit montrer** :

```text
failed=0
```

### Capture 10

Application accessible dans le navigateur.

### Capture 11

Ajout d’une donnée dans l’application.

### Capture 12

Donnée toujours présente après actualisation de l’application.

---

# 35. Présentation des captures

**Chaque capture doit** :

- être lisible ;
- être numérotée ;
- comporter une légende ;
- être expliquée.

**Exemple** :

```text
Figure 4 - Résultat de la commande terraform plan avant la création de l'infrastructure Azure.
```

Une capture seule sans explication ne permet pas de justifier correctement le travail réalisé.

---

# 36. Sécurité

**Dans votre rapport, expliquez au minimum les points suivants** :

- pourquoi un mot de passe ne doit pas être stocké directement dans **Git** ;
- pourquoi les variables **Terraform** sensibles doivent être protégées ;
- pourquoi les variables d’environnement sont préférables aux valeurs codées en dur ;
- pourquoi l’ouverture réseau de **MySQL** utilisée dans ce **TP** doit être renforcée en production ;
- pourquoi HTTPS doit être activé ;
- pourquoi les secrets ne doivent pas apparaître dans les captures d’écran.

---

# 37. Bonus

**Les améliorations suivantes peuvent être réalisées en bonus** :

- **Ansible Vault** ;
- **Azure Key Vault** ;
- intégration réseau privée ;
- **VNet** Integration ;
- **Private Endpoint** ;
- **CI/CD** avec **GitHub Actions** ;
- tests automatiques ;
- **HTTPS** personnalisé ;
- nom de domaine personnalisé ;
- monitoring avec **Application Insights** ;
- plusieurs environnements **Terraform** ;
- variables **Terraform** par environnement.

---

# 38. Nettoyage des ressources Azure

Après validation du **TP**, détruisez les ressources pour éviter de consommer inutilement votre crédit **Azure**.

**Placez-vous dans** :

```bash
cd terraform
```

**Puis** :

```bash
terraform destroy
```

**Confirmez avec** :

```text
yes
```

Vous devez vérifier que les ressources ont bien disparu du portail **Azure**.

---

# 39. Livrables

**Vous devez déposer** :

```text
tp-fullstack-azure/
|
|-- terraform/
|
|-- ansible/
|
|-- README.md
|
`-- rapport.pdf
```

**Ne fournissez pas** :

- le répertoire `.terraform/` ;
- les fichiers contenant des mots de passe ;
- `node_modules/` ;
- les fichiers temporaires ;
- les secrets **Azure**.

---

# 40. Fichier .gitignore recommandé

**Créez** :

```text
.gitignore
```

**Avec** :

```gitignore
.terraform/
*.tfstate
*.tfstate.*
.terraform.lock.hcl

terraform.tfvars

node_modules/

target/

.env
.env.*

*.log
```

**Attention** : selon votre contexte, vous pouvez décider de conserver `.terraform.lock.hcl` dans votre dépôt afin de verrouiller les versions utilisées. Expliquez votre choix.

---

# 41. Critères d’évaluation

| Critère | Points |
|---|---:|
| Architecture et compréhension du projet | 2 |
| Infrastructure **Terraform** | 4 |
| Qualité du code **Terraform** | 2 |
| **Playbook Ansible** | 4 |
| Déploiement de l’application | 2 |
| Connexion à la base **MySQL** | 2 |
| Tests fonctionnels | 1 |
| Sécurité | 1 |
| Rapport et captures d’écran | 2 |
| **Total** | **20** |

---

# 42. Résultat attendu

**À la fin du TP, l’architecture suivante doit être opérationnelle** :

```text
GitHub
   |
   v
Ansible
   |
   |-- Clone
   |-- npm install
   |-- npm run build
   |-- mvn package
   |
   v
Azure App Service
   |
   |-- React
   |-- Spring Boot
   |
   v
Azure Database for MySQL
```

**L’infrastructure doit pouvoir être recréée avec** :

```bash
terraform apply
```

**et l’application doit pouvoir être redéployée avec** :

```bash
ansible-playbook -i inventory.ini deploy.yml
```

---

# 43. Dépôt final

**Une fois le travail terminé** :

1. vérifiez que le rapport **PDF** est présent ;
2. vérifiez que les captures sont lisibles ;
3. vérifiez qu’aucun secret n’est présent dans le dépôt ;
4. compressez le projet si demandé ;
5. déposez le travail sur Google Classroom.

> **Google Classroom : [Accéder au dépôt du TP](https://classroom.google.com/c/ODg1MTMyMTIxMTMz?cjc=6ijgykiq)**

**Date limite : 15/09/23h59**
