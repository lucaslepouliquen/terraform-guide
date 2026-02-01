# Résumé Complet du Cours Terraform

## 0. Certification Terraform Associate

### Détails de la Certification

**Format de l'examen:**
- Durée: 60 minutes
- Prix: 70.50 USD
- Validité: 2 ans
- Type: Questions à choix multiples (QCM)
- Nombre de questions: ~57 questions
- Format: Multiple choices, multiple options, vrai/faux
- Passation: En ligne avec surveillance (proctored)

**Exigences techniques:**
- Navigateur PSI Secure Browser requis
- Webcam, haut-parleurs et microphone activés
- Pas de VM autorisées
- Pas de moniteurs supplémentaires ni écouteurs
- Environnement: calme, bien éclairé et propre

**Inscription:**
- URL: https://www.hashicorp.com/certification/terraform-associate

---

## 1. Introduction à l'Infrastructure as Code (IaC)

### Défis de l'infrastructure traditionnelle
- Déploiement lent et coûteux
- Automatisation limitée
- Erreurs humaines
- Incohérences
- Gaspillage de ressources

### Infrastructure as Code
- Code déclaratif pour provisionner l'infrastructure
- Versioning et contrôle
- Automatisation complète
- Reproductibilité

### Types d'outils IaC

**Configuration Management:**
- Ansible, Puppet, Chef
- Installation et gestion de logiciels
- Structure standardisée
- Idempotent

**Server Templating:**
- Docker, Packer, Vagrant
- Images pré-configurées (VM ou conteneurs)
- Infrastructure immuable

**Provisioning Tools:**
- Terraform, CloudFormation
- Déploiement d'infrastructures complètes
- Multi-providers

---

## 2. Pourquoi Terraform?

### Avantages
- Support multi-providers (AWS, Azure, GCP, VMware, etc.)
- Plus de 1000+ providers disponibles
- HashiCorp Configuration Language (HCL) déclaratif
- State management intégré
- Import de ressources existantes

### Ecosystem
- Terraform Cloud
- Terraform Enterprise
- Registry public avec modules réutilisables

---

## 3. Installation

```bash
# Linux
wget https://releases.hashicorp.com/terraform/1.14.3/terraform_1.14.3_linux_amd64.zip
unzip terraform_1.14.3_linux_amd64.zip
mv terraform /usr/local/bin
terraform version
```

---

## 4. HCL Basics

### Structure d'un fichier Terraform

```hcl
resource "provider_type" "resource_name" {
  argument1 = "value1"
  argument2 = "value2"
}
```

**Exemple:**
```hcl
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content  = "We love pets!"
}

resource "aws_instance" "webserver" {
  ami           = "ami-0c2f25c1f66a1ff4d"
  instance_type = "t2.micro"
}
```

### Workflow de base

1. **terraform init** - Initialise le répertoire, télécharge les providers
2. **terraform plan** - Affiche les changements prévus
3. **terraform apply** - Applique les changements
4. **terraform destroy** - Détruit les ressources

---

## 5. Variables

### Déclaration

```hcl
# variables.tf
variable "filename" {
  default     = "/root/pets.txt"
  type        = string
  description = "Path to the file"
}
```

### Types de variables

**Types simples:**
- `string` - Chaîne de caractères
- `number` - Nombre
- `bool` - Boolean (true/false)

**Types complexes:**
- `list` - Liste ordonnée: `["value1", "value2"]`
- `map` - Dictionnaire: `{key1 = "value1", key2 = "value2"}`
- `set` - Liste sans doublons
- `object` - Structure complexe avec types spécifiques
- `tuple` - Liste avec types spécifiques pour chaque élément

```hcl
variable "map_exemple"
{
  type = object({
    name = string
    age = number
  })
  default = {
  name = "John"
  age  = 52
  }
}
```

### Utilisation

```hcl
resource "local_file" "pet" {
  filename = var.filename
  content  = var.content
}
```

**Noms réservés:** On ne peut pas utiliser les noms suivants pour les variables: `source`, `version`, `providers`, `count`, `for_each`, `lifecycle`, `depends_on` et `locals`.

