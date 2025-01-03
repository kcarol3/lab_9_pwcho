# Zaawansowane programowanie w chmurze obliczneniowej

## Laboratorium 9

## 1. Uruchomienie projektu:

Należy pobrać repozytorium w katalogu głównym projektu wykonać następujące polecenia:

```bash
kubectl apply -f cluster-setup.yaml
```
a następnie:
```bash
kubectl apply -f networks-policy.yaml
```

## 2. Opis
 
### W pliku cluster-setup.yaml zdefiniowane zostały:

- Namespace appns-a i appns-b
- Deploy aplikacji app-a i app-b
- Serwisy dla obu deployów
- Serwisy typu ExternalName aby była możliwość odszukania przez ingress serwisów zlokalizowanych w namespaces appns-a i appns-b
- Default backend deploy 
- Serwis dla default backend deploy
- Konfiguracje ingress

### W pliku networks-policy:

- Definicje reguł sieciowych dla namespaces appns-a i appns-b

## Wyniki

- Sprawdzenie konfiguracji:
```bash
macbookpro@MacBook-Pro-MacBook lab9 % kubectl get deployments,pods -n appns-a
kubectl get deployments,pods -n appns-b
kubectl get deployments,pods -n default
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-a   2/2     2            2           18h

NAME                         READY   STATUS    RESTARTS   AGE
pod/app-a-5799485f95-q8tfs   1/1     Running   0          18h
pod/app-a-5799485f95-xmc4v   1/1     Running   0          18h
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app-b   2/2     2            2           18h

NAME                         READY   STATUS    RESTARTS   AGE
pod/app-b-76fd6dfbf5-6zqxg   1/1     Running   0          18h
pod/app-b-76fd6dfbf5-fldst   1/1     Running   0          18h
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-app    1/1     1            1           18h

NAME                              READY   STATUS    RESTARTS   AGE
pod/hello-app-7db98b8f65-hmgbs    1/1     Running   0          18h
```

- Sprawdzenie ingress:
```bash
macbookpro@MacBook-Pro-MacBook lab9 % kubectl get ingress -n default
NAME           CLASS   HOSTS                   ADDRESS        PORTS   AGE
lab9-ingress   nginx   a.lab9.net,b.lab9.net   192.168.58.2   80      18h
```

- Testowanie reguł sieciowych - sprawdzenie łączności z appns-a do appns-b:
```bash
macbookpro@MacBook-Pro-MacBook lab9 % kubectl exec -it app-a-5799485f95-g5r4q --namespace=appns-a -- curl -s -o /dev/null -w "%{http_code}\n" http://app-b.appns-b.svc.cluster.local

000
command terminated with exit code 6
```
Komunikacja nie powiodła się więc reguły sieciowe zostały poprawnie zdefiniowane

- Testowanie działania ingress:
```bash
macbookpro@MacBook-Pro-MacBook lab9 % curl -H "Host: a.lab9.net" http://localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

```bash
macbookpro@MacBook-Pro-MacBook lab9 % curl -H "Host: b.lab9.net" http://localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
