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

https://gitlab.neuland-bfi.de/admins/ansible-neuland-infra/tree/master/roles/kubernetes/users

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


