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
- `eval $(minikube -p minikube docker-env)` para apuntar al Docker interno de Minikube.

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
- `kubectl run <pod name> --image=<image>:<image version> --image-pull-policy IfNotPresent` crea un pod, pero `--image-pull-policy IfNotPresent` permite buscar si localemnete tenemos la imagen indicada.
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

- `kubectl port-forward service/<service name> <host port>:<service port>` crea un tunel para acceder al services desde el navegador mediante `localhost:<host port>`
- Internamente podemos acceder al servicio mediante su `<service ip>:<service port>` o con `<service name>:<service port>`.

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

### Crear el Dockerfile para el Backend

Archico `k8-hands-on/backend/Dockerfile`:

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

- `docker build -t k8-hands-on -f Dockerfile .`ejecuto el Dockerfile para crear la imagen **k8-hands-on**. La imagen la puedes encontar en DockerHub `ricardoandre97/backend-k8s-hands-on:v1`.
- `docker run -d -p 9091:9090 --name k8-hands-on k8-hands-on`ejecuto la imagen **k8-hands-on** y expongo el puerto **9091** de nuestro host para acceder a la aplicacion.
- `docker rm -fv k8-hands-on` para eliminar el contenedor si ya no lo necesito.

### Crear el manifiesto para el backend

Archivo `k8-hands-on/backend/backend.yml`:
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-k8-hands-on
  labels:
    app: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: k8-hands-on
        imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: backend-k8-hands-on
  labels:
    app: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9090
```

- Agregar `imagePullPolicy: IfNotPresent` en la parte de `containers` del Deployment.
- `kubectl apply -f backend.yml` crea el deployment y el servicio.
- Acceder al servicio en el navegador ingresando a `<service-ip>:<service-port>`.
- Si no se puede acceder posiblemente necesitas crear un tunel con `kubectl port-forward service/backend-k8-hands-on <host-port>:<service-port` e ingresar en el navegador `localhost:<host-port>`.
- Otra forma de acceder al servicio es creando un Pod preferiblemente de **nginx** en modo interactivo y hacer curl al services desde el Pod.

### Creando el Frontend

Crear el archivo `k8-hands-on/frontend/src/index.html`:

```html
<div id="id01"></div>
<img src="https://kubernetes.io/_common-resources/images/flower.svg">

<script>
var xmlhttp = new XMLHttpRequest();
var url = "http://localhost:8081";

xmlhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
        var resp = JSON.parse(this.responseText);
        document.getElementById("id01").innerHTML = "<h2>La hora es " + resp.time + " y el hostname es " + resp.hostname + "</h2>";
    }
};
xmlhttp.open("GET", url, true);
xmlhttp.send();

</script>
```

- `url = "http://localhost:8081"` se configuro la url con la direccion del `kubectl port-forward` del servicio backend por que minikube no esta localmente. Si lo estuviera la url seria `url = "http://backend-k8-hands-on"`, es decir el nombre o ip del servicio del backend.
- Para probarlo crear un pod de nginx y colocar el `index.html` en el directorio `/usr/share/nginx/html/index.html` del contenedor
- Finalmente hacer un `kubectl port-forward`al pod creado e ingresar la direccion en el navegador.
- Pueden haber problemas de CORS, para esto ver el siguiente paso.

### Una nueva version del BackEnd para permitir cualquier origen

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

	w.Header().Set("Content-Type", "application/json")
  w.Header().Set("Access-Control-Allow-Origin", "*")
	w.Write(jsonResp)
}

func main() {
    http.HandleFunc("/", ServeHTTP)
    http.ListenAndServe(":9090", nil)
}
```
- Se modifico el Header para corregir los errores de CORS y asi permitir todos los origenes.
- Se creo una nueva imagen **v2** y se aplicaron los cambios al Deployment y Services del BackEnd.

### Crear el Dockerfile para el Frontend

Archico `k8-hands-on/frontend/Dockerfile`:

```dockerfile
FROM nginx:alpine

COPY ./src/index.html /usr/share/nginx/html/index.html
```

