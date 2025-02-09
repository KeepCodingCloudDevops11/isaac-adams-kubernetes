![image](https://github.com/user-attachments/assets/642ecd1d-9254-4982-96a2-d5618946a545)# Despliegue de WordPress en Minikube con Helm

Este repositorio describe cómo desplegar **WordPress** usando **Helm** en un clúster de **Minikube** utilizando un archivo `values.yaml` personalizado.

## Prerrequisitos

Antes de comenzar, asegúrate de tener lo siguiente instalado: Minikube, Kubectl y Helm


### Instalación de Minikube

Si aún no tienes Minikube, puedes instalarlo siguiendo la [documentación oficial de Minikube](https://minikube.sigs.k8s.io/docs/).

### Instalación de kubectl

Sigue las instrucciones de la [guía oficial de kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

### Instalación de Helm

Para instalar Helm, puedes seguir la [guía oficial de instalación de Helm](https://helm.sh/docs/intro/install/).

## Pasos para desplegar WordPress en Minikube

### 1. Iniciar Minikube

Primero, necesitas iniciar un clúster de **Minikube**. Abre una terminal y ejecuta:

bash
minikube start
Esto iniciará el clúster de Kubernetes en tu máquina local.

2. Configurar kubectl para Minikube
Después de que Minikube haya iniciado, asegúrate de que tu kubectl está configurado para usar el clúster de Minikube:

kubectl config use-context minikube
3. Habilitar el Ingress en Minikube
El NGINX Ingress Controller debe estar habilitado en Minikube para poder acceder a las aplicaciones mediante un nombre de dominio. Para habilitar el complemento Ingress en Minikube, ejecuta:
```
minikube addons enable ingress
```
4. Crear el archivo values.yaml personalizado
A continuación, usa el siguiente archivo values.yaml para configurar WordPress y MySQL según tus necesidades.

Crea un archivo llamado values.yaml con el siguiente contenido:
```
wordpress:
  replicaCount: 3
  image:
    repository: wordpress
    tag: latest
  service:
    type: NodePort
    port: 80
    targetPort: 80

mysql:
  rootPassword: rootpassword
  user: wpuser
  password: wppassword
  database: wordpress
  image:
    repository: bitnami/mysql
    tag: 8.0
  persistence:
    enabled: true
    size: 1Gi
    storageClass: standard
    accessMode: ReadWriteOnce

namespace: proyectoFinal

hpa:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  cpuUtilization: 70
```
Este archivo configura WordPress y MySQL con:

WordPress con 3 réplicas , imagen latest y un servicio NodePort en el puerto 80.
MySQL con configuración persistente, base de datos llamada wordpress, y credenciales definidas.
Horizontal Pod Autoscaler (HPA) habilitado para escalar el número de réplicas de WordPress entre 3 y 10 basándose en la utilización de CPU.

5. Instalar WordPress en Minikube con el archivo values.yaml
Con el archivo values.yaml configurado, puedes proceder a instalar WordPress y MySQL en el clúster de Minikube usando Helm.

Para hacerlo, ejecuta el siguiente comando:
```
helm install wordpress wordpress-0.1.0.tgx -f values.yaml --namespace proyectoFinal
```
Esto instalará WordPress con todas las configuraciones personalizadas definidas en values.yaml.

6. Verificar que los recursos se han creado
Puedes verificar que los recursos se han creado correctamente usando el siguiente comando:

```
kubectl get all --namespace proyectoFinal
```
Este comando mostrará los pods, servicios y otros recursos creados en el clúster de Minikube en el espacio de nombres proyectoFinal. Asegúrate de que los pods estén en estado Running.

7. Configurar el archivo /etc/hosts
Para poder acceder a la aplicación usando un nombre de dominio como wordpress.local, agrega la IP de Minikube a tu archivo hosts.

Obtén la IP de Minikube:

```
minikube ip
```
Esto debería devolver una IP como 192.168.49.2.

Agrega esta línea al archivo /etc/hosts en Linux/macOS o C:\Windows\System32\drivers\etc\hosts en Windows:

```
192.168.49.2 wordpress.local
```
Esto permite que puedas acceder a WordPress usando wordpress.local.

8. Verificar el acceso a WordPress
Abre tu navegador y visita la URL:

```
http://wordpress.local
```
Deberías ver la página de instalación de WordPress. Si todo ha sido configurado correctamente, podrás completar la instalación de WordPress y comenzar a usarlo.


9. Eliminar el despliegue
Si deseas eliminar el despliegue de WordPress, ejecuta:

```
helm uninstall wordpress --namespace proyectoFinal
```
Esto eliminará todos los recursos de Kubernetes asociados con el despliegue de WordPress.


