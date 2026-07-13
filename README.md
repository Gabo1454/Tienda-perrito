# Tienda Perritos - EFT DevOps

## Descripción General

Tienda Perritos es una aplicación web desarrollada como proyecto académico para demostrar la implementación de una solución moderna basada en contenedores, Kubernetes y servicios Cloud de AWS.

La aplicación permite administrar un catálogo de productos para mascotas mediante operaciones CRUD (Crear, Leer, Actualizar y Eliminar), permitiendo visualizar, registrar, modificar y eliminar productos desde una interfaz web sencilla.

Este proyecto fue desarrollado para aplicar conceptos relacionados con:

* Arquitectura de software.
* Contenedorización con Docker.
* Orquestación de contenedores con Kubernetes.
* Despliegue en Amazon EKS.
* Automatización CI/CD mediante GitHub Actions.
* Monitoreo y observabilidad con Amazon CloudWatch.

---

## Evolución del Proyecto

Este proyecto corresponde a la Evaluación Final Transversal (EFT), que consolida y profundiza el trabajo desarrollado en las evaluaciones parciales anteriores.

### Evaluación 2

Arquitectura tradicional de tres capas desplegada sobre Amazon EC2, donde cada componente se ejecutaba en una instancia independiente:

* Capa Web (Frontend)
* Capa Aplicación (Backend)
* Capa Datos (MySQL)

Despliegue mediante contenedores Docker y un pipeline CI/CD básico (GitHub Actions → ECR → EC2).

### Evaluación 3 / EFT

La solución fue modernizada mediante Kubernetes y Amazon EKS, reemplazando la administración manual de instancias EC2 por una plataforma de orquestación de contenedores. La EFT profundiza específicamente en la automatización completa del ciclo CI/CD sobre esta arquitectura.

### Comparación entre EV2 y EV3/EFT

| Característica              | EV2 - Arquitectura 3 Capas             | EV3/EFT - Kubernetes y EKS               |
| ---------------------------- | --------------------------------------- | ------------------------------------------ |
| Infraestructura               | 3 instancias EC2                        | Clúster Amazon EKS (`cluster-eks`)         |
| Despliegue                    | Contenedores Docker en EC2              | Pods administrados por Kubernetes          |
| Orquestación                  | No disponible                           | Kubernetes                                 |
| Registro de imágenes          | Amazon ECR                              | Amazon ECR                                 |
| Pipeline CI/CD                | GitHub Actions → ECR → EC2              | GitHub Actions → ECR → EKS                 |
| Escalamiento                  | Manual                                  | Automático mediante HPA                    |
| Recuperación ante fallas      | Reinicio manual de contenedores         | Auto-Healing de Kubernetes                 |
| Balanceo de carga             | Configuración manual                    | LoadBalancer administrado por Kubernetes   |
| Monitoreo                     | Servicios básicos de AWS                | CloudWatch + métricas del clúster          |
| Actualización de versiones    | Actualización de contenedores en EC2    | Rollout automático de Deployments          |

### Mejoras Incorporadas

* Escalamiento automático mediante Horizontal Pod Autoscaler (HPA).
* Recuperación automática ante fallas (Auto-Healing).
* Balanceo de carga mediante Service de tipo LoadBalancer.
* Administración centralizada de contenedores mediante un único Managed Node Group.
* Entorno de desarrollo local reproducible mediante Docker Compose.
* Pipeline CI/CD extendido hasta el despliegue en EKS (build → push → deploy).

---

## Funcionalidades

* Consultar productos registrados.
* Agregar nuevos productos.
* Modificar información existente.
* Eliminar productos.
* Persistir información en una base de datos MySQL.
* Acceder al sistema mediante una interfaz web.

---

## Arquitectura de la Solución

```text
                 Internet
                     │
                     ▼
            Internet Gateway
                     │
                     ▼
             Load Balancer (subredes públicas)
                     │
                     ▼
              Frontend Pod  ──▶  Backend Pod  ──▶  MySQL Pod
                     (subredes privadas, mismo Managed Node Group)

                 Amazon EKS — cluster-eks
```

