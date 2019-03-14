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

https://www.neuland-bfi.de/~claus.beerta/k8s/

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