### Méthodes d'assignation (par ordre de précédence)

1. Variables d'environnement: `export TF_VAR_filename="/root/pets.txt"`
2. `terraform.tfvars` ou `*.auto.tfvars`
3. Fichiers variables: `-var-file="custom.tfvars"`
4. Ligne de commande: `-var "filename=/root/pets.txt"`

---

## 6. Output Variables

```hcl
output "pet-name" {
  value       = random_pet.my-pet.id
  description = "ID of the random pet"
}
```

Commandes:
- `terraform output` - Affiche tous les outputs
- `terraform output pet-name` - Affiche un output spécifique

---

Les output variables servent à exposer des informations après l'application de ton infrastructure. Elles ont plusieurs usages : 
Afficher des informations importantes :

```hcl
output "instance_ip" {
  value = aws_instance.web.public_ip
  description = "L'IP publique du serveur web"
}
```

Après `terraform apply`, tu verras :
```
Outputs:
instance_ip = "52.12.34.56"
```

Partager des données entre modules :
```hcl
# Module "network"
output "vpc_id" {
  value = aws_vpc.main.id
}

# Module "app" qui utilise "network"
module "network" {
  source = "./modules/network"
}

resource "aws_instance" "app" {
  vpc_id = module.network.vpc_id  # Utilise l'output du module
}
```

## 7. Resource Attributes & Dependencies

### Implicit Dependency

```hcl
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content  = "My pet is ${random_pet.my-pet.id}"
}

resource "random_pet" "my-pet" {
  prefix = "Mrs"
}
```

### Explicit Dependency

```hcl
resource "local_file" "pet" {
  filename   = "/root/pets.txt"
  content    = "I love pets"
  depends_on = [random_pet.my-pet]
}
```

---

## 8. Terraform State

### Caractéristiques
- Fichier `terraform.tfstate` en JSON
- Mapping entre configuration et infrastructure réelle
- Tracking des métadonnées
- Amélioration des performances
- Collaboration via remote backends

### Commandes State

```bash
terraform state list          # Liste les ressources
terraform state show          # Détails d'une ressource
terraform state mv            # Renommer une ressource
terraform state rm            # Retirer du state
terraform state pull          # Récupérer le state distant
```

---

## 9. Lifecycle Rules

```hcl
resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content  = "We love pets!"
  
  lifecycle {
    create_before_destroy = true   # Créer avant de détruire
    prevent_destroy      = true    # Empêcher la destruction
    ignore_changes       = [tags]  # Ignorer les changements sur tags
  }
}
```

---

## 10. Data Sources

Récupère des informations depuis ton infrastructure existante plutôt que de créer de nouvelles ressources.
Resource (resource) : crée ou gère une ressource (VM, réseau, etc.)
Data source (data) : lit des informations sur une ressource existante

```hcl
data "local_file" "dog" {
  filename = "/root/dog.txt"
}

resource "local_file" "pet" {
  filename = "/root/pets.txt"
  content  = data.local_file.dog.content
}
```

---

## 11. Meta-Arguments

### Count

```hcl
resource "local_file" "pet" {
  filename = var.filename[count.index]
  count    = length(var.filename)
}

variable "filename" {
  default = [
    "/root/pets.txt",
    "/root/dogs.txt",
    "/root/cats.txt"
  ]
}
```

### For-Each

```hcl
resource "local_file" "pet" {
  filename = each.value
  for_each = toset(var.filename)
}
```

**Note:** `for_each` est préférable à `count` car il évite les problèmes lors de suppressions d'éléments au milieu de la liste.

---

