---
title: Kubernetes Service Accounts verwalten.
body_classes: title-center title-h1h2
published: true
---

Wir können eingeschränkten Zugriff auf unsere Cluster mit Hilfe von kubernetes
Service Accounts und dem kubernetes RBAC Feature umsetzen.

Dafür müssen wir folgendes tun:
  - einen Kubernetes Service Account anlegen.
  - eine Rolle mit beschränktem Zugriff definieren.
  - Dem Kubernetes Service Account diese Rolle zuordnen.
  
  
Die Authentifizierung in Kubernetes Clustern die mit iMKE erzeugt werden,
geschieht über sogenannte bearer tokens. Wenn wir einen neuen Kubernetes
Service Account anlegen, wird ein solches Token oder Secret im jeweiligen
namespace hinterlegt. Dieses Secret wird gelöscht, wenn der Kubernetes
Service Account gelöscht wird.

## Anlegen eines Kubernetes Service Accounts

Um einen Kubernetes Service Account anzulegen, benutzen wir das folgende
Kommando, wobei wir `my-serviceaccount` mit dem Namen ersetzen, den wir dem
Kubernetes Service Account geben wollen:

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
  namespace: my-namespace
EOF
```

Dadurch wird im Cluster automatisch ein neues Access Token angelegt. Dieses
Token trägt einen Namen wir 'my-serviceaccount-token-####' wobei die '#'
zufällige alphanumerische Zeichen sind.

Um die Token in einem Namespace zu sehen, verwenden wir folgendes Kommando:

```bash
kubectl get secrets --namespace=my-namespace
```

Wir können uns dann das eigentliche Token mit dem folgenden Kommando ausgeben
lassen (nicht vergessen, '$SECRETNAME' durch den Namen zu ersetzen, der für
unseren Kubernetes Service Account gilt):

```bash
kubectl get secret $SECRETNAME -o jsonpath='{.data.token}' --namespace=my-namespace
```

Das Token das angezeigt wird, können wir an Dritte übermitteln, um ihnen zu
ermöglichen, mit dem Cluster zu arbeiten.

Jetzt haben wir einen Kubernetes Service Account, der sich an unserem Cluster
authentifizieren kann, aber er hat noch keine Rechte. Im nächsten Schritt
definieren wir eine Rolle und weisen diese Rolle dem Kubernetes Service Account zu,
damit er die entsprechenden Rechte erhält.

## Definition einer Rolle mit ihren Rechten

Grundsätzlich gibt es zwei Wege, einem Kubernetes Service Account Rechte zuzuweisen:
Cluster-weite Rollen oder Rollen, die auf einen jeweiligen Namespace beschränkt sind.
Da die Cluster-weiten Rollen Zugriff auf alle Namespaces geben, empfehlen wir, diese
nur zu verwenden wenn es absolut notwendig ist. Unsere Beispiele definieren Rollen,
die auf einen jeweiligen Namespace beschränkt sind.

Alle Rechte einer Rolle werden explizit freigegeben -- das gilt sowohl für persönliche
Zugänge als auch für Kubernetes Service Accounts. Wenn ein Account mehrere Rollen
innehat, bekommt man die kombinierten Rechte aller Rollen.

Um eine Rolle zu definieren, mit der ein Account die Informationen über Pods im
Namespace 'my-namespace' auslesen kann, verwenden wir den folgenden Befehl:

```bash
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
EOF
```

Jetzt wollen wir dem zuvor angelegten Kubernetes Service Account diese Rolle
zuweisen. Für dieses _role binding_ verwenden wir das folgende Kommando:


```bash
kubectl create rolebinding read-pods \
  --role=pod-reader \
  --serviceaccount=my-namespace:my-serviceaccount \
  --namespace=my-namespace
```

Um alle Ressourcen aufzulisten, rufen wir folgendes Kommando auf:

```bash
kubectl api-resources
```

Für die meisten Ressourcen sind folgende Verben definiert:

 - get
 - list
 - watch
 - create
 - edit
 - update
 - delete
 - exec

### Weiterführende Informationen.

Die offizielle Kubernetes-Dokumentation zu [Zugriffskontrollen](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/) and [using roles and role bindings in RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
behandelt das Thema ausführlich.

## Zusammenfassung

In dieser Anleitung haben wir gelernt, wie wir über die Kommandozeile
 - Kubernetes Service Accounts anlegen
 - Das automatisch generierte Bearer Token für einen Kubernetes Service Accounts auslesen können
 - Eine Rolle mit Hilfe des Kubernetes RBAC Features definieren
 - Mit Hilfe eines _role bindings_ solche Rollen einem Kubernetes Service Account zuordnen
