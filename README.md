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

Servicio que permite a un usuario comunicarse con kubernetes y darle ordenes como `kubectl run`, `kubectl apply`, `kubectl delete`, `kubectl get`, etc. Tambien se puede mediante archivos **JSON**.

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
