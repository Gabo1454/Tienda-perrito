# Tienda Perritos - EV3

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

Este proyecto corresponde a la tercera etapa de desarrollo de Tienda Perritos.

### Evaluación 2

En la Evaluación 2 se implementó una arquitectura tradicional de tres capas desplegada sobre Amazon EC2, donde cada componente se ejecutaba en una instancia independiente:

* Capa Web (Frontend)
* Capa Aplicación (Backend)
* Capa Datos (MySQL)

El despliegue se realizaba mediante contenedores Docker y un pipeline CI/CD basado en GitHub Actions.

### Evaluación 3

En la Evaluación 3 la solución fue modernizada mediante Kubernetes y Amazon EKS, reemplazando la administración manual de instancias EC2 por una plataforma de orquestación de contenedores capaz de proporcionar:

* Escalabilidad automática.
* Alta disponibilidad.
* Recuperación automática ante fallas.
* Despliegues automatizados.
* Administración centralizada de la infraestructura.

### Comparación entre EV2 y EV3

| Característica             | EV2 - Arquitectura 3 Capas                           | EV3 - Kubernetes y EKS                            |
| -------------------------- | ---------------------------------------------------- | ------------------------------------------------- |
| Infraestructura            | 3 instancias EC2 (Frontend, Backend y Base de Datos) | Clúster Amazon EKS                                |
| Despliegue                 | Contenedores Docker ejecutados en EC2                | Pods administrados por Kubernetes                 |
| Orquestación               | No disponible                                        | Kubernetes                                        |
| Registro de imágenes       | Amazon ECR                                           | Amazon ECR                                        |
| Pipeline CI/CD             | GitHub Actions → ECR → EC2                           | GitHub Actions → ECR → EKS                        |
| Escalamiento               | Manual                                               | Automático mediante HPA                           |
| Recuperación ante fallas   | Reinicio manual de contenedores                      | Auto-Healing de Kubernetes                        |
| Balanceo de carga          | Configuración manual                                 | LoadBalancer administrado por Kubernetes          |
| Monitoreo                  | Servicios básicos de AWS                             | CloudWatch + métricas del clúster                 |
| Administración             | Gestión individual de servidores                     | Administración centralizada mediante Kubernetes   |
| Disponibilidad             | Dependiente de cada instancia EC2                    | Mayor disponibilidad mediante replicación de Pods |
| Actualización de versiones | Actualización de contenedores en EC2                 | Rollout automático de Deployments                 |

### Mejoras Incorporadas en EV3

La Evaluación 3 toma como base la solución desarrollada en la Evaluación 2 y reemplaza la arquitectura tradicional de tres capas desplegada sobre Amazon EC2 por una plataforma basada en Kubernetes administrada mediante Amazon EKS.

Las principales mejoras obtenidas son:

* Escalamiento automático mediante Horizontal Pod Autoscaler (HPA).
* Recuperación automática ante fallas (Auto-Healing).
* Balanceo de carga mediante Services de tipo LoadBalancer.
* Administración centralizada de contenedores.
* Mayor disponibilidad y resiliencia de la aplicación.
* Mejor integración con prácticas DevOps.
* Despliegues automatizados y controlados.
* Capacidad de crecimiento sin administrar manualmente servidores EC2.

---

## Objetivo del Proyecto

El objetivo principal es demostrar cómo una aplicación moderna puede ser desplegada y administrada utilizando tecnologías Cloud-Native, permitiendo:

* Escalabilidad automática.
* Alta disponibilidad.
* Recuperación automática ante fallas.
* Automatización del despliegue.
* Monitoreo centralizado.

---

## Funcionalidades

La aplicación permite:

* Consultar productos registrados.
* Agregar nuevos productos.
* Modificar información existente.
* Eliminar productos.
* Persistir información en una base de datos MySQL.
* Acceder al sistema mediante una interfaz web.

---

## Arquitectura Actual de la Solución

La aplicación se ejecuta sobre un clúster de Amazon EKS utilizando Kubernetes como plataforma de orquestación.

Aunque la lógica del sistema continúa separada en Frontend, Backend y Base de Datos, estos componentes ya no se encuentran distribuidos en instancias EC2 independientes como en la Evaluación 2.

Actualmente cada componente se ejecuta como Pods administrados por Kubernetes dentro de un clúster EKS.

### Arquitectura General

```text
                 Internet
                     │
                     ▼
             Load Balancer
                     │
                     ▼
              Frontend Pod
                     │
                     ▼
              Backend Pod
                     │
                     ▼
                MySQL Pod

                 Amazon EKS
```

### Capa de Presentación (Frontend)

**Tecnologías utilizadas**

* HTML
* CSS
* JavaScript
* Nginx

**Responsabilidades**

* Mostrar la interfaz gráfica.
* Permitir la interacción del usuario.
* Consumir los servicios REST expuestos por el backend.

### Capa de Aplicación (Backend)

**Tecnologías utilizadas**

* Node.js
* Express

**Responsabilidades**

* Implementar la lógica de negocio.
* Validar solicitudes.
* Gestionar operaciones CRUD.
* Exponer endpoints REST.

### Capa de Datos (Base de Datos)

**Tecnologías utilizadas**