- `docker build -t frontend-k8-hands-on:v1 -f Dockerfile .`ejecuto el Dockerfile para crear la imagen **frontend-k8-hands-on:v1**. La imagen la puedes encontar en DockerHub `ricardoandre97/frontend-k8s-hands-on:v1`.
- `docker run -d -p 80:80 --name frontend-k8-hands-on frontend-k8-hands-on:v1`ejecuto la imagen **frontend-k8-hands-on:v1** y expongo el puerto **80** de nuestro host para acceder a la aplicacion.
- `docker rm -fv frontend-k8-hands-on` para eliminar el contenedor si ya no lo necesito.

### Crear el manifiesto para el frontend

Archivo `k8-hands-on/frontend/frontend.yml:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-k8-hands-on
  labels:
    app: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: frontend-k8-hands-on:v1
          imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-k8-hands-on
  labels:
    app: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

- Agregar `imagePullPolicy: IfNotPresent` en la parte de `containers` del Deployment.
- `kubectl apply -f frontend.yml` crea el deployment y el servicio del frontend.
- Acceder al servicio en el navegador ingresando a `<service-ip>:<service-port>`.
- Si no se puede acceder posiblemente necesitas ver los servicios creados con `mikibube service --all` y de esta manera ver la url valida.
- Otra forma de acceder al servicio es creando un Pod preferiblemente de **nginx** en modo interactivo y hacer curl al services desde el Pod.

## Namespaces & Context

### Que es un Namespace

- Es una separacion logica que nos brinda un scope o limite. Es decir, nos permite agrupar un grupo de recursos y aislarlos dentro de un cluster.
- Permite tener diferentes proyectos en los diferentes Namespaces.
- Permite limitar los recursos en cada Namespaces, permitiendo configurar el numero de replicas permitidas, memoria, acceso o autorizaciones, etc.

### Namespaces por default

- `Kubectl get namespaces` lista los Namespaces.
- Si no se especifica el namespaces, el recurso se creara en el namespaces **default**.
- Al namespaces **kube-public** pueden acceder todos los usuarios sin importar sus permisos.
- El namespaces **kube-system** es el que usa kubernetes para solos objetos que usa para es sistema. Se recomienda no modificar nada de aqui.

### Comandos basicos

- `kubectl create namespaces <namespaces-name>`crea un namespaces.
- `kubectl get ns --show-labels` obtener los namespaces junto con sus labels.
-  `kubectl describe namespace <namespaces-name>` describir un namespace. 

- Archivo ns.yml:
  ```yml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: dev-ns
    labels:
      name: dev
  ```
- `kubectl apply -f ns.yml` crea un namspaces desde un archivo **YAML**.

### Objetos en un namespaces

- Necesitamos colocal la bandera `--namespace <namespace-name>` o `-n <namespace-name>` al momento de crear un nuevo objeto.
- Si usamos un archivo **YAML** debemos colocar `namespace: <namespaces-name>` en la metadata del objeto:
  ```yml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: dev
    labels:
      name: dev
  ---
  apiVersion: v1
  kind: Namespace
  metadata:
    name: prod
    labels:
      name: prod
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: deploy-dev
    namespace: dev
    labels:
      app: front
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: front
    template:
      metadata:
        labels:
          app: front
      spec:
        containers:
        - name: nginx
          image: nginx:alpine
          ports:
          - containerPort: 80
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: deploy-prod
    namespace: prod
    labels:
      app: back
  spec:
    replicas: 5
    selector:
      matchLabels:
        app: back
    template:
      metadata:
        labels:
          app: back
      spec:
        containers:
        - name: nginx
          image: nginx:alpine
          ports:
          - containerPort: 80
  ```

### DNS en los servicios del namespace

Se tiene este template:

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: ci
  labels:
    name: ci
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-k8-hands-on
  namespace: ci
  labels:
    app: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: k8-hands-on:v2
          imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: backend-k8-hands-on
  namespace: ci
  labels:
    app: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9090
```

Para acceder al servicio desde otro namespaces se debe especificar `<svcName>.<nsName>.svc.cluster.local`. Por ejemplo, para acceder desde un pod de otro namespace al servicio **backend-k8-hands-on** y namespace **ci** usando `curl`, seria `curl backend-k8-hands-on.ci.svc.cluster.local`.

### Contextos

