# Terraform Module – Übungen

Diese Übungen bauen auf euren bisherigen Terraform-Grundlagen auf und führen Schritt für Schritt in das Arbeiten mit **Terraform Modulen** ein.

Module helfen dabei, Terraform-Code:

- sauberer zu strukturieren
- wiederverwendbar zu machen
- besser wartbar zu machen
- in Teams einfacher nutzbar zu machen

---

# Warum Module?

Am Anfang funktioniert oft alles in einer einzigen Datei:

```hcl
main.tf
```

Das ist für kleine Beispiele in Ordnung.

Sobald aber mehrere Ressourcen hinzukommen, wird der Code schnell:

- unübersichtlich
- schwer wartbar
- schwer wiederverwendbar

Ein Modul ist in Terraform im Prinzip ein **wiederverwendbarer Baustein**.

Beispiel:

- ein Modul für eine EC2-Instanz
- ein Modul für eine Security Group
- ein Modul für ein VPC-Setup

---

# Lernziele

Nach diesen Übungen sollen die Teilnehmer:

- verstehen, was ein Modul ist
- ein eigenes einfaches Modul erstellen
- Variablen an ein Modul übergeben
- Outputs aus einem Modul zurückgeben
- erkennen, warum Module in echten Projekten nützlich sind

---

# Ausgangssituation

Bisher habt ihr Infrastruktur direkt im Root-Projekt definiert, zum Beispiel:

```hcl
resource "aws_instance" "webserver" {
  ami           = "ami-123456789"
  instance_type = "t2.micro"
}
```

Jetzt wollen wir diesen Code auslagern.

---

# Zielstruktur

Wir wollen mit folgender Struktur arbeiten:

```text
terraform-module-demo/
│
├── main.tf
├── variables.tf
├── outputs.tf
└── modules/
    └── ec2_instance/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

---

# Übung 1 – Erstes Modul anlegen

## Ziel

Ein eigenes Modul für eine einfache EC2-Instanz erstellen.

## Aufgabe

Lege folgende Ordnerstruktur an:

```text
modules/ec2_instance/
```

Erstelle darin die Dateien:

- `main.tf`
- `variables.tf`
- `outputs.tf`

---

## `modules/ec2_instance/main.tf`

```hcl
resource "aws_instance" "this" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = var.instance_name
  }
}
```

---

## `modules/ec2_instance/variables.tf`

```hcl
variable "ami_id" {
  description = "AMI ID für die EC2 Instanz"
  type        = string
}

variable "instance_type" {
  description = "Instanztyp"
  type        = string
}

variable "instance_name" {
  description = "Name der Instanz"
  type        = string
}
```

---

## `modules/ec2_instance/outputs.tf`

```hcl
output "instance_id" {
  value = aws_instance.this.id
}

output "public_ip" {
  value = aws_instance.this.public_ip
}
```

---

# Übung 2 – Modul im Root-Projekt verwenden

## Ziel

Das neue Modul aus `main.tf` im Hauptprojekt aufrufen.

## Root `main.tf`

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

module "webserver" {
  source = "./modules/ec2_instance"

  ami_id        = var.ami_id
  instance_type = var.instance_type
  instance_name = "modul-webserver"
}
```

---

## Root `variables.tf`

```hcl
variable "aws_region" {
  description = "AWS Region"
  type        = string
  default     = "eu-central-1"
}

variable "ami_id" {
  description = "AMI ID"
  type        = string
}

variable "instance_type" {
  description = "EC2 Typ"
  type        = string
  default     = "t2.micro"
}
```

---

## Root `outputs.tf`

```hcl
output "webserver_public_ip" {
  value = module.webserver.public_ip
}

output "webserver_instance_id" {
  value = module.webserver.instance_id
}
```

---

## `terraform.tfvars`

```hcl
aws_region    = "eu-central-1"
ami_id        = "ami-xxxxxxxxxxxxxxxxx"
instance_type = "t2.micro"
```

---

## Ausführen

```bash
terraform init
terraform plan
terraform apply
```

## Beobachtung

Ihr verwendet jetzt keine Resource direkt im Root mehr, sondern ein Modul:

```hcl
module "webserver" { ... }
```

---

# Übung 3 – Modul mit anderen Werten erneut verwenden

## Ziel

Verstehen, dass ein Modul mehrfach verwendet werden kann.

## Aufgabe

Erstelle in der Root-`main.tf` einen zweiten Modulaufruf:

```hcl
module "appserver" {
  source = "./modules/ec2_instance"

  ami_id        = var.ami_id
  instance_type = "t2.micro"
  instance_name = "modul-appserver"
}
```

---

## Zusatz

Ergänze in `outputs.tf`:

```hcl
output "appserver_public_ip" {
  value = module.appserver.public_ip
}
```

---

## Frage

Was ist der Vorteil davon, dass wir den Code nicht doppelt als Resource schreiben müssen?

Diskutiert:

- weniger Copy-Paste
- bessere Wartbarkeit
- gleiche Standards
- schnelleres Arbeiten

---

# Übung 4 – Security Group als eigenes Modul bauen

## Ziel

Ein zweites eigenes Modul anlegen.

Erstelle einen neuen Modulordner:

```text
modules/security_group/
```

---

## `modules/security_group/main.tf`

```hcl
resource "aws_security_group" "this" {
  name        = var.name
  description = var.description

  ingress {
    from_port   = var.ingress_port
    to_port     = var.ingress_port
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = var.name
  }
}
```

---

## `modules/security_group/variables.tf`

```hcl
variable "name" {
  type = string
}

variable "description" {
  type = string
}

variable "ingress_port" {
  type = number
}
```

