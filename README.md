# KUBERNETES DE PRINCIPIANTE A EXPERTO

## Seccion 1: Introducion

### Que es kubernetes

Herramienta que permite facilitar la administracion de contenedores.

### Que Puede hacer kubernetes

- Service discovery: descubre los contenedores del backend para que los contenedores del fronned hagan las solicitudes a un solo un punto.
- Rollouts and rollbacks: actualizar y deshacer cambion en un contenedor.
- Optimizacion de recursos en los nodos: verifica que maquina es la mas indicada para alojar un contenedor.
- Self-healing: verifica que todos los contenedores esten funcionando, si uno falla crea uno nuevo para reemplazarlo.
- Configuracion de secretos
- Escalamiento horizontal
- Mucho mas!

## Seccion 2: Arquitectura de kubernetes

![Kubernetes Architecture](/img/kubernetesArchitecture.png)

### Arquitectura Master-Node

Kubernetes se divide en master y nodos. Master es el cerebro de kubernetes, es decir, controla a los nodos y realiza todas las operaciones. Los nodos son maquinas virtuales o fisicas donde corren los contenedores.

### Kubernetes API

Servicio que permite al cliente comunicarse con kubernetes y darle ordenes como `kubectl run`, `kubectl apply`, `kubectl delete`, `kubectl get`, etc. Tambien se puede mediante archivos **JSON**.

### Kube-Scheduler

Se encarga de colocar el contenedor en el lugar correcto. Por ejemplo, encuentra en que nodo debe colocar el contenedor ya creado.

### Kube Controller

- Contiene el Node controller, Replication controller, EndPoint controller, Services acount y Tokens Controller:
  - El Node controller verifica que todos los nodos esten funcionando y si uno cae, se encarga de crear uno nuevo.
  - Replication controller se encarga que todas las replicas solicitadas esten funcionando corecctamente.
  - EndPoint controller: se trata de servicios y pods a nivel de redes.
  - Services acount y Tokens Controller: se encarga de la autenticacion cuando hay cominicacion con Kubernetes API.

### Etcd

Es una base de datos key-value donde el cluster almacena el estado, datos, backups, configuraciones, etc.

### Kubelet

- Servicio que corre en cada nodo.
- Recibe informacion del Nodo master y lo plasma en los nodos.
- Envia informacion (estadisticas) al Nodo master.
- Por ejemplo, El master le dice al kubelet que cree un contenedor docker, kubelet se comunica con docker para crear el contenedor solicitado. En cada nodo debe estar instalado Docker.

### Kube-Proxy

- Servicio que corre en cada nodo.
- Se encarga de todo el tema de redes relacionados con contenedores.

### Container Runtime

- Instalado en cada nodo.
- Se encarga de crear los contenedores en cada nodo. si usamos Docker, este debe estar instalado en cada nodo.

## Seccion 3: Instalacion de Minikube

### Kubectl

Segir el tutorial oficial de [Install Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

```cmd
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO https://dl.k8s.io/release/v1.29.2/bin/linux/amd64/kubectl
curl -LO https://dl.k8s.io/release/v1.29.2/bin/linux/arm64/kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
# La salida deb ser: kubectl: OK 
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl /usr/bin/kubectl
kubectl version
```

### Instalacion de Docker

Segir el tutorial oficial de [Docker engine](https://docs.docker.com/engine/install/ubuntu/) or [Docker Desktop](https://docs.docker.com/desktop/install/linux-install/).

Si sale este error al ejecutar el comando `docker`, seguir los siguinetes pasos: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.45/containers/json": dial unix /var/run/docker.sock: connect: permission denied

1. Get the user: `whoami`
2. Create docker group if not exist : ``sudo groupadd docker
3. Add user to docker group : `sudo usermod -aG docker ${USER}`
4. Change docker.sock to new permission : `sudo chmod 666 /var/run/docker.sock`
5. Finally restart docker daemon service : `sudo systemctl restart docker`

### Instalacion de Minikube

Seguir el tutorial oficial de [Install Minikube](https://minikube.sigs.k8s.io/docs/start/).

- Editar el archivo `~/.bashrc` para agregar un alias. Por ejemplo agregar `alias kubectl="sudo kubectl"`. Una vez guardado los cambios ejecutar `source ~/.bashrc`.

### Comandos basicos de Minikube

- `minikube status` ver el estado del Minikube.
- `minikube stop` detener Minikube.
- `minikube start` iniciar Minikube.
- `minikube delete` eliminar Minikube 

## Seccion 4: Pods en Kubernetes VS Contenedores de Docker

### Contenedor Docker

- Un contenedor se ejecuta de manera aislada en un namespace:
  - IPC (Inter process Comunication): un contenedor puede ver solo sus propios procesos.
  - Cgroup: controla el consusmo de recursos en un contenedor.
  - Network: cada contenedor tiene su propia red.
  - Mount: permite montar un sistema de archivos dentro de un contenedor.
  - PID: es el id que se le da a ,los procesos dentro de un contenedor. Un contenedor solo puede ver sus propios procesos.
  - User: Un contenedor olo puede ver los usuarios creados en el.
  - UTS (Unix Timesharing System): Permite colocar un hostname unico a un contenedor.

### Pod en Kubernetes

- Docker crea una red Bridge virtual para unir los contenedores y asi estos se puedan comunicar. 
- Kubernetes comparte los namespaces: IPC, Network, UTS.
- Kubernetes crea un contenedor temporal para compartir su ID (ID de IPC, ID de Network, ID de UTS) con los demas contenedores.
- Como se hereda la misma configuracion de red, todos los contenedores tendran el mismo ID.
- Un Pod es uno o mas contenedores que comparten namespaces.
- Un Pod tiene una sola IP.

## Seccion 5: Explorando Pods

- `kubectl run pod <pod name> --image=<image>:<image version>` crea un pod.
- `kubectl get pods` listar todos los pods.
- `kubectl describe pod <pod name>` descrive un pod. Es decir, muestra todos los detalles y eventos del pod.