## 12. Version Constraints

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.0.0"  # Version 2.0.x uniquement
    }
  }
}
```

**Opérateurs:**
- `=` - Version exacte
- `!=` - Exclure une version
- `>`, `<`, `>=`, `<=` - Comparaisons
- `~>` - Versions compatibles (patch)

---

## 13. AWS IAM avec Terraform

### Configuration du Provider

```hcl
provider "aws" {
  region     = "us-west-2"
  # Méthodes d'authentification (par ordre de précédence):
  # 1. access_key/secret_key dans le provider
  # 2. Variables d'environnement AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY
  # 3. Fichier ~/.aws/credentials
}
```

### Création d'utilisateur IAM

```hcl
resource "aws_iam_user" "admin-user" {
  name = "lucy"
  tags = {
    Description = "Technical Team Leader"
  }
}
```

### Policy avec Heredoc

```hcl
resource "aws_iam_policy" "adminUser" {
  name   = "AdminUsers"
  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
EOF
}
```

### Policy depuis fichier

```hcl
resource "aws_iam_policy" "adminUser" {
  name   = "AdminUsers"
  policy = file("admin-policy.json")
}
```

### Attachment de Policy

```hcl
resource "aws_iam_user_policy_attachment" "lucy-admin-access" {
  user       = aws_iam_user.admin-user.name
  policy_arn = aws_iam_policy.adminUser.arn
}
```

---

## 14. AWS S3

### Création de bucket

```hcl
resource "aws_s3_bucket" "finance" {
  bucket = "finance-21092020"
  tags = {
    Description = "Finance and Payroll"
  }
}
```

### Upload d'objets

```hcl
resource "aws_s3_bucket_object" "finance-2020" {
  content = "/root/finance/finance-2020.doc"
  key     = "finance-2020.doc"
  bucket  = aws_s3_bucket.finance.id
}
```

### Bucket Policy

```hcl
resource "aws_s3_bucket_policy" "finance-policy" {
  bucket = aws_s3_bucket.finance.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action    = "*"
        Effect    = "Allow"
        Resource  = "${aws_s3_bucket.finance.arn}/*"
        Principal = {
          AWS = data.aws_iam_group.finance-data.arn
        }
      }
    ]
  })
}
```

---

## 15. DynamoDB

```hcl
resource "aws_dynamodb_table" "cars" {
  name         = "cars"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "VIN"
  
  attribute {
    name = "VIN"
    type = "S"
  }
}
```

---

## 16. Remote State Backend

### Configuration S3 + DynamoDB

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "finance/terraform.tfstate"
    region         = "us-west-1"
    dynamodb_table = "state-locking"
  }
}
```

**Avantages:**
- State partagé pour collaboration
- Verrouillage automatique (state locking)
- Sécurité accrue
- Chargement/upload automatique

### State Locking

Le state locking est un mécanisme qui empêche plusieurs utilisateurs de modifier simultanément l'infrastructure. De nombreux backends supportent cette fonctionnalité:
- AWS S3 avec DynamoDB
- Google Cloud Storage
- HashiCorp Consul
- Terraform Cloud

**Processus:**
1. Terraform acquiert un verrou avant d'écrire dans le state
2. Le verrou empêche d'autres opérations concurrentes
3. Le verrou est libéré après l'opération

```bash
# Message typique lors de l'acquisition du verrou
Acquiring state lock. This may take a few moments...
# Message lors de la libération
Releasing state lock. This may take a few moments.
```

### Migration vers remote backend

```bash
terraform init  # Terraform propose de migrer le state existant
```

Exemple de dialogue de migration:
```
Do you want to copy existing state to the new backend?
Pre-existing state was found while migrating the previous "local" backend 
to the newly configured "s3" backend. No existing state was found in the 
newly configured "s3" backend. Do you want to copy this state to the new 
"s3" backend? Enter "yes" to copy and "no" to start with an empty state.

Enter a value: yes
```

---

## 17. AWS EC2

### Instance de base

```hcl
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"
  
  tags = {
    Name        = "webserver"
    Description = "An Nginx WebServer"
  }
  
  # User data pour configuration initiale
  user_data = <<-EOF
              #!/bin/bash
              sudo apt update
              sudo apt install nginx -y
              systemctl enable nginx
              systemctl start nginx
              EOF
}
```

### SSH Key Pair

```hcl
resource "aws_key_pair" "web" {
  public_key = file("/root/.ssh/web.pub")
}

resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"
  key_name      = aws_key_pair.web.id
}
```

### Security Group

