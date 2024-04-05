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

Se encarga de crear el contenedor en el lugar correcto. Por ejemplo, encuentra en que nodo debe crear el contenedor.