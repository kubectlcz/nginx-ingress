# Instalace nginx-ingress demo controlleru

## Nasazeni Deployment yaml
Nasazeni tohoto deploymentu se nasadi nginx server
```
kubernetes apply -f nginx-deploy-main.yaml
```

## Vystaveni service na portu 80 Deploymentu
```
kubectl expose deploy nginx-deloy-main --port 80
```

## Nasazeni Ingress resource
timto pro nginx deployment nastavime jak se da k tomuto pristpupit `nginx.example.com`
```
kubectl apply -f ingress-resource.yaml
```

## Overeni ze ingress pro deplyment se nasaditl a bezi
```
kubectl get ing
kubectl describe ing ingress-resource-1
```

# Update dns
Kdyz potrebujete pridat dalsi dns nazev tak se musi
1. vytvorit dalsi deployment soubor jako `nginx-deploy-main.yaml` napr `nginx-deploy-main2.yaml` a v tom souboru dat jiny nazev deploymentu napr `nginx-deploy-main2`
2. Nasadit tento deployment napriklad
```
kubernetes apply -f nginx-deploy-main2.yaml
```
3. Vytvoreni service pro deployment
```
kubectl expose deploy nginx-deloy-main2 --port 80
```
4. Upravit ingresu pro service deploymentu 
pridat dalsi sekci do hosts v souboru `ingress-resource.yaml`
```
- host: new.nginx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-deploy-main2
            port:
              number: 80
```

5. updatovat /etc/hosts 
```
72.16.16.100 nginx.example.com
72.16.16.100 new.nginx.example.com
```