```hcl
resource "aws_security_group" "ssh-access" {
  name        = "ssh-access"
  description = "Allow SSH from Internet"
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "webserver" {
  ami                    = "ami-0edab43b6fa892279"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.ssh-access.id]
  key_name               = aws_key_pair.web.id
}
```

---

## 18. Provisioners

Les provisioners permettent d'exécuter des scripts sur les ressources créées.

### Remote-exec

```hcl
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"
  
  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install nginx -y",
      "sudo systemctl start nginx"
    ]
    
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("/root/.ssh/web")
      host        = self.public_ip
    }
  }
}
```

### Local-exec

```hcl
resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = "t2.micro"
  
  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> /tmp/ips.txt"
  }
}
```

**Note:** Les provisioners doivent être utilisés en dernier recours. Préférer user_data, configuration management ou AMI pré-configurées.

---

## 19. Terraform Taint

Marquer une ressource pour recréation:

```bash
terraform taint aws_instance.webserver
terraform apply  # Recréera l'instance
```

Pour annuler:
```bash
terraform untaint aws_instance.webserver
```

---

## 20. Debugging

### Niveaux de log

```bash
export TF_LOG=TRACE  # TRACE, DEBUG, INFO, WARN, ERROR
export TF_LOG_PATH=/tmp/terraform.log
terraform apply
```

### Format de sortie

```bash
terraform show -json  # Format JSON
```

---

## 21. Terraform Import

Importer des ressources existantes dans le state:

```bash
terraform import aws_instance.webserver i-0a1b2c3d4e5f
```

Configuration requise:
```hcl
resource "aws_instance" "webserver" {
  # Configuration doit correspondre à la ressource existante
}
```

---

## 22. Modules

### Structure d'un module

```
modules/
└── payroll-app/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── README.md
```

### Création d'un module

```hcl
# modules/payroll-app/main.tf
resource "aws_instance" "app_server" {
  ami           = var.ami
  instance_type = var.instance_type
  tags = {
    Name = "${var.app_region}-app-server"
  }
}

# modules/payroll-app/variables.tf
variable "app_region" {
  type = string
}

variable "ami" {
  type = string
}
```

### Utilisation d'un module

```hcl
# main.tf
module "us_payroll" {
  source = "../modules/payroll-app"
  
  app_region = "us-east-1"
  ami        = "ami-24e140119877avm"
}

module "uk_payroll" {
  source = "../modules/payroll-app"
  
  app_region = "eu-west-2"
  ami        = "ami-35e140119877avm"
}
```

### Module depuis Registry

```hcl
module "security-group_ssh" {
  source  = "terraform-aws-modules/security-group/aws/modules/ssh"
  version = "3.16.0"
  
  vpc_id              = "vpc-7d8d215"
  ingress_cidr_blocks = ["10.10.0.0/16"]
  name                = "ssh-access"
}
```

**Avantages:**
- Réutilisabilité
- Standardisation
- Configurations simplifiées
- Risque réduit

---

## 23. Workspaces

Les workspaces permettent de gérer plusieurs environnements avec la même configuration.

### Commandes

```bash
terraform workspace list           # Liste les workspaces
terraform workspace new dev        # Créer workspace "dev"
terraform workspace select dev     # Sélectionner workspace "dev"
terraform workspace show           # Afficher workspace actuel
```

### Utilisation

```hcl
resource "aws_instance" "example" {
  ami           = var.ami
  instance_type = terraform.workspace == "prod" ? "t2.large" : "t2.micro"
  
  tags = {
    Name = "${terraform.workspace}-server"
  }
}
```

### State par workspace

Chaque workspace a son propre state:
- `terraform.tfstate.d/dev/terraform.tfstate`
- `terraform.tfstate.d/prod/terraform.tfstate`

---

## 24. Conditional Expressions

```hcl
variable "env" {
  default = "dev"
}

resource "aws_instance" "webserver" {
  ami           = "ami-0edab43b6fa892279"
  instance_type = var.env == "prod" ? "t2.large" : "t2.micro"
}
```

Syntaxe: `condition ? true_val : false_val`

---

## 25. Fonctions Terraform

