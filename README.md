# Instalación y uso de Argo CD en Kubernetes (Azure)

## 1. Creación del namespace para Argo CD

Antes de instalar Argo CD, crea un namespace dedicado:

```bash
kubectl create namespace argocd
```

---

## 2. Instalación de Argo CD

Aplica el manifest oficial de Argo CD en tu namespace:

```bash
kubectl apply -n argocd --server-side -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## 3. Configurar acceso externo al servidor de Argo CD

Por defecto, `argocd-server` es de tipo `ClusterIP`. Para exponerlo con un LoadBalancer:

```bash
kubectl patch svc argocd-server -n argocd   -p '{"spec":{"type":"LoadBalancer"}}'
```

Luego, obtén la IP externa (EXTERNAL-IP) asignada:

```bash
kubectl get svc -n argocd
```

---

## 4. Obtener la contraseña inicial

La contraseña por defecto del usuario `admin` está en el secreto `argocd-initial-admin-secret`. Para obtenerla:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret   -o jsonpath="{.data.password}" | base64 -d
```

- **Usuario:** admin  
- **Contraseña:** resultado del comando anterior  

Accede a la interfaz web desde tu navegador usando la EXTERNAL-IP del paso 3:

```
https://<EXTERNAL-IP>
```

---

## 5. Desplegar aplicaciones con Argo CD

### Desde la interfaz web

1. Haz clic en **New App**  
2. Configura los campos principales:

| Campo            | Ejemplo                                  |
|-----------------|------------------------------------------|
| Application Name | mi-app                                   |
| Project          | default                                  |
| Repository URL   | https://github.com/mi-org/mi-repo.git    |
| Revision         | main                                     |
| Path             | manifests/                               |
| Cluster          | default (tu cluster actual)              |
| Namespace        | el namespace donde quieres desplegar     |

3. Haz clic en **Create**  
4. La app aparecerá como **OutOfSync**. Haz clic en **Sync** para desplegar.

### Desde kubectl (opcional, usando manifests)

```bash
kubectl apply -f <manifest.yaml> -n <namespace>
```

---

## 6. Verificar LoadBalancer de tu aplicación

Después de aplicar tu Service tipo LoadBalancer:

```bash
kubectl get svc -n <namespace>
```

Ejemplo de salida:

```
NAME          TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
nlb-service   LoadBalancer   10.0.199.149   20.74.21.114    8000:32324/TCP 7m
```

- **EXTERNAL-IP** → dirección pública que puedes usar para acceder a tu app  
- **PORT(S)** → puertos expuestos  

Prueba conectividad:

```bash
curl http://20.74.21.114:8000
curl http://20.74.21.114:8080
```

---

## Tips adicionales

- Si tu LoadBalancer aparece `<pending>`:
  - Asegúrate de que tu cluster soporta LoadBalancer (Azure sí, Minikube/Kind requieren MetalLB)  
- En Argo CD, el árbol de recursos te muestra estado **Healthy/OutOfSync**.  
- Puedes habilitar **Automatic Sync** para que Argo CD despliegue automáticamente los cambios de Git.  
- Para rollback, selecciona una versión anterior desde la interfaz.  

🐙 ¡Y listo!