---

## `modules/security_group/outputs.tf`

```hcl
output "security_group_id" {
  value = aws_security_group.this.id
}
```

---

## Modul im Root nutzen

Ergänze in `main.tf`:

```hcl
module "web_sg" {
  source = "./modules/security_group"

  name         = "modul-web-sg"
  description  = "Erlaubt HTTP"
  ingress_port = 80
}
```

---

# Übung 5 – Modulabhängigkeiten verbinden

## Ziel

Ein Modul mit einem anderen kombinieren.

Dafür muss das EC2-Modul erweitert werden, damit man eine Security Group ID übergeben kann.

---

## `modules/ec2_instance/variables.tf` erweitern

```hcl
variable "security_group_ids" {
  description = "Liste von Security Group IDs"
  type        = list(string)
  default     = []
}
```

---

## `modules/ec2_instance/main.tf` erweitern

```hcl
resource "aws_instance" "this" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  vpc_security_group_ids = var.security_group_ids

  tags = {
    Name = var.instance_name
  }
}
```

---

## Root `main.tf` anpassen

```hcl
module "webserver" {
  source = "./modules/ec2_instance"

  ami_id             = var.ami_id
  instance_type      = var.instance_type
  instance_name      = "modul-webserver"
  security_group_ids = [module.web_sg.security_group_id]
}
```

---

## Beobachtung

Jetzt greift ein Modul auf den Output eines anderen Moduls zu.

Das ist in echten Projekten sehr typisch.

---

# Übung 6 – Modul mit SSH und HTTP erweitern

## Ziel

Das Security-Group-Modul etwas flexibler machen.

Aktuell kann nur **ein Port** geöffnet werden.

Erweitere das Modul so, dass mehrere Ports möglich sind.

---

## Neue Idee für `variables.tf`

```hcl
variable "ingress_ports" {
  type = list(number)
}
```

---

## Neue Idee für `main.tf`

```hcl
resource "aws_security_group" "this" {
  name        = var.name
  description = var.description

  dynamic "ingress" {
    for_each = var.ingress_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = var.name
  }
}
```

---

## Modulaufruf im Root

```hcl
module "web_sg" {
  source = "./modules/security_group"

  name          = "modul-web-sg"
  description   = "Erlaubt SSH und HTTP"
  ingress_ports = [22, 80]
}
```

---

# Übung 7 – Reflektieren und vergleichen

## Aufgabe

Vergleicht diese beiden Varianten:

### Variante A – alles direkt im Root

```hcl
resource "aws_instance" "webserver" { ... }
resource "aws_security_group" "web_sg" { ... }
```

### Variante B – mit Modulen

```hcl
module "webserver" { ... }
module "web_sg" { ... }
```

---

## Diskussionsfragen

1. Welche Variante ist übersichtlicher?
2. Welche Variante ist besser für Teams?
3. Welche Variante ist besser, wenn dieselbe Infrastruktur mehrfach gebaut werden soll?
4. Welche Variante ist einfacher zu testen und weiterzuentwickeln?

---

# Übung 8 – Kleines Mini-Projekt

## Aufgabe

Baue mit Modulen folgende Infrastruktur:

- 1 Security Group Modul
- 2 EC2-Instanz-Module
  - `webserver`
  - `appserver`

Beide Instanzen sollen:

- dieselbe AMI verwenden
- dieselbe Security Group verwenden
- unterschiedliche Namen haben

---

## Erwartung

Im Root-Projekt soll nur noch ungefähr so etwas stehen:

```hcl
module "web_sg" {
  source = "./modules/security_group"
  ...
}

module "webserver" {
  source = "./modules/ec2_instance"
  ...
}

module "appserver" {
  source = "./modules/ec2_instance"
  ...
}
```

---

# Recherchefragen

## Frage 1
Was ist in Terraform der Unterschied zwischen:

- Root Module
- Child Module

---

## Frage 2
Welche Vorteile haben Module in großen Teams?

Denke an:

- Standardisierung
- Wiederverwendbarkeit
- Wartbarkeit
- Reviewbarkeit

---

## Frage 3
Was ist der Unterschied zwischen:

- eigenen lokalen Modulen
- Modulen aus der Terraform Registry

---

## Frage 4
Suche in der Terraform Registry nach bestehenden AWS-Modulen.  
Welche Module findest du dort z. B. für:

- VPC
- EC2
- Security Groups

---

# Reflexionsfragen

1. Warum sind Module in echten Projekten fast unverzichtbar?
2. Welche Probleme löst ein Modul?
3. Welche Nachteile kann eine zu starke Modularisierung haben?
4. Wann ist ein Projekt noch zu klein für Module?

---

# Bonusaufgabe 1 – Website-Projekt modularisieren

Nehmt euer Website-Projekt und lagert aus:

- EC2 in ein Modul
- Security Group in ein Modul

Wenn ihr möchtet, könnt ihr zusätzlich überlegen:

- ein Modul für Webserver
- ein Modul für Netzwerk

---

# Bonusaufgabe 2 – Modul dokumentieren

Erstellt für euer EC2-Modul eine kleine Dokumentation:

- Welche Variablen gibt es?
- Welche Outputs gibt es?
- Was macht das Modul?

Das ist eine sehr gute Übung für Teamarbeit.

---

# Abschluss

Wenn ihr fertig seid, könnt ihr eure Infrastruktur wieder entfernen:

```bash
terraform destroy
```

So bleibt eure Sandbox sauber.

---

# Merksatz

**Module sind wiederverwendbare Terraform-Bausteine.**  
Sie helfen dabei, Infrastruktur sauber, standardisiert und teamfähig zu definieren.
