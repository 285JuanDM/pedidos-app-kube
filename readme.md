# Parcial 1 Patrones arquitectonicos avanzados

## Descripción

Nuestro proyecto consiste en el despliegue de un sistema sencillo de gestion de pedidos, desde allí se peuden crear usuarios y ver la lista de pedidos ya creados, cuyos datos se almacenan mediante una conexión con PostgreSQL. El frontend de esta app hace uso de Vanilla HTML/CSS/JS mientras que el backend usa FastAPI, ambas partes son montadas como imagenes en Dockerhub, estas imagenes corren en un cluster en Kubernetes que en conjunto con ArgoCD garantizan una entrega continua y helm para el manejo de charts, además como repositorio de charts utilizamos ChartMuseum

### Funcionalidades clave de la app

- Creación de pedidos
- Ver lista de pedidos
- Almacenamiento de datos en PostgreSQL

## Tecnologías utilizadas

- FastAPI
- HTML/CSS/JS
- PostgreSQL
- Docker
- Kubernetes
- Helm Charts
- ChartMuseum
- ArgoCD

## Imagenes

- Frontend: 285juandm/frontend:1.0.1
- Backend: 285juandm/backend:1.0.0

## Lista de endpoints de acceso

- /health
- /api/users **→ Backend**
- /  **→ Frontend**

# Instalación del chart con helm

- _Aclaración:_ Debe haber una instalación y configuración previa de KubeCTL y helm

## Contenido del chart:

- templates: YAML Para backend y frontend, asi como un subchart db para PostgreSQL
- values.yaml: Configuraciones de puerto, las imagenes de frontend y backend así como las variables de entorno previamente mencionadas

# Instalación manual del chart de Helm

A continuación se describen los pasos para instalar el chart de Helm de manera manual usando ChartMuseum como repositorio de charts.

## Requisitos previos

- Kubectl configurado para acceder al cluster de Kubernetes.
- Helm.
- Acceso al ChartMuseum donde se encuentran los charts.

### 1. Añadir el repositorio de Helm (ChartMuseum)

```bash
helm repo add pedidos-app http://174.138.111.130:8080
helm repo update
```

### 2. Crear el namespace (si no existe)

```bash
kubectl create namespace prod
```

### 3. Instalar el chart en Kubernetes

```bash
helm install pedido-app-prod pedidos-app/pedidos-app --namespace prod --values values-prod.yaml
```

# Cómo está configurado ArgoCD para sincronizar


## Configuración de Argo CD para sincronización automática

- Cada aplicación (`Application`) en Argo CD tiene activado **Auto-Sync** mediante:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
````

* Esto permite que cualquier cambio detectado en el repositorio de origen se aplique automáticamente al cluster sin intervención manual.

* En las aplicaciones, se configuró `targetRevision: "*"`, indicando que Argo CD siempre toma la **última versión disponible** del chart en el repositorio (ChartMuseum).

* Flujo de actualización:

  1. Se sube una nueva versión del chart o se actualizan valores (por ejemplo, imagen de un pod) en ChartMuseum.
  2. Argo CD detecta el cambio y, al hacer **Refresh** o mediante Auto-Sync, aplica la actualización.
  3. Los pods, servicios, ingress, HPAs y demás recursos de Kubernetes se actualizan automáticamente según lo definido en el chart.

* Este mecanismo garantiza que cualquier cambio en el repositorio de charts se refleje en el cluster sin necesidad de ejecutar comandos `kubectl` o `helm` manualmente, permitiendo el despliegue continuo.
