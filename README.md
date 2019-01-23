# kubectl Installation

Installation von https://kubernetes.io/docs/tasks/tools/install-kubectl/. Mac User mit `brew install kubectl`

Bash Completion:
 ```bash
 echo "source <(kubectl completion bash)" >> ~/.bashrc
 ```
für Mac User:
```bash
brew install bash-completion
```

# kubectl Konfiguration

Kube Configs für Neuland Cluster:

https://gitlab.neuland-bfi.de/kubernetes-workshop/vortrag/tree/master/users

Datei als `~/.kube/config` ablegen.

# kubectl Cheat Sheet

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

* `kubectl get pods` (pods, deployment, service, ...)  Komponenten anzeigen lassen
* `kubectl create -f path/to/config.yaml` Neue Komponente erstellen
* `kubectl replace -f path/to/config.yaml` Komponente ändern
* `kubectl delete -f path/to/config.yaml` Komponente löschen
* `kubectl delete deployment frontend` (pods, deployment, service, ...) Komponente löschen
* `kubectl describe pod frontend-123-chg` Metainformationen vom pod anzeigen
* `kubectl log frontend-123-chg` Application Logs anzeigen


# Workshop
## 1. Aufgabe
Erstellt ein Deployment für folgendes image:
`026875375293.dkr.ecr.eu-central-1.amazonaws.com/workshop_frontend:latest`
Ihr könnt euch an dieser yaml orientieren:
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: raumschilder
  name: raumschilder
spec:
  replicas: 1
  strategy: {}
  template:
    metadata:
      labels:
        app: raumschilder
    spec:
      containers:
      - image: gitlab.neuland-bfi.de:5043/stefan.kuper/raumschilder:latest
        name: nodejs
        resources: {}
      restartPolicy: Always
status: {}
```

Zum testen könnt ihr `kubectl exec frontend-123-chg -- wget http://localhost:3000 -O -` nutzen.
(Was aus unerklärlichen Gründen nicht jedes Mal klappt. Aber spätestens der dritte Versuch sollte.)

## 2. Aufgabe
Jetzt kommt der Service und der ingress dazu.

Denk dran `{my-dwarf}` zu ersetzen!

Wieder analog zu:
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: raumschilder
  name: raumschilder
spec:
  ports:
  - name: nodejs
    port: 3000
  selector:
    app: raumschilder
```
und
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  labels:
    app: raumschilder
  name: raumschilder
  annotations:
    ingress.kubernetes.io/whitelist-source-range: "10.66.0.0/16,192.168.0.0/16"
spec:
  rules:
  - host: {my-dwarf}.ingress.aws-dev.neuland-bfi.de
    http:
      paths:
      - backend:
          serviceName: raumschilder
          servicePort: 3000
        path: /
```
Testen könnt ihr indem ihr `http://dori.ingress.aws-dev.neuland-bfi.de` im Browser öffnet.


## 3. Aufgabe

Dann könnt ihr mit dem backend weitermachen.
Deployment, service und ingress erstellen.

* image: `026875375293.dkr.ecr.eu-central-1.amazonaws.com/workshop_backend:mysql-memory`
* ingress host: http://`{my-dwarf}`-**api**.ingress.aws-dev.neuland-bfi.de
* Die Application läuft auf port `9000`!

## 4. Aufgabe

Das frontend muss das backend kennen, damit es requests dahin schicken kann.
Daher müsst ihr dem docker image vom frontend eine Umgebungsvariable mitgeben.

`API_ROOT_URL` = `http://{my-dwarf}-api.ingress.aws-dev.neuland-bfi.de/`

Google (oder so) ist euer Freund!

## 5. Aufgabe

Im backend die deployment Strategie konfigurieren:  RollingUpdate mit maxSurge von 1.

Und eine livenessProbe + readinessProbe im backend konfigurieren.

Ihr könnt einfach einen HTTP Call auf `localhost:9000/` machen. Wenn ein 200 status code zurück kommt ist alles gut.

Testen könnt ihr das indem ihr beim backend das image `026875375293.dkr.ecr.eu-central-1.amazonaws.com/workshop_backend:latest` einspielt. Das fährt (noch) nicht hoch.

Und ihr könnt euch auch auf die Fehlersuche machen, warum das backend nicht hochfahren will. (Die nächste Aufgabe beschreibt wie der Fehler zu beheben ist)

Beim frontend könnt ihr die probes auch gerne analog hinzufügen.


## 6. Aufgabe

### 6.1 Secret hinzufügen
Dem backend die Datenbankzugänge verfügbar machen.

Die Datei `database.conf` mit folgendem Inhalt anlegen. Variable ersetzen!
```
include "application.conf"

db.default.driver=com.mysql.jdbc.Driver
db.default.url="jdbc:mysql://mariadb-mariadb.default.svc.cluster.local:3306/{my-dwarf}"
db.default.user={my-dwarf}
db.default.password="{my-dwarf}"
ebean.default="models.*"
```

Aus der Datei ein k8s Secret vom Typ `generic` erstellen.

### 6.2 Backend anpassen
1. Das secret muss in das docker image reingemounted werden.
2. Der docker backend Prozess muss folgendes Argument mitgeliefert bekommen `-Dconfig.file=/path/to/file/in/container/database.conf`