* MySQL

**Responsabilidades**

* Almacenar la información de los productos.
* Garantizar la persistencia de los datos.

---

## Flujo de Funcionamiento

1. El usuario accede a la aplicación web.
2. El frontend envía solicitudes HTTP al backend.
3. El backend procesa la solicitud.
4. El backend consulta o modifica información en MySQL.
5. La respuesta es enviada al frontend.
6. El usuario visualiza los resultados.

---

## Infraestructura AWS

La solución fue desplegada utilizando los siguientes servicios:

* Amazon EKS (Elastic Kubernetes Service)
* Amazon ECR (Elastic Container Registry)
* Amazon VPC
* AWS IAM
* Elastic Load Balancer (ELB)
* Amazon CloudWatch

---

## Contenedorización con Docker

Cada componente del sistema se ejecuta dentro de su propio contenedor Docker.

**Imágenes utilizadas**

* tienda-frontend
* tienda-backend
* tienda-db

**Beneficios**

* Portabilidad.
* Aislamiento.
* Facilidad de despliegue.
* Consistencia entre entornos.

---

## Kubernetes y Amazon EKS

Para administrar los contenedores se utilizó Kubernetes mediante Amazon EKS.

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
tienda-backend (ClusterIP)
tienda-db (ClusterIP)
```

### Beneficios Obtenidos

* Escalamiento automático.
* Alta disponibilidad.
* Balanceo de carga.
* Recuperación automática ante fallas.
* Administración centralizada de contenedores.

---

## Escalamiento Automático (HPA)

La aplicación utiliza Horizontal Pod Autoscaler (HPA).

Cuando aumenta la carga de trabajo:

* Kubernetes crea nuevos Pods automáticamente.

Cuando disminuye la carga:

* Kubernetes reduce la cantidad de Pods para optimizar recursos.

Esto permite utilizar eficientemente la infraestructura disponible.

---

## Auto-Healing

Kubernetes monitorea continuamente los contenedores.

Si un Pod falla:

1. Kubernetes detecta el problema.
2. El Deployment identifica la pérdida del Pod.
3. ReplicaSet crea automáticamente un nuevo Pod.
4. El servicio continúa disponible para los usuarios.

Este mecanismo mejora la disponibilidad y resiliencia de la aplicación.

---

## Integración Continua y Despliegue Continuo (CI/CD)

El proyecto implementa un pipeline automatizado utilizando GitHub Actions.

### Flujo CI/CD

1. Se realiza un cambio en el código fuente.
2. Se ejecuta un Push hacia la rama principal.
3. GitHub Actions inicia automáticamente el pipeline.
4. Se construyen las imágenes Docker.
5. Las imágenes son publicadas en Amazon ECR.
6. Kubernetes utiliza las nuevas imágenes almacenadas en ECR.
7. Se ejecuta un rollout controlado de los Deployments.
8. La nueva versión queda disponible para los usuarios.

### Evolución del Pipeline

**EV2**

```text
GitHub → Docker Build → ECR → EC2
```

**EV3**

```text
GitHub → Docker Build → ECR → EKS → Kubernetes Rollout
```

Esto permite actualizar los Pods automáticamente sin intervención manual.

---

## Principios DevOps Aplicados

* Control de versiones mediante GitHub.
* Contenedorización mediante Docker.
* Infraestructura Cloud en AWS.
* Integración Continua (CI).
* Despliegue Continuo (CD).
* Observabilidad mediante CloudWatch.
* Automatización mediante Kubernetes.

---

## Monitoreo y Observabilidad

Se utilizó Amazon CloudWatch para:

* Supervisar el estado del clúster.
* Registrar eventos del plano de control.
* Analizar métricas de CPU y memoria.
* Revisar logs de Kubernetes.
* Monitorear la salud general de la aplicación.

---

## Cómo Utilizar la Aplicación

1. Acceder a la URL pública del frontend.
2. Visualizar los productos registrados.
3. Crear nuevos productos mediante el formulario.
4. Modificar productos existentes.
5. Eliminar productos que ya no sean necesarios.

Todos los cambios realizados son almacenados automáticamente en la base de datos MySQL.

---

## Estructura del Proyecto

```text
frontend/
├── index.html
├── app.js
├── Dockerfile
└── default.conf

backend/
├── server.js
├── package.json
└── Dockerfile

db/
├── Dockerfile
└── scripts SQL

k8s/
├── namespace.yaml
├── mysql-deployment.yaml
├── mysql-service.yaml
├── backend-deployment.yaml
├── backend-service.yaml
├── frontend-deployment.yaml
├── frontend-service.yaml
├── backend-hpa.yaml
└── frontend-hpa.yaml

.github/
└── workflows/
    └── cicd-eks.yml
```

---

## Tecnologías Utilizadas

* HTML
* CSS
* JavaScript
* Node.js
* Express
* MySQL
* Docker
* Kubernetes
* Amazon EKS
* Amazon ECR
* AWS IAM
* Amazon CloudWatch
* GitHub Actions

---

## Autor

Proyecto desarrollado con fines académicos para aplicar conceptos de Cloud Computing, DevOps, Docker, Kubernetes y servicios de Amazon Web Services (AWS).
