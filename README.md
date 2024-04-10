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
- `minikube start` o `minikube start --vm-driver=<docker / virtualbox>` iniciar Minikube.
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
- Un Pod es la unidad mas pequeña de Kubernetes.

## Seccion 5: Explorando Pods

- `kubectl api-resources` ver los comandos que tienen **SHORTNAMES**.
- `kubectl api-versions` ver las verciones de kubernetes disponibles.
- `kubectl run <pod name> --image=<image>:<image version>` crea un pod.
- `kubectl get pods` listar todos los pods.
- `kubectl get pod <pod name> -o yaml` obtiene el **.yaml** desde un Pod creado.
- `kubectl get pod <pod name> -o wide` la bandera **-o wide** permite ver informacion adicional al listar los Pods.
- `kubectl describe pod <pod name>` descrive un pod. Es decir, muestra todos los detalles y eventos del pod.
- `kubectl delete pod <pod name>` eliminar un pod. Se pueden listar varios pod a eliminar.
- `kubectl exec -it <pod name> -- sh` entrar a un contenedor en modo interactivo.
- `kubectl port-forward <pod name> <host port>:<pod port>` ecrear un túnel seguro desde una dirección local a un puerto específico en un pod en ejecución en Kubernetes.
- `kubectl logs <pod name>` ver los logs de un pod. Si agrego la bandera `-f` puedo obserbar los logs en vivo.

### Manifiestos de Kubernetes

Un manifiesto es el usu de archivos **YAML** para la creacion de objetos en el cluster.

- Archivo pod-yml:
  ```yml
  apiVersion: v1
  kind: Pod
  metadata:
    name: podtest2
  spec:
    containers:
      - name: container1
        image: nginx:alpine
  ```
- `kubectl apply -f pod.yml` crea el pod con el archivo pod.yml
- `kubectl delete -f pod.yml` elimina todos lo creado con pod.yml
- Podemos crear varios pods con un solo archivo.yml:
  ```yml
  apiVersion: v1
  kind: Pod
  metadata:
    name: podtest2
  spec:
    containers:
      - name: container1
        image: nginx:alpine
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: podtest3
  spec:
    containers:
      - name: container2
        image: nginx:alpine
  ```

### Pods con mas de un contenedor

- Se necesita colocar 2 imagenes en la parte de **containers**:
  ```yml
  apiVersion: v1
  kind: Pod
  metadata:
    name: podtest
  spec:
    containers:
      - name: container1
        image: nginx:alpine
      - name: container2
        image: nginx:alpine
  ```
- `kubectl logs <pod name> -c <container name>` ver los logs de un contenedor si el pod tiene dos o mas contenedores.
- `kubectl exec -it <pod name> -c <container name> -- sh` ingresa a un contenedor en especifico del pod si el pod tiene dos o mas contenedores.

### Labels y Pods

Los labels son metadata para identificar los pods dentro del cluster.

- `kubectl get po -l app=frontend` filtra los pods con el label **app: frontend**.
- Se recomienda que por lo menos incluyamos el label **app: <value>**.
- `kubectl label pods <pod name> <label-key>=<label-value>` agrega un label a un pod.

### Problemas con los Pods

- Si el pod falla, no puede recuperarse solospor si solo.
- Por si mismos no pueden replicarse.
- Los Pods no pueden actualizarse asi mismos.

## Seccion 6: SeplicaSets

### Que es ReplicaSet

- Indica cuantas replicas o copias de un pod quiero crear.
- Si un pod falla, ReplicaSet se encargara de crearlo nuevamente.
- Es importante que cada pod tenga un label para que el ReplicaSet pueda identificar los pods y crear uno nuevo si alguno falla.
- ReplicaSet agrega un **Owner References** en la metadata del pod para saber a que ReplicaSet pertenece.

### Funcionamiento de ReplicaSet

Archivo replicaSet.yml:
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-test
  labels:
    app: rs-test
spec:
  replicas: 5
  selector:
    matchLabels:
      app: pod-label
  template:
    metadata:
      labels:
        app: pod-label
    spec:
      containers:
        - name: container1
          image: python:3.6-alpine
          command: ['sh', '-c', 'echo cont1 > index.html && python -m http.server 8082']
        - name: container2
          image: python:3.6-alpine
          command: ['sh', '-c', 'echo cont2 > index.html && python -m http.server 8083']
