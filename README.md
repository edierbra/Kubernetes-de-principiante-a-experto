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
- `kubectl --rm -it run <pod name> --image=<image>:<image version> -- sh` crea un pod y en modo interactivo y una vez salgamos se elimina automaticamente.
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

## Seccion 8: Services & Endpoints

### Que es un servicio

- Observa los Pods con ciertos Labels sin importar a que Deployment o ReplicaSet pertenecen, para que en el caso que un usuario haga una solicitud al servicio, este lo redirija a los Pods que observa.
- La carga se distribuye mediante el load balancer llamado **round robin**.
- El servicio tiene asignado un DNS y una IP al que se realizan las solicitudes.

### Enpoints

- Un Endpoint es una lista de IPs
- Kubernetes crea un Endpoint cada vez que se crea un Servicio con un selector.
- Cada vez que el Servicio encuentra un Pod que coincida con los labels asignados, guarda la IP del Pod en el Endpoint.
- Si un Pod muere, el Servicio lo elimina del EndPoint.
- Cada vez que se crea un nuevo Pod que coincida con los labels del Servicio, este lo guarda en el Endpoint.

### Estructura del Service

Archivo svc.yml:

```yml
apiVersion: v1
kind: Service
metadata:
  name: src-test
  labels:
    app: front
spec:
  selector:
    app: pod-label
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```

- Se Recomienda siempre colocar un label en la metadata del Service.
- El `selector:` indica que labels debo usar para observar a los Pods.
- `protocol`indica el protocolo a usar.
- `port` indica el puerto donde el Servicio va a estar escuchando.
`targetPort` indica el puerto en donde los Pods estan escuchando. Es decir, a que puerto en el los Pods el servicio debe mandar las solicitudes.
- `kubectl describe svc <service name>` para describir un Services por su nombre. De esta manera podremos ver sus Endpoints y demas caracteristicas.

### Pods & Endpoints

- Si un Pod conincide con los Labels del selectro del services, estos se agregaran al endpoint automaticamente.
- Es una mala practica crear Pods indempendientes.
- `kubectl get en` o `kubectl get endpoints` para obtener los endpoints.

### Servicios y DNS

- Internamente podemos acceder al servicio mediante su `<service ip>:<service port>` o con `<service name>:<service port>`.
- `kubectl port-forward service/<service name> <host port>:<service port>` crea un tunel para acceder al services desde el navegador mediante `localhost:<host port>`

### Tipos de Servicios

-  Debemos agregar `type: <type>` dentro del spec del template del Services para especificar el tipo de servicio (ClusterIP, NodePort, LoadBalancer, entre otros).

#### ClusterIp Service

- Es el servicio por defecto.
- Solo es accecible dentro del Cluster y se usa para la comunicacion interna entre servicios.
- Con la ip del servicesno podremos acceder de manera externa al cluster.

#### NodePort Servives

- Expone un servicio fuera del cluster.
- Cada vez que se crea un NodePort se crea un ClusterIP.
- Basicamente es exponer un puerto a nivel del Nodo para que un cliente pueda acceder al ClusterIP, luego al Servicio y finalmente a los Pods.
- El rango de puertos permitidos para el servicio NodePort es del puerto 30000 al puerto 32767.
- `minikube service svc-test2` ver los servicios creados y la direccion externa para acceder a ellos.

#### LoadBalancer Servives

- Provee al nodo un Load Balancer de cloud.
- Cada vez que se crea un LoadBalancer se crea un NodePort y un ClusterIP.
- Cuando un cliente accede al LoadBalancer, puede acceder al NodePort, luego al ClusterIP, despues al Servicio y finalmente a los Pods.
- Por lo general el LoadBalancer se encuentra en una subred publica (puede acceder a internet) pero la comunicacion del balancador al nodo se haga mediante una IP privada. Es decir el nodo se encuentra en una subred privada, no accede a internet.

## Golang, JavaScript y Kubernetes

### Creando el Backend

- Creamos el archivo `k8-hands-on/backend/src/main.go`, puedes ver [esta pagina de ayuda](https://dev.to/moficodes/build-your-first-rest-api-with-go-2gcj):
```go
package main

import (
	"encoding/json"
    "net/http"
	"os"
	"time"
)

type HandsOn struct {
	Time time.Time `json:"time"`
	Hostname string `json:"hostname"`
}

func ServeHTTP(w http.ResponseWriter, r *http.Request) {

	resp := HandsOn{
		Time: time.Now(),
		Hostname: os.Getenv("HOSTNAME"),
	}
	jsonResp, err := json.Marshal(&resp)
	if err != nil { 
		w.Write([]byte("Error"))
		return
	}

	w.WriteHeader(http.StatusOK)
    w.Header().Set("Content-Type", "application/json")
	w.Write(jsonResp)
}

func main() {
    http.HandleFunc("/", ServeHTTP)
    http.ListenAndServe(":9090", nil)
}
```

- `docker pull golang` descargamos el contenedor de golang.
- Nos dirigimos al directorio: `k8-hands-on/backend/src/`.
- `docker run --rm -dit -v $PWD/:/go --net host --name golang golang bash` ejecutamos el contenedor golang en modo interactivo y en segundo plano, compartimos el directorio actual (`k8-hands-on/backend/src/`), en la red host y con un nombre `golang`.
- `docker exec -it golang bash` ingeresamos al contenedor.
- `go run main.go` corere el servicio creado.
- `http://localhost:9090/` permite acceder al servicio desde el navegador

### Crear el Dockerfile

Archico `k8-hands-on/backend/src/Dockerfile`:
```dockerfile
FROM golang:1.13 as builder

WORKDIR /app
COPY main.go .
RUN CGO_ENABLED=0 GOOS=linux GOPROXY=https://proxy.golang.org go build -o app ./main.go

FROM alpine:latest

WORKDIR /app
COPY --from=builder /app/app .
CMD ["./app"]
```

- `sudo docker build -t k8-hands-on -f Dockerfile `ejecuto el Dockerfile para crear la imagen **k8-hands-on**.
- `docker run -d -p 9091:9090 --name k8-hands-on k8-hands-on`ejecuto la imagen **k8-hands-on** y expongo el puerto **9091** de nuestro host para acceder a la aplicacion.
- `docker rm -fv k8-hands-on` para eliminar el contenedor si ya no lo necesito