### Fonctions Numériques

```hcl
max(5, 12, 9)          # 12
min(5, 12, 9)          # 5
ceil(10.1)             # 11
floor(10.9)            # 10
```

### Fonctions String

```hcl
split(",", "ami-xyz,ami-abc")              # ["ami-xyz", "ami-abc"]
join(",", ["ami-xyz", "ami-abc"])          # "ami-xyz,ami-abc"
lower("AMI-XYZ")                           # "ami-xyz"
upper("ami-xyz")                           # "AMI-XYZ"
title("ami xyz")                           # "Ami Xyz"
substr("ami-xyz", 0, 3)                    # "ami"
```

### Fonctions Collection

```hcl
length(["a", "b", "c"])                    # 3
index(["a", "b", "c"], "b")                # 1
element(["a", "b", "c"], 1)                # "b"
contains(["a", "b"], "b")                  # true
```

### Fonctions Map

```hcl
keys({a = 1, b = 2})                       # ["a", "b"]
values({a = 1, b = 2})                     # [1, 2]
lookup({a = 1, b = 2}, "a")                # 1
```

### Fonctions de Type

```hcl
toset(["a", "b", "a"])                     # ["a", "b"]
tolist(set)                                # Convertit en liste
tomap(object)                              # Convertit en map
```

---

## 26. Commandes Terraform Essentielles

### Commandes de base

```bash
terraform init          # Initialiser le répertoire
terraform plan          # Prévisualiser les changements
terraform apply         # Appliquer les changements
terraform destroy       # Détruire toutes les ressources
terraform validate      # Valider la configuration
terraform fmt           # Formater les fichiers HCL
```

### Commandes avancées

```bash
terraform show          # Afficher le state actuel
terraform output        # Afficher les outputs
terraform refresh       # Synchroniser le state avec l'infrastructure réelle
terraform graph         # Générer un graphe de dépendances
terraform graph | dot -Tsvg > terraform_graph.svg # Graphe au format SVG
terraform console       # Console interactive
terraform providers     # Afficher les providers requis
```

### Options utiles

```bash
terraform apply -auto-approve           # Appliquer sans confirmation
terraform apply -target=resource.name   # Appliquer sur une ressource spécifique
terraform plan -out=plan.tfplan        # Sauvegarder le plan
terraform apply plan.tfplan            # Appliquer un plan sauvegardé
terraform destroy -target=resource.name # Détruire une ressource spécifique
```

### Exemples de sorties de commandes

**terraform validate:**
```bash
$ terraform validate
Success! The configuration is valid.
```

En cas d'erreur:
```bash
$ terraform validate
Error: Unsupported argument
  on main.tf line 4, in resource "local_file" "pet":
   4: file_permissions = "0777"

An argument named "file_permissions" is not expected here. 
Did you mean "file_permission"?
```

**terraform fmt:**
```bash
$ terraform fmt
main.tf  # Affiche les fichiers formatés
```

**terraform providers:**
```bash
$ terraform providers

Providers required by configuration:
.
└── provider[registry.terraform.io/hashicorp/aws]

Providers required by state:

    provider[registry.terraform.io/hashicorp/aws]
```

---

## 27. Organisation des Fichiers

### Structure recommandée

