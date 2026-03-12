# Terraform nach Folie 10 – einfache Anleitung mit AWS Sandbox, AWS CLI und 4 Übungen

Diese Anleitung knüpft **nach Folie 10** an und führt die Teilnehmer Schritt für Schritt in die ersten **praktischen Terraform-Arbeiten** ein.

Ziel ist es, dass die Teilnehmer:

- eine **AWS Cloud Sandbox in Pluralsight** starten,
- ihre **AWS CLI** dafür einrichten,
- die ersten **Terraform-Dateien** verstehen,
- und mit **4 kleinen, sehr einfachen Übungen** arbeiten.

---

## 1. Was kommt nach Folie 10?

Bis Folie 10 ging es vor allem um:

- Probleme manueller Infrastruktur
- Ziele von Infrastrukturmanagement
- Standardisierung, Nachvollziehbarkeit und Automatisierung

Ab jetzt kommt der praktische Teil:

- **Infrastructure as Code mit Terraform**
- **AWS als Provider**
- **erste Ressourcen in Code beschreiben**
- **Terraform init, plan, apply und destroy anwenden**

---

## 2. AWS Cloud Sandbox in Pluralsight starten

> Hinweis: Die Oberfläche kann sich leicht ändern. Die grundsätzlichen Schritte bleiben aber ähnlich.

### Schritte

1. In **Pluralsight** einloggen.
2. Im Menü zu **Hands-on** wechseln.
3. Die Kachel **Cloud Sandboxes** öffnen.
4. **AWS** auswählen.
5. Auf **Start Sandbox** klicken.
6. Warten, bis die Sandbox bereit ist.
7. Danach die bereitgestellten Informationen ansehen:
   - AWS Console Link
   - Benutzername oder temporäre Zugangsdaten
   - Region (falls angezeigt)
   - gegebenenfalls Ablaufzeit der Sandbox

### Wichtig

Eine Sandbox ist normalerweise **zeitlich begrenzt**. Änderungen darin sind meist nur für die Dauer der Sandbox verfügbar. Deshalb eignet sie sich sehr gut zum Üben, aber nicht als dauerhafte Umgebung.

---

## 3. AWS CLI einrichten

Es gibt zwei typische Wege:

- **Variante A:** mit `aws configure`
- **Variante B:** mit temporären Umgebungsvariablen

Wenn Pluralsight in der Sandbox **Access Key, Secret Key und Session Token** anzeigt, ist **Variante B** oft die passendere Methode.

---

## 4. AWS CLI installieren

### Prüfen, ob AWS CLI schon installiert ist

```bash
aws --version
```

Wenn ein Versionswert angezeigt wird, ist die CLI bereits installiert.

---

### Installation unter Windows

Die AWS CLI v2 mit dem offiziellen Installer installieren und danach ein neues Terminal öffnen.

Danach testen:

```bash
aws --version
```

---

### Installation unter macOS

Mit dem offiziellen PKG-Installer oder alternativ mit Homebrew arbeiten.

Beispiel mit Homebrew:

```bash
brew install awscli
aws --version
```

---

### Installation unter Linux

Nach der Installation ebenfalls prüfen:

```bash
aws --version
```

---

## 5. AWS CLI mit `aws configure` einrichten

Diese Variante eignet sich gut, wenn Access Key und Secret Key vorhanden sind.

```bash
aws configure
```

Dann nacheinander eingeben:

- AWS Access Key ID
- AWS Secret Access Key
- Default region name
- Default output format

Beispiel:

```text
AWS Access Key ID [None]: AKIA...
AWS Secret Access Key [None]: geheim...
Default region name [None]: us-east-1
Default output format [None]: json
```

---

## 6. AWS CLI mit temporären Umgebungsvariablen einrichten

Diese Variante ist besonders wichtig, wenn die Sandbox **temporäre Credentials** bereitstellt.

### Linux / macOS

```bash
export AWS_ACCESS_KEY_ID="DEIN_ACCESS_KEY"
export AWS_SECRET_ACCESS_KEY="DEIN_SECRET_KEY"
export AWS_SESSION_TOKEN="DEIN_SESSION_TOKEN"
export AWS_DEFAULT_REGION="us-east-1"
```

