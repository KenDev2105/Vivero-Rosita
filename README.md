#  Plataforma Web para "Viveros Rosita"

Landing page de promoción para **Viveros Rosita**, desplegada con **Docker**, servida por **Apache con HTTPS** y distribuida en un **clúster con balanceo de carga Nginx**, para mantenerse estable incluso en temporadas de alta demanda.

> **Tipo de proyecto:** Negocio real

---

##  Tabla de contenidos

1. [Equipo](#-equipo)
2. [Descripción del proyecto](#-descripción-del-proyecto)
3. [Arquitectura](#-arquitectura)
4. [Tecnologías](#-tecnologías)
5. [Estructura del proyecto](#-estructura-del-proyecto)
6. [Requisitos previos](#-requisitos-previos)
7. [Instalación y ejecución](#-instalación-y-ejecución)
8. [Verificación del clúster](#-verificación-del-clúster)
9. [Alcance por unidad](#-alcance-por-unidad)

---

##  Equipo

| Integrante |
|---|
| Oscar Stanley Marquez Portillo |
| Kevin Alejandro Granillo Soto |
| Kennet Jared Mejia Portillo |
| Anderson Isaac Ortiz Canizález |
| José Aristides Zelaya Parada |

---

##  Descripción del proyecto

### ¿A quién ayuda el sitio?

A los administradores de Viveros Rosita y a sus clientes. Ayuda a los dueños a promocionar sus productos y especies botánicas, y a los clientes a conocer y ver las flores, plantas y suministros de jardinería que ofrece el vivero.

### ¿Qué problema concreto resuelve?

Resuelve la falta de visibilidad digital del vivero. Actualmente, los clientes deben visitar el local para conocer los productos y flores disponibles.

El sitio es una landing page de promoción que presenta los productos de forma atractiva y, gracias a la infraestructura en clúster, se mantiene **estable y disponible** incluso ante picos de tráfico en temporadas de alta demanda (como el Día de la Madre o la primavera).

---

##  Arquitectura

```
                    ┌──────────────────────┐
   Cliente  ───────►│   Nginx (Balanceador) │
  (navegador)       │    + Health checks    │
                    └─────────┬────────────┘
                              │
                 ┌────────────┴────────────┐
                 ▼                         ▼
        ┌────────────────┐        ┌────────────────┐
        │  Nodo 1        │        │  Nodo 2        │
        │  Apache + HTTPS│        │  Apache + HTTPS│
        │  index.html    │        │  index.html    │
        │  style.css     │        │  style.css     │
        └────────────────┘        └────────────────┘
              (contenedores Docker, orquestados con Docker Compose)
```

### Componentes

- **Contenedor y sitio web:** el sitio es una landing page de una sola página (`index.html` + `style.css`) enfocada en la promoción de productos y la visualización de flores. Docker empaqueta el sitio junto con su servidor web para que funcione igual en cualquier equipo o entorno. El contenedor aísla la aplicación del resto del sistema, facilita su instalación y permite crear copias idénticas rápidamente, base para levantar varios nodos en el clúster.
- **Servidor web y HTTPS:** el sitio se sirve mediante **Apache** configurado con **HTTPS** usando un **certificado autofirmado** para el entorno local.
- **Clúster y balanceo de carga:** un entorno con **Docker Compose** levanta **al menos 2 nodos** del sitio, administrados por un balanceador **Nginx** con **health checks** para verificar el estado de los contenedores.

---

##  Tecnologías

| Tecnología | Uso |
|---|---|
| HTML5 / CSS3 | Landing page (`index.html`, `style.css`) |
| Apache HTTP Server | Servidor web de cada nodo |
| OpenSSL | Generación del certificado autofirmado |
| Docker | Contenedorización del sitio y su servidor |
| Docker Compose | Orquestación del clúster |
| Nginx | Balanceador de carga y health checks |

---

##  Estructura del proyecto

> Estructura sugerida; ajústala a la organización real del repositorio.

```
viveros-rosita/
├── docker-compose.yml        # Define los nodos web y el balanceador
├── web/
│   ├── Dockerfile            # Imagen Apache + sitio + certificado
│   ├── index.html            # Landing page
│   ├── style.css             # Estilos
│   └── certs/                # Certificado autofirmado (local)
├── nginx/
│   └── nginx.conf            # Balanceo de carga y health checks
└── README.md
```

---

##  Requisitos previos

- [Docker](https://docs.docker.com/get-docker/) (v20.10 o superior)
- [Docker Compose](https://docs.docker.com/compose/install/) (v2 o superior)
- Git
- OpenSSL (solo si deseas regenerar el certificado autofirmado)

---

##  Instalación y ejecución

### 1. Clonar el repositorio

```bash
git clone <URL_DEL_REPOSITORIO>
cd viveros-rosita
```

### 2. Generar el certificado autofirmado 

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout web/certs/server.key \
  -out web/certs/server.crt \
  -subj "/CN=localhost"
```

### 3. Levantar el clúster

```bash
docker compose up --build -d
```

### 4. Acceder al sitio

Abre en tu navegador:

```
https://localhost
```

>  Al usar un certificado autofirmado, el navegador mostrará una advertencia de seguridad. Es el comportamiento esperado en el entorno local; acepta la excepción para continuar.

### 5. Detener el clúster

```bash
docker compose down
```

---

##  Verificación del clúster

**Ver el estado de los contenedores:**

```bash
docker compose ps
```

**Ver los logs del balanceador:**

```bash
docker compose logs -f nginx
```

**Probar la tolerancia a fallos:** detén un nodo y comprueba que el sitio sigue disponible.

```bash
docker compose stop <nombre_del_nodo_1>
curl -k https://localhost
```

El balanceador, gracias a los health checks, dejará de enviar tráfico al nodo caído y redirigirá las peticiones al nodo activo.

---

##  Alcance por unidad

| Unidad | Componente |
|---|---|
| **Unidad 1** | Sitio web estático y contenedorización con Docker |
| **Unidad 2** | Servidor Apache con HTTPS (certificado autofirmado) |
| **Unidad 3** | Clúster con Docker Compose, balanceo con Nginx y health checks |

---