```

- La metadata del ReplicaSet debe tener un nombre y por lo menos el label `app: <label value>`.
- `replicas: 5` indica el numero de replicas, en este caso 5.
- `selector:` indica que pods debe tener en cuenta.
- `template:` indica la estructura del pod.
- El label del pod debe coincidir con los labels indicados en el **matchLabels** del **selector**.
- `kubectl apply -f archivo.yml` crea el ReplicaSet.
- ReplicaSet puede usar pod ya creados si estos coinciden con los labels definidos en el selector.

### Owner References

Cada ReplicaSet tiene en su metedata un identificador unico **uid**, los pods que pertenecen a este ReplicaSet tendran en su metadadta este mismo **uid** en la seccion de ownerReferences.

### Problemas de ReplicaSet

- ReplicaSet puede utilizar pods ya creados si estos coinciden con los labels definidos en el selector, lo que puede causar que use un pod muy diferente a los indicados en el archivo .yml del ReplicaSet
- No permite cambios directamente en el pod dentro del template YAML.

## Seccion 7: Deployments

### Que es un Deployment

- Un Deployment es el dueño de un ReplicaSet.
- Permite actualizar tu aplicacion. En este proceso el Deployment  usa dos parametros:
  - Max Unavailable: porcentaje de pods ue pueden estar inactivos (Por defecto 25%) al momento de actualizar el ReplicaSet.
  - Max Surge: porcentaje de Pods permitidos en estado de actualizacion (Por defecto 25)
- Para actualizar el ReplicaSet, el Deployment crea otro ReplicaSet.
- Una vez terminada la actualizacion, los pods del ReplicaSet anterior deben estar inactivos.
- Kubernetes mantiene 10 ReplicaSets dentro de un Deployment.

### Estructura del Deployment

Archivo dep.yml:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-test
  labels:
    app: front
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod-label
  template:
    metadata:
      labels:
        app: pod-label
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
```

- La metadata del Deployment debe tener un nombre y por lo menos el label `app: <label value>`.
- `replicas: 3` indica el numero de replicas, en este caso 3.
- `selector:` indica que pods debe tener en cuenta.
- `template:` indica la estructura del pod.
- El label del pod debe coincidir con los labels indicados en el **matchLabels** del **selector**.
- `kubectl apply -f archivo.yml` crea el Deployment.
- `kubectl get deployment <deploy name> --show.labels` la bandera `--show.labels` permite mostrar los labels del deployment.

### Owner References

Cada Deployment tiene en su metedata un identificador unico **uid**, los ReplicaSet que pertenecen a este Deployment tendran en su metadadta este mismo **uid** en la seccion de ownerReferences. Y de la misma manera los ReplicaSets tendaran Pods.

### Rolling Updates

- `kubectl apply -f deployment.yml` realiza los cambios realizados en el template.
- `kubectl rollout status deployment deploy-test` estados del rollout de un deployment.
- `kubectl rollout history deploy deploy-test` ver el historial de rollouts.
- `kubectl rollout history deploy <deploy name> --revision=2` permite revisar un rollout en especifico, en este caso es el rollout 2.


### Limite de ReplicaSets

Agregar `revisionHistoryLimit: 5` en el **spec** del template del Deployment, en este caso 5 es el limite. De esta manera solo Kubernetes solo guardara una version.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-test
  labels:
    app: front
spec:
  revisionHistoryLimit: 1
  replicas: 3
  selector:
    matchLabels:
      app: pod-label
  template:
    metadata:
      labels:
        app: pod-label
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 70
```
### Change-Cause

Existen diferentes formas formas. Se recomienda hacerlo directamente en el template:

- `kubectl apply -f deploy.yml --record=true` la bandera `--record=true` permite agregar el comando ejecutado como causa.
- `kubectl annotate deploy <deploy name> kubernetes.io/change-cause="<cause message>"` para especificar el cambio realizado.
- Agregar una `annotations` con el parametro `kubernetes.io/change-cause: "<cause message>` en la metadata del template del Deployment:

  ```yml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      kubernetes.io/change-cause: "Changes port to 110"
    name: deploy-test
    labels:
      app: front
  spec:
    revisionHistoryLimit: 10
    replicas: 3
    selector:
      matchLabels:
        app: pod-label
    template:
      metadata:
        labels:
          app: pod-label
      spec:
        containers:
        - name: nginx
          image: nginx:alpine
          ports:
          - containerPort: 110
  ```

### Roll Back

Permite vover a una version anterior del Deployment por si algo sale mal.

- `kubectl rollout undo deploy <deploy name> --to-revision=4` permite volver a la version de la revicion 4.
