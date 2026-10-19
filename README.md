# VIRTUALIZACI-N
Repositorio para tareas generales y parciales del curso de Virtualización
# Assessment 06 - Argo CD

En esta actividad instalé Argo CD en Minikube, lo expuse con Traefik y desplegué las apps de `parcial-fdcm` con GitOps.

## 1. Instalar Argo CD

Creé la carpeta `argocd/` con un `kustomization.yaml` que instala Argo CD en el namespace `argocd-fdcm`. Usé Kustomize porque el instalador oficial trae el namespace `argocd` fijo.

```bash
kubectl apply -k argocd --server-side --force-conflicts
kubectl get pods -n argocd-fdcm
```

Todos los pods quedaron en `Running`.

## 2. Exponerlo con Traefik

Creé un IngressRoute (`argocd/ingressroute.yaml`) con el dominio `argocd.fdcm.test` y lo agregué al `/etc/hosts` con la IP del LoadBalancer de Traefik.

```bash
kubectl apply -f argocd/ingressroute.yaml
echo "192.168.59.240 argocd.fdcm.test" | sudo tee -a /etc/hosts
```

Ya se puede entrar a `http://argocd.fdcm.test`:

![login](/image/loginArgo.png)
![pagina principal](/image/paginaPrincipal.png)

## 3. Conectar el repositorio

El repo es público, así que solo lo registré con un Secret (`argocd/repo-secret.yaml`).

```bash
kubectl apply -f argocd/repo-secret.yaml
```

## 4. Crear las Application

Copié los YAML de las apps de la rama `assessment-02` a la carpeta `parcial/` y creé tres Application en `argocd/apps/`, apuntando a la rama `assessment-06`:

- `parcial-app1-app2`: app1 y app2
- `parcial-app3-app4`: app3 y app4
- `parcial-ingress`: el Ingress de las apps

```bash
kubectl apply -f argocd/apps/
kubectl -n argocd-fdcm get applications
```

Las tres quedaron en Synced y Healthy:

![antes](/image/argocd-antes.png)

## 5. Cambio en el repositorio

Cambié las réplicas de app1 de 1 a 2 en `parcial/app1-app2/app1.yaml` y hice commit y push. Argo CD detectó el cambio, lo sincronizó y la Application volvió a Synced y Healthy con 2 pods de app1.

![despues](/image/despues.png)

## Problemas

Tuve un problema de DNS dentro de Minikube y no descargaba las imágenes. Las descargué en mi Mac y las cargué con `minikube image load`, y cambié `imagePullPolicy` a `IfNotPresent`. También tuve que cambiar el DNS de CoreDNS para que Argo CD pudiera conectarse a GitHub.