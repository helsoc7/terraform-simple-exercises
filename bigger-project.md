# Terraform Beispielprojekt – Statische Website auf EC2 deployen

## Praktische Ausgangssituation

Ein kleines Unternehmen möchte eine **einfache Landingpage** bereitstellen.  
Die Seite besteht nur aus:

- `index.html`
- `style.css`
- `script.js`

Die Dateien liegen bereits in einem Git-Repository.

Die Infrastruktur soll **nicht manuell** in der AWS Console aufgebaut werden, sondern mit **Terraform** erstellt werden.

## Ziel des Projekts

Mit Terraform soll Folgendes umgesetzt werden:

- eine EC2-Instanz erstellen
- eine Security Group erstellen
- HTTP (Port 80) erlauben
- NGINX auf dem Server installieren
- die Website-Dateien automatisch nach `/var/www/html` kopieren
- die Seite im Browser aufrufen

---

# Lernziele

Die Teilnehmer lernen dabei:

- mehrere Terraform-Ressourcen kombinieren
- Abhängigkeiten zwischen Ressourcen verstehen
- mit `user_data` arbeiten
- Terraform für ein echtes Mini-Szenario einsetzen
- Infrastruktur und Anwendungsdateien gemeinsam denken

---

# Projektidee

Wir bauen eine kleine Webserver-Infrastruktur:

```text
Besucher
   ↓
Öffentliche IP der EC2
   ↓
Security Group erlaubt Port 80
   ↓
NGINX Webserver
   ↓
index.html + style.css + script.js
```

---

# Projektstruktur

```text
terraform-website-demo/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
└── website/
    ├── index.html
    ├── style.css
    └── script.js
```

---

# Schritt 1 – Website-Dateien vorbereiten

Lege im Projektordner einen Unterordner `website/` an.

## `website/index.html`

```html
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Terraform Demo Website</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <main class="container">
    <h1>Willkommen auf meiner Terraform Website</h1>
    <p>Diese Seite wurde automatisch mit Terraform und EC2 bereitgestellt.</p>
    <button id="btn">Klick mich</button>
    <p id="message"></p>
  </main>

  <script src="script.js"></script>
</body>
</html>
```

## `website/style.css`

```css
body {
  font-family: Arial, sans-serif;
  background: #f4f7fb;
  color: #222;
  margin: 0;
  padding: 0;
}

.container {
  max-width: 700px;
  margin: 80px auto;
  background: white;
  padding: 32px;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.08);
  text-align: center;
}

button {
  background: #2563eb;
  color: white;
  border: none;
  padding: 12px 18px;
  border-radius: 8px;
  cursor: pointer;
}

button:hover {
  background: #1d4ed8;
}
```

## `website/script.js`

```javascript
const btn = document.getElementById("btn");
const message = document.getElementById("message");

btn.addEventListener("click", () => {
  message.textContent = "JavaScript läuft erfolgreich auf dem Webserver.";
});
```

---

# Schritt 2 – Terraform-Konfiguration erstellen

## `variables.tf`

```hcl
variable "aws_region" {
  description = "AWS Region"
  type        = string
  default     = "eu-central-1"
}

variable "instance_type" {
  description = "EC2 Instanztyp"
  type        = string
  default     = "t2.micro"
}

variable "ami_id" {
  description = "AMI für Amazon Linux"
  type        = string
}
```

---

## `terraform.tfvars`

> Die AMI kann je nach Region variieren.  
> Du kannst die passende AMI in der AWS Console oder über die AMI-Suche nachsehen.

```hcl
aws_region    = "eu-central-1"
instance_type = "t2.micro"
ami_id        = "ami-xxxxxxxxxxxxxxxxx"
```

---

## `main.tf`

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

resource "aws_security_group" "web_sg" {
  name        = "terraform-demo-web-sg"
  description = "Erlaubt HTTP und SSH"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Alles raus"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "terraform-demo-web-sg"
  }
}

resource "aws_instance" "webserver" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y nginx

              systemctl enable nginx
              systemctl start nginx
              EOF

  tags = {
    Name = "terraform-demo-webserver"
  }
}
```

---

## `outputs.tf`

```hcl
output "public_ip" {
  description = "Öffentliche IP des Webservers"
  value       = aws_instance.webserver.public_ip
}