Cada componente se ejecuta como Pods administrados por Kubernetes, distribuidos sobre un único **Managed Node Group** (`nodo-eks`, capacidad Spot, t3.large, escalado 1-3), sin nodos dedicados por componente: es Kubernetes quien decide en qué nodo del pool compartido programa cada pod.

### Capa de Presentación (Frontend)
**Tecnologías:** HTML, CSS, JavaScript, Nginx
**Responsabilidades:** interfaz gráfica, interacción con el usuario, consumo de la API REST del backend vía `tienda-backend:3001`.

### Capa de Aplicación (Backend)
**Tecnologías:** Node.js, Express
**Responsabilidades:** lógica de negocio, validación, operaciones CRUD, endpoints REST, conexión a MySQL vía `tienda-db:3306`.

### Capa de Datos (Base de Datos)
**Tecnologías:** MySQL 8
**Responsabilidades:** persistencia del catálogo de productos.

---

## Flujo de Funcionamiento

1. El usuario accede a la aplicación web.
2. El frontend envía solicitudes HTTP al backend (`tienda-backend:3001`).
3. El backend procesa la solicitud.
4. El backend consulta o modifica información en MySQL (`tienda-db:3306`).
5. La respuesta es enviada al frontend.
6. El usuario visualiza los resultados.

La comunicación entre componentes se resuelve siempre por **nombre de servicio** (DNS interno), nunca por IP fija — tanto en desarrollo local (red de Docker Compose) como en producción (CoreDNS de Kubernetes).

---

## Infraestructura AWS

* **Amazon EKS** — clúster `cluster-eks`
* **Amazon ECR** — repositorios `tienda-frontend`, `tienda-backend`, `tienda-db`
* **Amazon VPC** — `10.0.0.0/20`, 2 subredes públicas + 4 privadas, distribuidas en 2 Zonas de Disponibilidad
* **Internet Gateway** — salida/entrada de tráfico público de la VPC
* **NAT Gateway** — uno por Zona de Disponibilidad, para tráfico saliente de los nodos en subredes privadas
* **AWS IAM** — roles `LabEksClusterRole` y `LabEksNodeRole`
* **Elastic Load Balancer** — expuesto vía Service tipo `LoadBalancer` (`tienda-frontend`)
* **Amazon CloudWatch** — métricas y logs del plano de control del clúster

---

## Contenedorización con Docker

Cada componente se ejecuta en su propio contenedor, con imágenes independientes:

* `tienda-frontend`
* `tienda-backend`
* `tienda-db`

**Beneficios:** portabilidad, aislamiento, facilidad de despliegue, consistencia entre entornos.

### Desarrollo local con Docker Compose

Se incluye un archivo `docker-compose.yml` en la raíz del proyecto, que levanta los tres servicios en una red interna común, replicando localmente la misma lógica de comunicación por nombre de servicio que se usa después en EKS (`tienda-backend`, `tienda-db`). Esto permite validar la integración completa del sistema antes de publicar imágenes en ECR y desplegar en el clúster.

```bash
docker compose up --build
```

Luego abrir `http://localhost` en el navegador.

---

## Kubernetes y Amazon EKS

### Clúster y Node Group

* Clúster: `cluster-eks`
* Managed Node Group: `nodo-eks` (Spot, t3.large, mínimo 1 / deseado 1 / máximo 3)

### Namespace

```text
tienda
```

### Deployments

```text
tienda-frontend
tienda-backend
tienda-db
```

### Services

```text
tienda-frontend (LoadBalancer)
tienda-backend  (ClusterIP)
tienda-db       (Headless Service)
```

### Beneficios Obtenidos

* Escalamiento automático.
* Alta disponibilidad.
* Balanceo de carga.
* Recuperación automática ante fallas.
* Administración centralizada de contenedores.

---

## Escalamiento Automático (HPA)

