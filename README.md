# VIRTUALIZACIÓN

Repositorio para tareas generales y parciales del curso de Virtualización.

# Assessment 02 - Kubernetes, MetalLB y Traefik

En este assessment se configuró un ambiente de Kubernetes utilizando Minikube, MetalLB y Traefik. El objetivo fue publicar cuatro aplicaciones web utilizando una sola dirección IP.

## Tecnologías utilizadas

- Kubernetes
- Minikube
- VirtualBox
- MetalLB
- Traefik
- Helm
- Nginx
- Apache HTTP Server
- Traefik Whoami

---

## 1. Configuración del clúster

Para realizar la práctica se utilizó Minikube con VirtualBox.

La IP del nodo de Minikube fue:

```text
192.168.59.101
```

Para comprobar la IP se utilizó:

```bash
minikube ip
```

El perfil utilizado fue:

```text
minikube
```

---

## 2. MetalLB

Primero se habilitó MetalLB utilizando el addon de Minikube:

```bash
minikube addons enable metallb
```

Después se verificó que los componentes estuvieran funcionando:

```bash
kubectl get pods -n metallb-system
```

En esta configuración fue necesario instalar los CRDs de MetalLB:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.2/config/crd/bases/metallb.io_ipaddresspools.yaml

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.2/config/crd/bases/metallb.io_l2advertisements.yaml
```

### Configuración del rango de IP

Se utilizó el rango:

```text
192.168.59.240 - 192.168.59.250
```

La configuración se encuentra en:

```text
metallb/metallb-config.yaml
```

El archivo contiene:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.59.240-192.168.59.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default-advertisement
  namespace: metallb-system
spec:
  ipAddressPools:
    - default-pool
```

Se aplicó con:

```bash
kubectl apply -f metallb/metallb-config.yaml
```

Para comprobar la configuración:

```bash
kubectl get ipaddresspools -n metallb-system
kubectl get l2advertisements -n metallb-system
```

MetalLB utiliza el namespace:

```text
metallb-system
```

---

## 3. Instalación de Traefik

Para instalar Traefik se utilizó Helm.

Primero se agregó el repositorio:

```bash
helm repo add traefik https://traefik.github.io/charts
```

Luego:

```bash
helm repo update
```

Se instaló Traefik en su propio namespace y configurado como `LoadBalancer`:

```bash
helm install traefik traefik/traefik \
  --namespace traefik \
  --create-namespace \
  --set service.type=LoadBalancer
```

Se comprobó que el pod estuviera funcionando:

```bash
kubectl get pods -n traefik
```

Traefik utiliza el namespace:

```text
traefik
```

---

## 4. IP del LoadBalancer

Después de instalar Traefik, MetalLB le asignó la IP:

```text
192.168.59.240
```

Se puede comprobar con:

```bash
kubectl get svc -n traefik
```

El servicio aparece como:

```text
traefik   LoadBalancer   ...   192.168.59.240
```

Esta es la única IP externa que se utiliza para acceder a las cuatro aplicaciones.

---

## 5. Aplicaciones

Las cuatro aplicaciones se crearon dentro del namespace:

```text
parcial-fdcm
```

Se utilizaron cuatro Deployments:

- `app1` - Nginx
- `app2` - Apache HTTP Server
- `app3` - Traefik Whoami
- `app4` - Nginx

Cada Deployment tiene su propio Service.

Los archivos se encuentran en:

```text
apps/
```

Para comprobar los Deployments:

```bash
kubectl get deployments -n parcial-fdcm
```

Resultado:

```text
app1   1/1
app2   1/1
app3   1/1
app4   1/1
```

---

## 6. Services

Cada aplicación tiene un Service de tipo `ClusterIP`.

Esto permite que las aplicaciones sean accesibles dentro del clúster, mientras que Traefik se encarga de recibir las solicitudes externas.

Los Services son:

| Aplicación | Service |
|---|---|
| app1 | app1-service |
| app2 | app2-service |
| app3 | app3-service |
| app4 | app4-service |

Para comprobarlos:

```bash
kubectl get svc -n parcial-fdcm
```

---

## 7. Ingress y Traefik

Se creó un Ingress para que Traefik pueda identificar a qué aplicación enviar cada solicitud.

El archivo utilizado es:

```text
traefik/traefik-ingress.yaml
```

Los dominios configurados son:

| Dominio | Service |
|---|---|
| app1.local | app1-service |
| app2.local | app2-service |
| app3.local | app3-service |
| app4.local | app4-service |

Todos los dominios utilizan la misma IP:

```text
192.168.59.240
```

La idea es que Traefik reciba todas las solicitudes y dependiendo del dominio las mande al Service correspondiente.

---

## 8. Configuración de los dominios

Como se trabajó de forma local, no fue necesario comprar dominios.

Se utilizó el archivo:

```text
/etc/hosts
```

Se agregaron estas líneas:

```text
192.168.59.240 app1.local
192.168.59.240 app2.local
192.168.59.240 app3.local
192.168.59.240 app4.local
```

De esta forma los cuatro dominios apuntan a la IP de Traefik.

---

## 9. Pruebas

Se comprobó que los recursos principales estuvieran funcionando.

### Deployments

```bash
kubectl get deployments -n parcial-fdcm
```

### Pods

```bash
kubectl get pods -n parcial-fdcm
```

### Services

```bash
kubectl get svc -n parcial-fdcm
```

### Ingress

```bash
kubectl get ingress -n parcial-fdcm
```

También se hicieron pruebas directamente desde la terminal:

```bash
curl http://app1.local
curl http://app2.local
curl http://app3.local
curl http://app4.local
```

Las cuatro aplicaciones también fueron probadas desde el navegador utilizando sus respectivos dominios.

### App 1

![App 1](image-4.png)

### App 2

![App 2](image-5.png)

### App 3

![App 3](image-6.png)

### App 4

![App 4](image-7.png)

---

## 10. Comandos de verificación

### MetalLB

```bash
kubectl get pods -n metallb-system
kubectl get ipaddresspools -n metallb-system
kubectl get l2advertisements -n metallb-system
```

### Traefik

```bash
kubectl get pods -n traefik
kubectl get svc -n traefik
```

### Aplicaciones

```bash
kubectl get deployments -n parcial-fdcm
kubectl get pods -n parcial-fdcm
kubectl get svc -n parcial-fdcm
kubectl get ingress -n parcial-fdcm
```

### Pruebas HTTP

```bash
curl http://app1.local
curl http://app2.local
curl http://app3.local
curl http://app4.local
```

---


## Conclusión

En esta práctica se logró configurar un clúster de Kubernetes utilizando Minikube, MetalLB y Traefik.

MetalLB asignó la IP:

```text
192.168.59.240
```

a Traefik, y esta misma IP se utilizó para las cuatro aplicaciones.

Las aplicaciones quedaron dentro del namespace:

```text
parcial-fdcm
```

y cada una tiene su propio Service.

Finalmente, mediante Traefik y los dominios configurados en `/etc/hosts`, fue posible acceder a las cuatro aplicaciones usando:

```text
http://app1.local
http://app2.local
http://app3.local
http://app4.local
```