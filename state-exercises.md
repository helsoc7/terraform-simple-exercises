
# Terraform State – Übungen

Nachdem wir erste Infrastruktur mit Terraform erstellt haben, wollen wir nun verstehen, wie **Terraform den Zustand unserer Infrastruktur speichert und verwaltet**.

Terraform speichert den aktuellen Zustand in einer Datei:

```
terraform.tfstate
```

Diese Datei enthält Informationen wie:

- Welche Ressourcen existieren
- IDs der Ressourcen (z.B. EC2 Instance ID)
- Attribute der Ressourcen (IP-Adresse, Status etc.)

Terraform vergleicht immer:

```
Terraform Code (Soll-Zustand)
VS
Terraform State + Cloud (Ist-Zustand)
```

---

# Übung 5 – Terraform State anschauen

## Ziel

Verstehen, welche Informationen Terraform über unsere Infrastruktur speichert.

## Aufgabe

1. Stelle sicher, dass deine EC2 Instanz existiert:

```
terraform apply
```

2. Schaue dir den Terraform State an:

```
terraform show
```

oder

```
terraform state list
```

## Beobachte

Welche Ressourcen werden angezeigt?

Zum Beispiel:

```
aws_instance.webserver
```

## Zusatzaufgabe

Zeige nur Details zu dieser Resource:

```
terraform state show aws_instance.webserver
```

Welche Informationen findest du dort?

- Instance ID
- AMI
- Public IP
- Instance Type

---

# Übung 6 – Änderung im Code beobachten

## Ziel

Verstehen, wie Terraform Änderungen erkennt.

## Aufgabe

Öffne deine Datei:

```
main.tf
```

Ändere den Instanztyp:

```
instance_type = "t2.micro"
```

zu

```
instance_type = "t3.micro"
```

## Führe nun aus

```
terraform plan
```

## Beobachte

Terraform zeigt dir eine Änderung:

```
~ instance_type = "t2.micro" -> "t3.micro"
```

Frage:

- Wird die Instanz ersetzt?
- Wird sie nur geändert?

Diskutiert im Kurs.

---

# Übung 7 – Infrastruktur manuell verändern

## Ziel

Verstehen, was passiert, wenn Änderungen **nicht über Terraform** durchgeführt werden.

## Aufgabe

1. Öffne die AWS Console
2. Suche deine EC2 Instanz
3. Stoppe die Instanz manuell

## Danach führe aus

```
terraform plan
```

## Beobachtung

Terraform erkennt:

- Der Zustand stimmt nicht mehr mit dem Code überein.

Diskussionsfrage:

Warum ist es problematisch, Infrastruktur **manuell zu ändern**?

---

# Übung 8 – Terraform Refresh / Synchronisation

## Ziel

Verstehen, wie Terraform den Zustand mit der Cloud abgleicht.

Terraform kann den Zustand mit der echten Infrastruktur synchronisieren.

Führe aus:

```
terraform refresh
```

Danach:

```
terraform show
```

Terraform aktualisiert jetzt den State basierend auf der realen Infrastruktur.

Frage:

Warum ist dieser Schritt wichtig?

---

# Übung 9 – Terraform State Datei analysieren

## Ziel

Verstehen, wie Terraform Informationen speichert.

Öffne die Datei:

```
terraform.tfstate
```

Du wirst eine JSON-Datei sehen.

Beispielstruktur:

```
{
  "resources": [
    {
      "type": "aws_instance",
      "name": "webserver",
      "instances": [
        {
          "attributes": {
            "instance_type": "t2.micro",
            "public_ip": "..."
          }
        }
      ]
    }
  ]
}
```

## Aufgabe

Suche in der Datei:

- instance_type
- public_ip
- instance_id

Frage:

Warum sollte diese Datei **nicht in GitHub öffentlich gepusht werden**?

---

# Reflexionsfragen

Diskutiert gemeinsam:

1. Warum ist der Terraform State kritisch für die Infrastruktur?
2. Was passiert, wenn zwei Personen gleichzeitig `terraform apply` ausführen?
3. Warum speichern große Teams den State **nicht lokal**, sondern z.B. in:
   - AWS S3
   - Terraform Cloud
4. Welche Probleme könnten entstehen, wenn der State verloren geht?

---

# Bonusaufgabe – Terraform Destroy

Zum Schluss könnt ihr eure Infrastruktur wieder löschen.

```
terraform destroy
```

Terraform zeigt:

```
- aws_instance.webserver will be destroyed
```

Bestätigt mit:

```
yes
```

Danach existiert die Infrastruktur nicht mehr.
