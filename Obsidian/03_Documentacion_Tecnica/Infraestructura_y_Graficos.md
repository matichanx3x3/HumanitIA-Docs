# Infraestructura y Gráficos

El siguiente diagrama detalla la orquestación general actual de microservicios usando Docker Compose:

```mermaid
flowchart TD
    %% Contenedores e Integraciones
    subgraph Edge Network [Red Bridge: edge_network]
        direction TB
        
        subgraph Capa_Ingesta [Capa 1: Ingesta y Gateway]
            MQTT[Mosquitto Broker\nPuerto: 1883 / 9001]
            Worker[Worker Ingesta\nPython / SQLAlchemy]
        end
        
        subgraph Capa_Logica [Capa 3: Core Lógico y API]
            API[API FastAPI\nPuerto: 8000]
        end
        
        subgraph Capa_Persistencia [Capa 2: Persistencia]
            DB[(PostgreSQL 15 +\nPostGIS + pgvector)]
            Adminer[Adminer\nPuerto: 8080]
        end
        
        subgraph Capa_Presentacion [Capa 4: Presentación]
            Frontend[Nginx + Vue 3\nPuerto: 8081]
        end
    end
    
    %% Simulación externa
    Sensores((Simulador / \nSensores ESP32)) -->|Publica Telemetría| MQTT
    
    %% Relaciones Internas
    MQTT -->|Consume| Worker
    Worker -->|Inserta| DB
    API -->|Lee/Escribe| DB
    Frontend -->|HTTP REST| API
    Adminer -.->|Visualiza| DB

    %% Estilos de Mermaid
    classDef blue fill:#2b3a42,stroke:#3b4a52,color:#fff;
    classDef green fill:#3b5249,stroke:#4b6259,color:#fff;
    classDef orange fill:#52423b,stroke:#62524b,color:#fff;
    
    class MQTT,Worker blue;
    class API green;
    class DB,Adminer orange;
```

---

## Desglose por Capas

### 2.1 Capa de Comunicación e Ingesta (Edge Gateway)

Este diagrama ilustra cómo los datos crudos del campo entran al Hub, se filtran y se preparan para su almacenamiento.

```mermaid
graph TD
    %% Entradas externas
    subgraph Campo [Entorno de Campo]
        A[Nodo Gateway LoRa] -.->|USB / Serial| B
        C[Cámaras IP / ESP32-CAM] -.->|Red Local RTSP| D
    end

    %% Capa de Ingesta
    subgraph Ingesta [Capa de Comunicación e Ingesta]
        B[Contenedor 1: Eclipse Mosquitto <br> Broker MQTT]
        E[Contenedor 2: Python Worker Ingesta]
        D[Contenedor 3: Python/Bash RTSP Worker]
        
        B == Suscripción a tópicos ==> E
        E -- Filtra, normaliza y estructura JSON --> E
    end

    %% Salidas a Persistencia
    subgraph Persistencia [Destinos]
        F[(PostgreSQL)]
        G[(Local File System)]
    end

    E -->|Batch SQL Insert| F
    D -->|Guarda imágenes/keyframes| G
    
    classDef container fill:#ebf8ff,stroke:#2b6cb0,stroke-width:2px;
    class B,E,D container;
```

---

### 2.2 Capa de Persistencia Multimodal (Data & GIS)

Este esquema detalla la estructura interna del contenedor de base de datos y cómo sus diferentes extensiones gestionan la naturaleza multimodal de la información.

```mermaid
graph LR
    %% Entradas
    A[Worker Ingesta] -->|Escritura de Telemetría| DB
    B[FastAPI Core] <-->|Consultas CRUD y GeoJSON| DB

    %% Contenedor PostgreSQL
    subgraph DB_Container [Contenedor 4: PostgreSQL Server]
        DB[(Motor Principal Relacional)]
        
        subgraph Extensiones [Módulos Internos]
            TS[Particionamiento Nativo <br> Series Temporales / Big Data]
            GIS[PostGIS <br> Geometría y Polígonos]
            VEC[pgvector <br> Embeddings para IA]
        end
        
        DB --- TS
        DB --- GIS
        DB --- VEC
    end

    classDef db fill:#fefcbf,stroke:#d69e2e,stroke-width:2px;
    class DB_Container,DB db;
    classDef ext fill:#fffff0,stroke:#d69e2e,stroke-dasharray: 5 5;
    class Extensiones,TS,GIS,VEC ext;
```

---

### 2.3 Capa de Lógica de Negocio y Core API

Aquí se visualiza el "cerebro" del sistema, donde conviven la seguridad, las reglas clásicas y las interfaces preparadas para la Inteligencia Artificial.

```mermaid
graph TD
    %% Actores Externos
    U[Frontend / PWA] -->|HTTP Request + JWT| API
    IA[Futura IA Local <br> Ollama / SLM] -.->|HTTP Request| API

    %% Contenedor FastAPI
    subgraph Core [Contenedor 5: API RESTful FastAPI]
        API((Router Central))
        
        API --> M1[Motor Determinista <br> Lógica Riego]
        API --> M2[Seguridad RBAC <br> Validación Roles]
        API --> M3[Geo-REST Endpoint <br> Mapas GeoJSON]
        API --> M4[AI Stubs <br> Function Calling]
    end

    %% Interacciones de Salida
    M1 -->|Publica orden de riego| MQTT[Broker Mosquitto]
    M4 -->|Publica acción hardware| MQTT
    M2 <-->|Verifica Token| DB[(PostgreSQL)]
    M3 <-->|Query Geoespacial| DB

    classDef core fill:#e6fffa,stroke:#319795,stroke-width:2px;
    class Core,API,M1,M2,M3,M4 core;
```

---

### 2.4 Capa de Presentación (Frontend)

Este diagrama muestra cómo el agricultor o agrónomo consume la información procesada sin necesidad de conectarse a internet, utilizando la red local de la finca.

```mermaid
graph LR
    %% Usuario
    User((Agricultor / Agrónomo)) -->|Navegador Red Local| NGINX

    %% Servidor Web
    subgraph Presentacion [Contenedor 6: NGINX Web Server]
        NGINX[Proxy Reverso y Servidor Estático]
        
        subgraph Archivos [Archivos Servidos]
            UI[Aplicación PWA <br> Vue 3]
            IMG[Imágenes y Videos <br> Cámaras IP]
        end
        
        NGINX --- UI
        NGINX --- IMG
    end

    %% Conexiones Internas
    UI -->|Peticiones Asíncronas REST| API[Capa Core API: FastAPI]
    IMG -.->|Lee de| FS[(Local File System)]

    classDef web fill:#edf2f7,stroke:#4a5568,stroke-width:2px;
    class Presentacion,NGINX web;
```

---

## PDFs Gráficos Originales
En tu estructura de directorios antigua existen estos archivos que documentan a nivel de hardware, arquitectura física, y conectividad IoT. Puedes utilizarlos como referencia anexa en el Obsidian:
- `Arquitectura_Hardware_Fisico_Hub.pdf`
- `Arquitectura_Hub_Edge_AI_Agricola.pdf`
- `Arquitectura_Software_Microservicios_Hub.pdf`
- Documentos PDF dentro de la carpeta `Capas`.
