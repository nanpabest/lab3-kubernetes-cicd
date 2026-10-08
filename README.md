LABORATORIO 3 - CI/CD SOBRE KUBERNETES
RESUMEN

Implementación de un flujo CI/CD para una aplicación NestJS utilizando Docker, Kubernetes y Jenkins con agentes Kubernetes efímeros.

Repositorio:
https://github.com/nanpabest/lab3-kubernetes-cicd.git

Pipeline:
install -> test -> build -> push -> deploy

Resultado final:
Finished: SUCCESS

STACK

Aplicación Original: NestJS / Node.js 24
Dependencias: pnpm
Pruebas: Jest
Contenedores: Docker
Orquestación: Kubernetes de Docker Desktop
CI/CD: Jenkins
SCM: GitHub
Registry principal: Docker Hub
Registry adicional validado: GitHub Container Registry

RECURSOS

Namespace:
ns-hernan-contreras

Deployment:
app-hernan-contreras

Service:
svc-hernan-contreras

ConfigMap:
config-hernan-contreras

Secret:
secret-hernan-contreras

Job Jenkins:
lab3-hernan-contreras

Imagen utilizada por Kubernetes:
hernancontreras/tarea-final:hernan-contreras

Imagen versionada:
hernancontreras/tarea-final:3.0.0

ARCHIVOS PRINCIPALES

Dockerfile
.dockerignore
Jenkinsfile
agent.yaml
entrega.yaml
evidencias/

VALIDACIÓN LOCAL

Instalar dependencias:

corepack enable

pnpm install --frozen-lockfile

Ejecutar pruebas:

pnpm test --runInBand

Construir aplicación:

pnpm build

Construir imagen Docker:

docker build -t hernancontreras/tarea-final:3.0.0 -t hernancontreras/tarea-final:hernan-contreras .

Publicar imagen:

docker push hernancontreras/tarea-final:3.0.0

docker push hernancontreras/tarea-final:hernan-contreras

KUBERNETES

Aplicar manifiestos:

kubectl apply -f entrega.yaml

Verificar Pods:

kubectl get pods -n ns-hernan-contreras

Verificar Deployment:

kubectl get deployment app-hernan-contreras -n ns-hernan-contreras -o wide

Verificar Service:

kubectl get svc svc-hernan-contreras -n ns-hernan-contreras

Verificar configuración inyectada:

kubectl exec -n ns-hernan-contreras deployment/app-hernan-contreras -- printenv | Select-String "AMBIENTE|API_KEY"

El Deployment mantiene 2 réplicas y utiliza:

hernancontreras/tarea-final:hernan-contreras

PRUEBA FUNCIONAL

Exponer temporalmente el Service:

kubectl port-forward svc/svc-hernan-contreras 8080:80 -n ns-hernan-contreras

Consultar:

curl.exe http://localhost:8080/lab

Durante la validación final se utilizó temporalmente el puerto local 8083 debido a que 8080 estaba ocupado:

kubectl port-forward svc/svc-hernan-contreras 8083:80 -n ns-hernan-contreras

curl.exe http://localhost:8083/lab

El endpoint /lab confirmó la lectura de:

AMBIENTE
API_KEY

JENKINS

Jenkins fue instalado dentro del clúster mediante Helm:

helm repo add jenkins https://charts.jenkins.io

helm repo update

kubectl create namespace jenkins

helm install jenkins jenkins/jenkins -n jenkins

Acceso local:

kubectl --namespace jenkins port-forward svc/jenkins 9090:8080

URL:

http://localhost:9090

KUBERNETES CLOUD

URL:

https://kubernetes.default

Namespace:

jenkins

Conexión validada:

Connected to Kubernetes v1.36.1

La autenticación se configuró mediante una credencial Jenkins asociada al ServiceAccount jenkins.

Generación del token:

kubectl create token jenkins -n jenkins --duration=24h

ID de credencial:

kubernetes-jenkins-token

La CA del clúster utilizada por Jenkins se obtuvo mediante:

kubectl exec -n jenkins jenkins-0 -c jenkins -- cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt

CREDENCIALES

Docker Hub:

ID:
dockerhub-credentials

Tipo:
Username with password

Usuario:
hernancontreras

Password:
Docker Hub Personal Access Token

Los secretos no se encuentran escritos directamente en Jenkinsfile.

RBAC

El ServiceAccount:

system:serviceaccount:jenkins:jenkins

dispone de permisos namespace-scoped para operar el Deployment.

Validación:

kubectl auth can-i get deployments.apps -n ns-hernan-contreras --as=system:serviceaccount:jenkins:jenkins

kubectl auth can-i patch deployments.apps -n ns-hernan-contreras --as=system:serviceaccount:jenkins:jenkins

Resultado:

yes

AGENTE JENKINS

agent.yaml define un Pod efímero con:

node:24-alpine
docker:27-cli
docker:27-dind
bitnami/kubectl:latest
jnlp

Responsabilidades:

node:
install / test

docker:
build / push

dind:
Docker daemon

kubectl:
deploy

El contenedor kubectl utiliza:

runAsUser: 1000
runAsGroup: 1000

PIPELINE

install:
corepack enable
pnpm install --frozen-lockfile

test:
pnpm test --runInBand

Resultado validado:
2 suites aprobadas
5 tests aprobados

build:
construcción de los tags 3.0.0 y hernan-contreras

push:
publicación en Docker Hub utilizando Jenkins Credentials

deploy:
actualización y rollout del Deployment Kubernetes

JOB

Nombre:
lab3-hernan-contreras

Definición:
Pipeline script from SCM

Repositorio:
https://github.com/nanpabest/lab3-kubernetes-cicd.git

Branch:
*/main

Script Path:
Jenkinsfile

VALIDACIÓN FINAL

Deployment:

READY: 2/2
UP-TO-DATE: 2
AVAILABLE: 2

Imagen:

hernancontreras/tarea-final:hernan-contreras

Pods:

2 Running

Pipeline:

Finished: SUCCESS

Endpoint:

/lab operativo

EVIDENCIAS

evidencias/01-cluster-info.txt
evidencias/02-nodes.txt
evidencias/03-pods.txt
evidencias/04-deployment.txt
evidencias/05-service.txt
evidencias/06-logs.txt
evidencias/07-printenv.txt
evidencias/08-configmap.txt
evidencias/09-secret.txt
evidencias/10-port-forward.txt
evidencias/11-curl-lab.txt
evidencias/12-pods-post-jenkins.txt
evidencias/13-deployment-post-jenkins.txt
evidencias/14-printenv-post-jenkins.txt
evidencias/15-curl-lab-post-jenkins.txt
log-pipeline-jenkins.txt