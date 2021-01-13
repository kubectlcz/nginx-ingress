# Instalace nginx-ingress demo controlleru

## Nasazeni Deployment yaml
Nasazeni tohoto deploymentu se nasadi nginx server
```
kubernetes apply -f nginx-deploy-main.yaml
```

## Vystaveni port Deploymentu
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