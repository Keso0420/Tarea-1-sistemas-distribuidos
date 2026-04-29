¡Claro! Un buen **README** en GitHub es la cara de tu proyecto. Aquí tienes una estructura profesional que explica la arquitectura, el flujo de datos y cómo ejecutar los experimentos que realizaste.

---

# 🛰️ Sistema Distribuido de Análisis Urbano: Santiago de Chile

Este proyecto implementa una arquitectura de microservicios diseñada para procesar consultas geoespaciales masivas sobre un dataset de edificios en la Región Metropolitana. El sistema utiliza una estrategia de **Caché Distribuida (Cache-Aside)** para optimizar la latencia y reducir la carga en el generador de respuestas.

## 🏗️ Arquitectura del Sistema

El sistema se compone de 4 microservicios orquestados mediante **Docker Compose**:

1.  **Generador de Tráfico (`trafico.py`)**: Simula clientes reales enviando consultas (Q1-Q5) bajo distribuciones de probabilidad **Zipf** (tráfico sesgado) o **Uniforme** (tráfico aleatorio).
2.  **Sistema de Caché (`cache.py`)**: Actúa como un Proxy inteligente. Verifica la existencia de datos en una base de datos **Redis** antes de delegar el trabajo al backend.
3.  **Generador de Respuestas (`respuestas.py`)**: El motor de cómputo. Carga un dataset de Santiago en memoria y realiza cálculos de área, densidad y distribución de confianza.
4.  **Almacenamiento de Métricas (`metricas.py`)**: Servicio de telemetría que registra cada evento para el análisis posterior de rendimiento (Throughput, Latencia, Hit Rate).



---

## 🚀 Cómo Ejecutar

### Requisitos previos
* Docker y Docker Compose.
* Python 3.12+ (para el script de análisis local).
* Dataset `santiagoCSV.gz` ubicado en la carpeta `data/`.

### Instalación y Puesta en Marcha
1.  Clona el repositorio.
2.  Levanta la infraestructura completa:
    ```bash
    sudo docker compose up --build
    ```
3.  El sistema comenzará a generar tráfico automáticamente y verás los logs en tiempo real.

---

## 📊 Tipos de Consultas (Q1 - Q5)

El sistema soporta cinco operaciones analíticas:
* **Q1**: Conteo total de edificios en una zona con un umbral de confianza.
* **Q2**: Cálculo de área promedio y área total construida.
* **Q3**: Densidad de edificios por km².
* **Q4**: Comparación de densidad entre dos zonas geográficas.
* **Q5**: Distribución de confianza (Histograma) basada en intervalos dinámicos.

---

## 🧪 Análisis de Rendimiento

Para evaluar la eficiencia de la caché, el proyecto incluye un script de análisis estadístico (`analisis.py`) que procesa los registros de telemetría.

### Métricas Calculadas:
* **Hit Rate**: Efectividad de la caché para evitar procesamiento innecesario.
* **Throughput**: Capacidad de respuesta del sistema (req/seg).
* **Latencia p50 y p95**: Percentiles críticos para entender la experiencia del usuario.
* **Eviction Rate**: Tasa de reemplazo de llaves en Redis según la política **LRU**.
* **Cache Efficiency**: Relación costo-beneficio entre latencia de hit vs latencia de miss.

### Ejecución del Análisis
Una vez finalizado el tráfico, ejecuta en tu terminal local:
```bash
python3 analisis.py
```

---

## ⚙️ Configuración de Experimentos

Puedes modificar el comportamiento del sistema editando `docker-compose.yml`:

```yaml
# Cambia el tamaño de la memoria para probar políticas de reemplazo
command: redis-server --maxmemory 50mb --maxmemory-policy allkeys-lru
```

Y en `trafico.py`, puedes alternar entre:
* `distribucion="zipf"`: Ideal para observar altos Hit Rates.
* `distribucion="uniform"`: Ideal para poner a prueba la saturación de la memoria.

---

## 🛠️ Tecnologías Utilizadas
* **Lenguaje**: Python 3.12
* **Framework**: Flask (Microservicios REST)
* **Caché**: Redis (Base de datos en memoria)
* **Procesamiento de Datos**: Pandas & NumPy
* **Contenerización**: Docker & Docker Compose