output "website_url" {
  description = "URL der Website"
  value       = "http://${aws_instance.webserver.public_ip}"
}
```

---

# Schritt 3 – Terraform ausführen

## Initialisieren

```bash
terraform init
```

## Prüfen

```bash
terraform plan
```

## Infrastruktur erstellen

```bash
terraform apply
```

Am Ende bekommst du die öffentliche IP bzw. URL ausgegeben.

---

# Schritt 4 – Website-Dateien auf den Server bringen

Bis hierhin läuft NGINX, aber noch nicht deine eigene Website.

Jetzt gibt es **zwei didaktisch sinnvolle Wege**.

---

# Variante A – Dateien manuell per SCP kopieren

Diese Variante ist einfacher zu verstehen, weil die Teilnehmer klar sehen:

- Terraform erstellt Infrastruktur
- wir deployen die Dateien anschließend

## Beispiel

```bash
scp -r website/* ec2-user@DEINE_PUBLIC_IP:/home/ec2-user/
```

Dann per SSH verbinden:

```bash
ssh ec2-user@DEINE_PUBLIC_IP
```

Und auf dem Server:

```bash
sudo cp /home/ec2-user/index.html /var/www/html/index.html
sudo cp /home/ec2-user/style.css /var/www/html/style.css
sudo cp /home/ec2-user/script.js /var/www/html/script.js
```

Danach die IP im Browser öffnen.

---

# Variante B – Dateien direkt über Terraform kopieren

Diese Variante ist näher an echter Automatisierung, aber etwas komplexer.

Dafür braucht man:

- ein SSH Key Pair
- eine Verbindung per `connection`
- einen `file`-Provisioner
- einen `remote-exec`-Provisioner

> Für Einsteiger ist das eher eine **Erweiterung** als der erste Schritt.

## Zusätzliche Idee

Wenn ihr möchtet, könnt ihr später einen zweiten Projektstand bauen:

- Terraform erstellt die Instanz
- Terraform kopiert die Website-Dateien
- Terraform verschiebt die Dateien automatisch an den richtigen Ort

---

# Erweiterte Variante mit Key Pair und File Deployment

## Zusätzliche Variablen

Ergänze in `variables.tf`:

```hcl
variable "key_name" {
  description = "Name des AWS Key Pairs"
  type        = string
}

variable "private_key_path" {
  description = "Pfad zum privaten SSH Schlüssel"
  type        = string
}
```

## Beispiel in `terraform.tfvars`

```hcl
key_name         = "mein-keypair"
private_key_path = "~/.ssh/mein-keypair.pem"
```

## EC2 Resource erweitert

```hcl
resource "aws_instance" "webserver" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  key_name               = var.key_name
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y nginx
              systemctl enable nginx
              systemctl start nginx
              EOF

  tags = {
    Name = "terraform-demo-webserver"
  }

  provisioner "file" {
    source      = "website/"
    destination = "/home/ec2-user/website"

    connection {
      type        = "ssh"
      user        = "ec2-user"
      host        = self.public_ip
      private_key = file(var.private_key_path)
    }
  }

  provisioner "remote-exec" {
    inline = [
      "sudo cp /home/ec2-user/website/index.html /var/www/html/index.html",
      "sudo cp /home/ec2-user/website/style.css /var/www/html/style.css",
      "sudo cp /home/ec2-user/website/script.js /var/www/html/script.js",
      "sudo systemctl restart nginx"
    ]

    connection {
      type        = "ssh"
      user        = "ec2-user"
      host        = self.public_ip
      private_key = file(var.private_key_path)
    }
  }
}
```

---

# Warum ist dieses Projekt didaktisch gut?

Weil die Teilnehmer hier nicht nur eine „leere EC2“ erstellen, sondern ein greifbares Ergebnis bekommen:

- eine echte Website
- ein sichtbares Deployment
- ein nachvollziehbares Zusammenspiel aus Infrastruktur und Anwendung

Das macht Terraform viel anschaulicher.

---

# Empfohlener Unterrichtsablauf

## Phase 1 – Infrastruktur erstellen

Die Teilnehmer bauen:

- Security Group
- EC2 Instanz
- NGINX Installation per `user_data`

## Phase 2 – Website-Dateien bereitstellen

Zuerst einfach:

- HTML
- CSS
- JS per SCP hochladen

## Phase 3 – Reflexion

Fragen:

- Welche Teile übernimmt Terraform?
- Welche Teile sind noch manuell?
- Wie könnte man den Deployment-Schritt weiter automatisieren?

---

# Kleine Übungen zum Projekt

## Übung 1 – Titel der Website ändern

Ändere in `index.html` die Überschrift und prüfe im Browser, ob die Änderung sichtbar ist.

---

## Übung 2 – Neues CSS hinzufügen

Füge der Seite eine neue Hintergrundfarbe oder Button-Farbe hinzu.

---

## Übung 3 – JavaScript erweitern

Erweitere `script.js`, sodass nach dem Klick zusätzlich die aktuelle Uhrzeit angezeigt wird.

---

## Übung 4 – Terraform Tags ergänzen

Füge an der EC2 Instanz zusätzliche Tags hinzu:

```hcl
tags = {
  Name        = "terraform-demo-webserver"
  Environment = "Dev"
  Owner       = "Kurs"
}
```

Dann:

```bash
terraform plan
terraform apply
```

---

# Recherchefragen

## Frage 1
Was ist der Unterschied zwischen:

- `user_data`
- `file` Provisioner
- `remote-exec`

---

## Frage 2
Warum ist es sinnvoll, Website-Dateien im Repository mit abzulegen?

Denke an:

- Versionierung
- Nachvollziehbarkeit
- Teamarbeit

---

## Frage 3
Warum sollte man Port 22 und Port 80 bewusst konfigurieren und nicht einfach „alles öffnen“?

---

## Frage 4
Welche Nachteile haben Provisioner in Terraform?

Tipp: Suche nach „Terraform provisioners best practices“.

---

# Mögliche Erweiterungen für später

Wenn das Projekt gut läuft, kann man es später ausbauen:

- Elastic IP hinzufügen
- Domain verbinden
- S3 statt EC2 verwenden
- Load Balancer ergänzen
- Auto Scaling vorbereiten
- Remote State mit S3 einführen
- GitHub Actions für automatisches Deployment nutzen

---

# Abschluss

Zum Schluss kann die Infrastruktur wieder gelöscht werden:

```bash
terraform destroy
```

So bleibt die Sandbox sauber und unnötige Ressourcen werden entfernt.