- Tutorial de [set-context](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_config/kubectl_config_set-context/) y [use-context](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_config/kubectl_config_use-context/).
- `kubectl config view` ver la configuracion de kubectl.
- `kubectl config current-context` ver el context actual.
- `kubectl config set-context <context-name> --namespace=<namespace-name> --cluster=<cluster-name> --user=<user-name>` crear un nuevo context, para esto se debe tener en cuenta la informacion de la configuracion de kubectl.
- `kubectl config use-context <context-name>` usar un context. De esta manera nos ahorramos escribir `--namespace <namespace-name>` o `-n <namespace-name>` cada vez que ejecutamos un comando.

## Limitar la Ram y CPU en los Pods

- Es importante limitar los recursos para que los 
Pods no consuman todos los recursos disponibles en el nodo.
- Se pueden aplicar limites en Ram y CPU:
  - La Ram se puede limitar en Bytes, MB, GB, etc. Por ejemplo `50Mi => 50MB`
  - En CPUs tenemos que **1 CPU = 1000 mili CPUs = 1000 mili Cores**. Por lo tanto se puede limitar en porcentajes `0.1 => 10%` o en mili CPUs/Cores `100m => 100CPUs/Cores`

### Limits & Request

- **Request** es la cantidad de recurspos que el pod siempre va a poder disponer.
- **Limit** es la cantidad de recursos limite que el pod va a poder disponer.
- Limit es mayor que Request, si hay recursos en el Nodo, el Pod va a disponer de esta diferencia de recursos no garantizada.
- Cuando un Pod supera el Limite, kubernetes eliminara o reiniciara el Pod dependiendo de las politicas configuradas

#### Limit Ram

Archivo **limit-ram.yml**:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: limit-ram
spec:
  containers:
    - name: memory-demo
      image: polinux/stress
      resources:
        requests:
          memory: "100Mi"
        limits:
          memory: "200Mi"
      command: ["stress"]
      args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

- Si el Pod supera el limite tendra un estado **OOMKilled**, indicando que supero el limite de memoria.
- Si le asisgno un Request mucho mayor al disponible en el nodo, el estado sera **Pending**.
- Para ver las causas se recomienda ejecutar `kubectl logs/describe`.

#### Limit CPU

Archivo **limit-cpu.yml**:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: limit-cpu
spec:
  containers:
  - name: cpu-demo
    image: vish/stress
    resources:
      requests:
        cpu: "1"
      limits:
        cpu: "0.5"
    args:
    - -cpus
    - "2"
```

- Kubernetes asegura que no se sobrepase el limite de CPU disponible, por lo que si el POd supera el limite no habra advertencia o consecuencia.
- Si le asisgno un Request mucho mayor al disponible en el nodo, el estado sera **Pending**.

### QoS Classes

- **Guaranteed**: tienen menos probabilidades de enfrentar la evacuación.
  - Cada Contenedor en el Pod debe tener un **limit** de memoria y un **request** de memoria.
  - Para cada Contenedor en el Pod, el **limit** de memoria debe ser igual al **request** de memoria.
  - Cada Contenedor en el Pod debe tener un **limit** de CPU y un **request** de CPU.
  - Para cada Contenedor en el Pod, el **limit** de CPU debe ser igual al **request** de CPU.
- **Burstable**: el Pod puede intentar usar cualquier cantidad de recursos del nodo.
  - El Pod no cumple con los criterios para la clase de QoS **Guaranteed**.
  - Al menos un Contenedor en el Pod tiene un **request** o **limit** de memoria o CPU.
- **BestEffort**: usa todos los recursos disponibles, por lo que kubelet prefiere evacuar los Pods de **BestEffort** si el nodo se enfrenta a presión de recursos.
  - Un Pod tiene una clase de QoS de **BestEffort** si no cumple con los criterios ni para **Guaranteed** ni para **Burstable**. 
  - En otras palabras, un Pod es **BestEffort** solo si ninguno de los Contenedores en el Pod tiene un **limit** de memoria o un **request** de memoria, y ninguno de los Contenedores en el Pod tiene un **limit** de CPU o un **request** de CPU. Los Contenedores en un Pod pueden solicitar otros recursos y aún así ser clasificados como Mejor Esfuerzo.

## Seccion 12: LimitRange

### Que es un LimitRange

- Son limites por defecto que puedo configurarle a un Pod (A nivel de Objeto) en caso que este no tenga especificado una configuracion.
- Permite controlar el minimo o maximo de recursos para un objeto.
- El LimitRange solo se aplica en el Namespaces donde se encuentra.

### Configuracion

Archivo **limit-range.yml**:

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  labels:
    name: dev
---
apiVersion: v1
kind: LimitRange
metadata:
  name: men-cpu-limit-range
  namespace: dev
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 1
    defaultRequest:
      memory: 256Mi
      cpu: 0.5
    type: Container
---
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: dev
spec:
  containers:
    - name: container1
      image: nginx:alpine
```