| Componente | Réplicas mín. | Réplicas máx. | Umbral CPU |
|---|---|---|---|
| Backend  | 2 | 10 | 70% |
| Frontend | 2 | 6  | 60% |

Cuando la utilización de CPU supera el umbral configurado, Kubernetes crea nuevos Pods automáticamente hasta el máximo definido; cuando la carga disminuye, reduce gradualmente hasta el mínimo.

---

## Auto-Healing

Si un Pod falla, el ReplicaSet del Deployment correspondiente detecta la pérdida y crea automáticamente un nuevo Pod, sin intervención manual, manteniendo la disponibilidad del servicio.

---

## Integración Continua y Despliegue Continuo (CI/CD)

Pipeline automatizado mediante GitHub Actions (`.github/workflows/deploy-eks.yml`).

### Flujo del pipeline

1. Push a la rama `main` (o ejecución manual vía `workflow_dispatch`).
2. Checkout del código.
3. Configuración de credenciales AWS (GitHub Secrets).
4. Login a Amazon ECR.
5. Generación del tag de imagen a partir del commit SHA.
6. Build y push de las tres imágenes a ECR.
7. Configuración de `kubectl` contra el clúster EKS.
8. Aplicación de manifiestos base (namespace, Secret de la BD, Deployment/Service de MySQL).
9. Actualización de los Deployments de backend y frontend (`apply` + `set image` + `rollout status`).
10. Aplicación condicional de los manifiestos de HPA.
11. Verificación final de Pods, Services y HPA.

### Evolución del Pipeline

**EV2:** `GitHub → Docker Build → ECR → EC2`
**EV3/EFT:** `GitHub → Docker Build → ECR → EKS → Kubernetes Rollout`

---

## Principios DevOps Aplicados

* Control de versiones mediante GitHub.
* Contenedorización mediante Docker, con entorno local reproducible vía Docker Compose.
* Infraestructura Cloud en AWS.
* Integración Continua (CI) y Despliegue Continuo (CD).
* Gestión de secretos mediante GitHub Secrets y Kubernetes Secrets.
* Observabilidad mediante CloudWatch.
* Automatización y autoescalado mediante Kubernetes.

---

## Monitoreo y Observabilidad

Amazon CloudWatch se utiliza para:

* Supervisar el estado del clúster y del plano de control.
* Registrar eventos del plano de control (api, audit, scheduler, controllerManager).
* Analizar métricas de CPU y memoria.
* Revisar logs de Kubernetes.

---

## Cómo Utilizar la Aplicación

1. Acceder a la URL pública del frontend (`kubectl get svc tienda-frontend -n tienda`).
2. Visualizar los productos registrados.
3. Crear nuevos productos mediante el formulario.
4. Modificar productos existentes.
5. Eliminar productos que ya no sean necesarios.

---

## Estructura del Proyecto

```text
tienda-perritos/
├── docker-compose.yml
├── frontend/
│   ├── index.html
│   ├── app.js
│   ├── Dockerfile
│   └── default.conf
├── backend/
│   ├── server.js
│   ├── package.json
│   └── Dockerfile
├── db/
│   ├── Dockerfile
│   └── init.sql
├── k8s/
│   ├── namespace.yaml
│   ├── mysql-secret.yaml
│   ├── mysql-deployment.yaml
│   ├── mysql-service.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-hpa.yaml
│   └── frontend-hpa.yaml
└── .github/
    └── workflows/
        └── deploy-eks.yml
```

---

## Tecnologías Utilizadas

* HTML, CSS, JavaScript
* Node.js, Express
* MySQL
* Docker, Docker Compose
* Kubernetes
* Amazon EKS, ECR, VPC, IAM, CloudWatch
* GitHub Actions

---

## Autor

Gabriel Andrés Águila Gormaz — Proyecto desarrollado con fines académicos para aplicar conceptos de Cloud Computing, DevOps, Docker, Kubernetes y servicios de Amazon Web Services (AWS).