```
project/
├── main.tf              # Ressources principales
├── variables.tf         # Déclarations de variables
├── outputs.tf           # Déclarations d'outputs
├── provider.tf          # Configuration du provider
├── terraform.tfvars     # Valeurs des variables
└── modules/             # Modules locaux
    └── module-name/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

---

## 28. Best Practices

1. **State Management**
   - Toujours utiliser un remote backend pour la collaboration
   - Activer le state locking
   - Ne jamais éditer le state manuellement

2. **Organisation du Code**
   - Séparer les environnements avec des workspaces ou des répertoires
   - Utiliser des modules pour la réutilisabilité
   - Garder les fichiers HCL courts et lisibles

3. **Sécurité**
   - Ne jamais commit de credentials dans le code
   - Utiliser des variables pour les données sensibles
   - Utiliser `sensitive = true` pour les outputs sensibles

4. **Versioning**
   - Versionner tout le code Terraform
   - Utiliser des version constraints pour les providers
   - Utiliser le .gitignore approprié:
     ```
     .terraform/
     *.tfstate
     *.tfstate.backup
     .terraform.lock.hcl
     ```

5. **Testing**
   - Toujours exécuter `terraform plan` avant `apply`
   - Valider avec `terraform validate`
   - Utiliser `terraform fmt` pour la cohérence

6. **Documentation**
   - Documenter les modules avec un README
   - Utiliser des descriptions pour variables et outputs
   - Commenter le code complexe

---

## 29. Mutable vs Immutable Infrastructure

### Mutable (Configuration Management)
- Mise à jour sur place
- Risque de configuration drift
- Moins reproductible

### Immutable (Terraform par défaut)
- Recréation des ressources
- État prévisible
- Reproductibilité garantie

Terraform favorise l'infrastructure immuable en recréant les ressources lors de changements (sauf updates in-place possibles).

---

## 30. Terraform Cloud

### Présentation

Plateforme SaaS pour Terraform offrant:
- Remote state management
- UI web pour visualisation
- VCS integration
- Collaboration en équipe
- Governance et policies
- Private module registry
- Cost estimation

### Avantages

**Terraform Cloud offre:**
- **Shared State**: État partagé entre les membres de l'équipe
- **Consistent and Reliable Environment**: Environnement cohérent et fiable
- **UI Interface**: Interface graphique intuitive
- **Secret Management**: Gestion sécurisée des secrets
- **Access Controls**: Contrôles d'accès granulaires
- **Private Registry**: Registre privé pour modules
- **Policy Controls**: Contrôles de politique (Sentinel)

### Plans Tarifaires

**FREE PLAN:**
- Remote State
- Remote Operations
- Private Module Registry
- Community Support

**TEAM PLAN:**
- Toutes les fonctionnalités du plan gratuit
- Team Management (gestion d'équipe)

**TEAM AND GOVERNANCE:**
- Toutes les fonctionnalités du plan Team
- Policy as Code (Sentinel)
- Policy Enforcement
- Cloud SLA and Support

**BUSINESS:**
- Toutes les fonctionnalités précédentes
- SSO (Single Sign-On)
- Custom Concurrency
- Self-hosted options
- Premium Support

**URL de tarification:** https://www.hashicorp.com/products/terraform/pricing

### Intégration avec Version Control

Terraform Cloud peut s'intégrer avec les systèmes de contrôle de version pour:
- Déclencher automatiquement des plans lors de commits
- Effectuer des reviews de code
- Gérer les versions de l'infrastructure
- Supporter différentes versions de Terraform (1.0, 0.12, etc.)

---

## 31. Conclusion

Terraform est un outil puissant pour l'Infrastructure as Code qui permet:
- De définir l'infrastructure de manière déclarative
- De gérer des ressources multi-cloud
- D'automatiser le provisioning
- De versionner l'infrastructure
- De faciliter la collaboration
- De garantir la reproductibilité

### Points clés à retenir

1. **Workflow:** init → plan → apply → destroy
2. **State:** Source de vérité de l'infrastructure
3. **Modules:** Réutilisabilité et standardisation
4. **Remote backend:** Collaboration et sécurité
5. **HCL:** Langage simple et déclaratif
6. **State Locking:** Protection contre les modifications concurrentes
7. **Workspaces:** Gestion multi-environnements
8. **Providers:** Support de multiples plateformes cloud

### Préparation à la Certification

Pour réussir l'examen Terraform Associate:
- Maîtriser les concepts fondamentaux (state, providers, resources)
- Pratiquer avec des labs hands-on
- Comprendre les workflows Terraform
- Connaître les commandes essentielles
- Comprendre la gestion du state et les backends
- Savoir utiliser les modules et workspaces
- Pratiquer avec des questions à choix multiples

**Ressources:**
- Documentation officielle: https://www.terraform.io/docs
- Registry: https://registry.terraform.io
- Certification: https://www.hashicorp.com/certification/terraform-associate