### Windows PowerShell

```powershell
$env:AWS_ACCESS_KEY_ID="DEIN_ACCESS_KEY"
$env:AWS_SECRET_ACCESS_KEY="DEIN_SECRET_KEY"
$env:AWS_SESSION_TOKEN="DEIN_SESSION_TOKEN"
$env:AWS_DEFAULT_REGION="us-east-1"
```

### Test

```bash
aws sts get-caller-identity
```

Wenn die Einrichtung funktioniert, gibt AWS Informationen zur aktuell verwendeten Identität zurück.

---

## 7. Terraform kurz wiederholen

Terraform arbeitet mit dem Prinzip:

- **Soll-Zustand im Code definieren**
- **Ist-Zustand in AWS abfragen**
- **Abweichungen berechnen**
- **Änderungen anwenden**

Die wichtigsten Befehle:

```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

### Bedeutung

- `terraform init` → lädt Provider und initialisiert das Projekt
- `terraform plan` → zeigt geplante Änderungen
- `terraform apply` → setzt Änderungen um
- `terraform destroy` → löscht verwaltete Ressourcen wieder

---

## 8. Einfaches Terraform-Projekt anlegen

### Ordnerstruktur

```text
terraform-demo/
├── main.tf
├── variables.tf
└── outputs.tf
```

---

## 9. Erstes Beispiel: AWS Provider

Datei: `main.tf`

```hcl
terraform {
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
```

Datei: `variables.tf`

```hcl
variable "aws_region" {
  description = "AWS Region"
  type        = string
  default     = "us-east-1"
}
```

### Erklärung

- `required_providers` sagt Terraform, welchen Provider es braucht.
- `provider "aws"` stellt die Verbindung zu AWS her.
- `var.aws_region` macht die Region flexibel.

---

## 10. Terraform initialisieren

Im Projektordner ausführen:

```bash
terraform init
```

Was passiert?

- Terraform lädt den AWS Provider herunter.
- Der Ordner `.terraform` wird angelegt.
- Das Projekt ist bereit für `plan` und `apply`.

---

## 11. Zweites Beispiel: eine sehr einfache EC2-Instanz

> Hinweis: Eine AMI-ID kann je nach Region abweichen. Wenn die angegebene AMI in eurer Region nicht funktioniert, müsst ihr eine passende Amazon-Linux-AMI in der AWS Console suchen.

Datei: `main.tf`

```hcl
terraform {
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

resource "aws_instance" "demo" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = "terraform-demo-instance"
  }
}
```

Datei: `variables.tf`

```hcl
variable "aws_region" {
  type    = string
  default = "us-east-1"
}

variable "ami_id" {
  description = "AMI fuer die EC2-Instanz"
  type        = string
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}
```

Datei: `outputs.tf`

```hcl
output "instance_id" {
  value = aws_instance.demo.id
}