- **type** infica a que objeto se aplica el LimitRange.
- `kubectl apply/get/describe/logs` Podemos usar los comandos tradicionales con el objeto `LimitRange` o `limits`.

### LimitRange con valores Minimos y Maximos

Archivo **min-max-limit.yml**:

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: prod
  labels:
    name: prod
---
apiVersion: v1
kind: LimitRange
metadata:
  name: min-max
  namespace: prod
spec:
  limits:
  - max:
      memory: 1Gi
      cpu: 1
    min:
      memory: 500Mi
      cpu: 100m
    type: Container
---
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: prod
spec:
  containers:
    - name: container1
      image: nginx:alpine
      resources:
        limits:
          memory: 500Mi
          cpu: 8.5
        request:
          memory: 400Mi
          cpu: 8.3
```

- Si un contenedor supera el min o max tanto en request o limit, el contenedor no sera creado y saldra un error de lo que ocurre.
- Si algún contenedor en ese Pod no especifica su propia **request** y **limit** de memoria, el plano de control asigna el **request** y **limit** de memoria predeterminados a ese contenedor. Ademas, Verifica que cada contenedor en ese Pod solicite la cantidad de memoria especificaca entre el **min** y **max**. Igualmente sucede con el **request** y **limit** de CPU.

## Seccion 13: ResourceQuota

### Que es un ResourceQuota

- A diferencia del LimitRange este se aplica a nivel de Namespaces; es decir, limita el uso de recursos  o numero de Pods en un Namespace.
- Si se supera el limite de recursos del ResourceQuota, kubernetes no creara mas objetos.

### Estructura de ResourceQuota

Archivo **res-quota.yml**:

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: uat
  labels:
    name: uat
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: res-quota
  namespace: uat
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: uat
  name: dp-quota
  labels:
    app: front
spec:
  replicas: 2
  selector:
    matchLabels:
      app: pod-quota
  template:
    metadata:
      labels:
        app: pod-quota
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        resources:
          requests:
            memory: 500Mi
            cpu: 0.5
          limits:
            memory: 500Mi
            cpu: 1
```

- Es obligatorio tener un **limit** y **request**  en cada contenedor.
- `kubectl apply/get/describe/logs` Podemos usar los comandos tradicionales con el objeto `ResourceQuota` o `quota`.
- `kubectl describe ns <namespace-name>` para ver que recursos estan usados de los disponibles en el namespace.

### Limitar numero de Pods

archivo **pod-quota.yml**:

```yml
apiVersion: v1
kind: Namespace
metadata:
  name: qa
  labels:
    name: qa
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: res-quota
  namespace: qa
spec:
  hard:
    pods: 3
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: qa
  name: dp-quota
  labels:
    app: front
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod-quota
  template:
    metadata:
      labels:
        app: pod-quota
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
```

- si se exede el numero de pods, no se crearan los nuevos.

## Seccion 14: Health Checks & Probes

### Que son los Probes

- Verifica periodicamente si un contenedor esta funcionando correctamente; es decir, hace un diagnostico periodicamente.
- Existen 3 formas de ejecutarlo:
  1. Mediante un **comando** y si devuelve 0 esta bien.
  2. Mediante TCP verifica si el puerto del contenedor esta funcionando.
  3. Mediante HTTP hace una solicitud al contenedor, si la respuesta es exitosa el contenedor esta funcionando.

  ### Typos de Probes

  #### Liveness

  - Diagnostico para verificar si la aplicacion esta funcionando correctamente.

  #### Readiness

  Verifica si la aplicacion ya inicio correctamente.

  #### Startup

  Para aplicaciones demoradas en iniciar.