# Instalace nginx-ingress na kubernetes cluster
Dokument ze ktereho muzete brat prikazy copy & paste

# Zakladni dokument
[NGINX dokumentace](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/)

## Clone repository
```
git clone git@github.com:kubectlcz/nginx-ingress.git
cd nginx-ingress
```


## 1. Configurace RBAC

### Vytvoreni namespace a servisniho uctu pro Ingress controller
```
kubectl apply -f common/ns-and-sa.yaml
```

### Vytvoreni cluster role a cluster role binding pro servisni ucet
```
kubectl apply -f rbac/rbac.yaml
```

### Vytvoreni App protect role a role bindingu
```
kubectl apply -f rbac/ap-rbac.yaml
```

## 2. Vytvoreni zakladnich zdroju

V teto sekci vytvorime zakladni zdroje pro ingress controller.

### Vytvoreni secret s TLS certifikatu a klice pro defaultni server v NGINX
```
kubectl apply -f common/default-server-secret.yaml
```

### Vytvoreni config map for upravovani NGINX configurace
```
kubectl apply -f common/nginx-config.yaml
```

### Toto je asi nejdulezitejsi cast vytvoreni IngressClass na kterou se pak budeme odkazovat v demo ingress
```
kubectl apply -f common/ingress-class.yaml
```

## 3. Nasazeni Ingress controlleru
Jelikoz chcem aby instance ingress controlleru bezelo na vsech nodech tak pujdeme cestou DaemonSet

### Vytvoreni nginx-ingress jako DaemonSet
```
kubectl apply -f daemon-set/nginx-ingress.yaml
```

### Kontrola ze Ingress bezi
```
kubectl get pods --namespace=nginx-ingress
```

## 4. Dat pristup k Ingress controlleru
Jeliz jdeme cestou DaemonSet tak nepotrebujeme delat loadbalancer jako v pripade Deploymentu takze staci vytvorit NodePort service, ktera se postara o konektivitu na Ingress controller

### Vytvoreni NodePort service
```
kubectl create -f service/nodeport.yaml
```

### Zkotrolujte jestli service bezi a je nastavena ok
```
kubectl describe svc nginx-ingress --namespace=nginx-ingress
```

# Zaver
v tuto chvili by melo byt vsechno nastaveno a muzete zkustit nasadit [demo](../demo/README.md)