output "public_ip" {
  value = aws_instance.demo.public_ip
}
```

Datei: `terraform.tfvars`

```hcl
ami_id = "ami-xxxxxxxxxxxxxxxxx"
```

---

## 12. Ablauf im Terminal

```bash
terraform init
terraform plan
terraform apply
```

Nach `apply` bestätigt man mit:

```text
yes
```

Zum Schluss:

```bash
terraform destroy
```

---

## 13. Drittes Beispiel: eine Security Group

Jetzt erweitern wir das Projekt um eine sehr einfache Security Group.

Datei: `main.tf`

```hcl
terraform {
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

resource "aws_security_group" "demo_sg" {
  name        = "terraform-demo-sg"
  description = "Einfache Demo Security Group"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "demo" {
  ami           = var.ami_id
  instance_type = var.instance_type

  vpc_security_group_ids = [aws_security_group.demo_sg.id]

  tags = {
    Name = "terraform-demo-instance"
  }
}
```

### Erklärung

- `aws_security_group` erstellt eine Firewall-Regel in AWS.
- Über `aws_security_group.demo_sg.id` wird die Security Group an die EC2-Instanz gebunden.
- Terraform erkennt diese Abhängigkeit automatisch.

---

## 14. Viertes Beispiel: Variablen bewusst nutzen

Datei: `variables.tf`

```hcl
variable "aws_region" {
  type    = string
  default = "us-east-1"
}

variable "ami_id" {
  type = string
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "instance_name" {
  type    = string
  default = "meine-demo-instanz"
}
```

Datei: `main.tf`

```hcl
resource "aws_instance" "demo" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = var.instance_name
  }
}
```

Datei: `terraform.tfvars`

```hcl
ami_id         = "ami-xxxxxxxxxxxxxxxxx"
instance_type  = "t2.micro"
instance_name  = "kurs-demo-01"
```

### Vorteil

Die Infrastruktur bleibt gleich aufgebaut, aber Werte wie Name, Region oder Instanztyp können leicht geändert werden.

---

# 15. Vier sehr einfache Übungen

## Übung 1 – AWS CLI testen

### Aufgabe

Richte die AWS CLI mit den Sandbox-Zugangsdaten ein und prüfe die Verbindung.

### Schritte

1. Zugangsdaten aus der Pluralsight-Sandbox kopieren.
2. CLI konfigurieren.
3. Diesen Befehl ausführen:

```bash
aws sts get-caller-identity
```

### Ziel

Die Teilnehmer sehen, dass ihre CLI mit AWS sprechen kann.

### Reflexionsfrage

Wozu braucht Terraform eine funktionierende AWS-Authentifizierung?

---

## Übung 2 – Nur den Provider definieren

### Aufgabe

Erstelle ein leeres Terraform-Projekt mit:

- `main.tf`
- `variables.tf`

Füge nur den AWS Provider ein und führe aus:

```bash
terraform init
```

### Ziel

Die Teilnehmer verstehen, dass ein Projekt auch ohne Ressourcen initialisiert werden kann.

### Beobachtung

Es wird noch nichts in AWS erstellt.

---

## Übung 3 – Erste EC2-Instanz planen

### Aufgabe

Ergänze eine einfache `aws_instance` Resource und führe aus:

```bash
terraform plan
```

### Ziel

Die Teilnehmer sehen die geplanten Änderungen, bevor etwas erstellt wird.

### Reflexionsfrage

Warum ist `terraform plan` in Teams und in produktiven Umgebungen so wichtig?

---

## Übung 4 – Tag oder Instanztyp ändern

### Aufgabe

Ändere einen einfachen Wert, zum Beispiel:

- den Namen der Instanz in den Tags
- oder den `instance_type`

Führe danach erneut aus:

```bash
terraform plan
```

### Ziel

Die Teilnehmer sehen, dass Terraform Änderungen erkennt und anzeigt.

### Zusatz

Wenn genug Zeit da ist:

```bash
terraform apply
```

und danach wieder:

```bash
terraform destroy
```

---

## 16. Typische Fehlerquellen

### Fehler 1 – AWS CLI nicht korrekt eingerichtet

Symptome:

- Authentifizierungsfehler
- Zugriff verweigert
- Terraform kann nicht mit AWS sprechen

### Fehler 2 – Session Token vergessen

Gerade bei temporären Sandbox-Zugangsdaten wird oft der `AWS_SESSION_TOKEN` vergessen.

### Fehler 3 – falsche Region

Eine AMI-ID kann nur in der passenden Region gültig sein.

### Fehler 4 – `terraform destroy` vergessen

In Übungsumgebungen sollte am Ende immer aufgeräumt werden.

---

## 17. Mini-Zusammenfassung für die Teilnehmer

- Terraform beschreibt Infrastruktur **als Code**.
- AWS ist in diesem Beispiel der **Provider**.
- Die AWS CLI stellt sicher, dass Terraform mit AWS sprechen darf.
- Mit `init`, `plan`, `apply` und `destroy` läuft der grundlegende Workflow.
- Schon kleine Änderungen im Code lassen sich sauber nachvollziehen.

---

## 18. Nächster sinnvoller Schritt im Unterricht

Nach diesen einfachen Übungen könnt ihr weitermachen mit:

- mehreren Ressourcen gleichzeitig
- Security Groups detaillierter
- Outputs
- Terraform State
- Modulen
- Git + Terraform als Team-Workflow

