# ADR-002 — Stack Tecnológico del Nodo de Borde (Edge)

**Fecha:** 28 Septiembre 2026
**Estado:** Aceptado (la elección del motor OCR queda sujeta a validación empírica, ver [Validación pendiente](#validación-pendiente))

---

## Contexto

El **Nodo de Borde** es el componente que vive en la portería: recibe el video de la cámara, detecta la placa patente, lee sus caracteres y decide si el vehículo puede entrar. Sus necesidades son:

1. **Latencia total < 100 ms por fotograma.** La literatura revisada en el Marco Teórico fija este presupuesto como el mínimo para sistemas de barrera: 100 ms equivale a 10 cuadros por segundo (FPS), el umbral para no perder un vehículo en movimiento.
2. **Offline-First.** La decisión de abrir o no el portón no puede depender de internet. Si se cae la red, el nodo debe seguir validando con su propia copia de las patentes autorizadas (*Edge Caching*).
3. **Recursos limitados.** El hardware de borde tiene poca memoria y consumo energético acotado, lo que descarta modelos grandes.
4. **Doble alcance.** Se distingue entre:
   - **Arquitectura de Referencia:** diseño industrial para la portería real (NVIDIA Jetson Orin Nano 8 GB + TensorRT), documentado en la tesis.
   - **Prototipo de Validación:** lo que efectivamente se construye en el Trabajo de Título. Software ejecutado en un PC con GPU dedicada, usando video de prueba, cámara web o RTSP desde un teléfono, dentro de Docker. **No se instala hardware en la portería.**

Este ADR justifica el stack para **ambos** alcances e indica cuándo una razón aplica solo a uno de ellos.

## Decisión

| Capa | Tecnología | Rol |
|---|---|---|
| Lenguaje | **Python 3.11** | Lenguaje único del pipeline |
| Captura y preprocesamiento | **OpenCV** | Lectura de video/RTSP y filtros de imagen |
| Detección de placa | **YOLOv12s** | Encontrar y recortar la patente en el fotograma |
| Lectura de caracteres (OCR) | **EasyOCR** | Convertir el recorte en texto (ej. `ABCD12`) |
| Persistencia local | **SQLite** (modo WAL) | Lista de patentes autorizadas y registros offline |
| API / dashboard local | **FastAPI** + **React lite** | Visualización en tiempo real para el operador |

La contenedorización con Docker se justifica por separado en [ADR-003](./_ADR-003-docker-infra.md).

## Justificación

### 1. Python 3.11

- Todas las piezas de IA del pipeline (YOLO, EasyOCR, OpenCV) tienen su interfaz principal en Python, por lo que no se necesitan puentes entre lenguajes.
- Mantiene un solo lenguaje de backend en todo el proyecto: el Nodo Central también usa Python con FastAPI ([ADR-001](./ADR-001-stack-central.md)).
- La versión se fija en la imagen Docker (`python:3.11-slim`) para que el entorno sea idéntico en cualquier PC.

### 2. OpenCV

- Estándar de facto para leer flujos de video. Una misma interfaz sirve para un archivo `.mp4`, una cámara web o un stream RTSP de cámara IP. Esto permite desarrollar con video grabado y demostrar con cámara real sin cambiar el código.
- Incluye los filtros de preprocesamiento (mejora de contraste, redimensionado, recorte) que se aplican antes de la detección.

### 3. YOLOv12s (detección)

**¿Qué es?** YOLO (*You Only Look Once*) es una familia de redes neuronales que detecta objetos en una imagen en una sola pasada, lo que la hace muy rápida. El número indica la versión y la letra el tamaño del modelo.

**¿Por qué la familia YOLO y no otros detectores?** El Marco Teórico compara alternativas: Faster R-CNN es preciso pero lento por trabajar en dos etapas, y SSD y EfficientDet-D0 pierden precisión con placas pequeñas o en ángulo. YOLO es el estándar de la industria para detección en tiempo real.

**¿Por qué la versión 12?**
- YOLOv8 introdujo la detección *anchor-free* (sin "cajas guía" predefinidas: el modelo predice directamente el centro del objeto), lo que mejora el manejo de placas a distintas distancias. YOLOv12 **hereda** esta ventaja.
- Lo propio de YOLOv12 es su **mecanismo de atención**: la red aprende a "fijarse" en la zona relevante de la imagen e ignorar el fondo (árboles, rejas, otros autos). Según Buleu et al. (2025), YOLOv12 mejora un 2,3 % la precisión respecto de YOLOv11 en detección de patentes.
- Buleu et al. validaron YOLOv12 en laptops y dejaron su despliegue en el borde como trabajo futuro. Este proyecto aborda justamente ese vacío.

**¿Por qué la variante *Small* (s) y no *Nano* (n) ni otras mayores?**
- Las variantes *Large*/*XL* no caben en el presupuesto de memoria y latencia del borde, y no aportan beneficios relevantes en una tarea de una sola clase (la patente).
- En la Arquitectura de Referencia, la GPU de la Jetson y la compilación con TensorRT permiten ejecutar *Small* dentro de los 100 ms (14–30 FPS según la literatura), con más precisión que *Nano*.
- En el Prototipo de Validación, la GPU del PC de desarrollo ejecuta la variante *Small* con holgura.

> **Alternativa considerada — YOLOv26 (enero 2026):** optimizada para inferencia en CPU. No se elige porque ambos alcances cuentan con GPU. Queda como opción si el sistema se llevara a hardware sin aceleración gráfica.

### 4. EasyOCR (lectura de caracteres)

**¿Qué es?** Una biblioteca de reconocimiento óptico de caracteres (OCR) basada en PyTorch. Combina un detector de texto (CRAFT) con una red que lee la secuencia completa de caracteres (CRNN + CTC), sin separar letra por letra.

**Frente a Tesseract (descartado):** Tesseract está diseñado para documentos impresos y escaneados. Según el Marco Teórico, procesa unas 25 lecturas por minuto en CPU y se degrada mucho ante placas inclinadas o en perspectiva, algo habitual en una cámara de portería.

**Frente a PaddleOCR (alternativa fuerte, no descartada):** el Marco Teórico reconoce a PaddleOCR como el motor más rápido y preciso documentado (hasta 120 lecturas/min en GPU). Aun así, **para el Prototipo de Validación** se elige EasyOCR por las siguientes razones:

1. **Las debilidades documentadas de EasyOCR corresponden a ejecución sin GPU.** La literatura reporta ~8 lecturas/min y confianza oscilante *en entornos carentes de aceleración gráfica*. El prototipo corre con GPU dedicada, por lo que esa limitación no aplica en su contexto. Esto se verificará con mediciones (ver [Validación pendiente](#validación-pendiente)).
2. **Uso comprobado junto a YOLO en ALPR.** Mobeen (2026) utiliza EasyOCR como etapa de lectura tras un detector YOLO, restringiendo su salida mediante verificación de la sintaxis posicional de la patente. Este proyecto aplica el mismo principio: el texto leído se filtra con las reglas de los formatos chilenos antes de consultar la base de datos.
3. **Mismo ecosistema que el detector.** EasyOCR corre sobre PyTorch, igual que la implementación de YOLO usada. Ambos modelos comparten framework, dependencias CUDA y memoria de GPU dentro de un solo contenedor. PaddleOCR requiere su propio framework (PaddlePaddle) con su propio stack de CUDA, lo que duplica dependencias y complica la imagen Docker.
4. **Integración simple.** Se instala con `pip` y se usa con pocas líneas de código, lo que reduce el riesgo en un prototipo con plazos de titulación.

**En la Arquitectura de Referencia (Jetson + TensorRT)**, PaddleOCR sigue siendo la opción recomendada por la literatura. El pipeline aísla el OCR en su propio módulo (`pipeline/ocr.py`), por lo que cambiar de motor no afecta al resto del sistema.

### 5. SQLite (persistencia local)

- **Sin servidor:** la base de datos es un archivo dentro del mismo contenedor. No hay otro proceso que pueda caerse ni puertos que abrir.
- **Implementa el patrón *Edge Caching*** descrito en el Marco Teórico: el nodo guarda localmente la lista de patentes autorizadas y la consulta en milisegundos, sin depender de la red. La sincronización con el Nodo Central ocurre en segundo plano.
- **Modo WAL (*Write-Ahead Logging*):** permite que el hilo de validación lea mientras el hilo de I/O escribe registros de acceso, sin bloquearse entre sí.
- Una consulta por clave indexada (la patente) cabe con holgura en el objetivo de < 5 ms para la etapa de validación.

### 6. FastAPI + React lite (API y dashboard local)

- FastAPI expone el estado del pipeline (última lectura, decisión, latencias) de forma asíncrona, sin interferir con el hilo de inferencia. Es el mismo framework del Nodo Central.
- El dashboard React es una vista liviana para el operador: muestra la detección en tiempo real y la señal simulada de "apertura de portón". No contiene lógica de decisión.

## Presupuesto de latencia

Distribución objetivo de los 100 ms (ver [system-architecture.md](./system-architecture.md)):

| Etapa | Tecnología | Objetivo |
|---|---|---|
| Captura + preprocesamiento | OpenCV | < 10 ms |
| Detección | YOLOv12s | < 30 ms |
| Lectura OCR | EasyOCR | < 40 ms |
| Validación | SQLite | < 5 ms |
| **Total** | | **< 100 ms** |

## Consecuencias

### Positivas
- Un solo lenguaje (Python) y un solo framework de IA (PyTorch) en el pipeline, lo que simplifica el desarrollo, la imagen Docker y la depuración.
- Operación completamente offline: el portón sigue funcionando aunque se caiga internet.
- Uso de un detector del estado del arte (YOLOv12s) en un escenario de borde que la literatura dejó como trabajo futuro.
- El OCR está aislado en su propio módulo, así que puede reemplazarse sin rediseñar el sistema.

### Negativas / Riesgos
- **EasyOCR no es el motor más rápido documentado.** Si no cumple los 40 ms o la tasa de lectura mínima, habrá que migrar a PaddleOCR (mitigado por el módulo aislado y la validación pendiente).
- **EasyOCR puede entregar confianzas oscilantes en video.** Se mitiga con el filtro de formato chileno y, si es necesario, con la consolidación de lecturas de varios fotogramas consecutivos.
- **YOLOv12 es reciente (febrero 2025):** hay menos ejemplos y soporte comunitario que para YOLOv8. Si surgen problemas de compatibilidad, YOLOv8s es el respaldo, también documentado en el Marco Teórico.
- **Dependencia de GPU:** el rendimiento objetivo asume aceleración gráfica. En un equipo sin GPU el pipeline funciona, pero no cumpliría los 100 ms.
- SQLite admite un solo escritor a la vez, lo cual es suficiente para un nodo por portería, pero no escalaría a muchos procesos escribiendo en paralelo.

## Validación pendiente

La elección de EasyOCR se confirmará con datos durante la implementación del módulo OCR (`alpr-edge-node` #4). Se compararán **EasyOCR y PaddleOCR** sobre el mismo conjunto de recortes de patentes chilenas, midiendo:

| Métrica | Criterio de aceptación |
|---|---|
| Latencia OCR por recorte (GPU) | < 40 ms (objetivo), < 50 ms (mínimo) |
| Tasa de lectura correcta de patente | ≥ 85 % (mínimo), ≥ 90 % (objetivo) |

Si EasyOCR no cumple los mínimos y PaddleOCR sí, este ADR se actualizará y se reemplazará el motor OCR.

## Referencias

Las fuentes citadas se encuentran en el Marco Teórico de la tesis (`Tesis/documento/capitulos/marco_teorico.tex`):
- Buleu et al. (2025): YOLOv12 aplicado a reconocimiento de patentes.
- Mobeen (2026): pipeline ALPR en tiempo real con YOLOv8 + EasyOCR y verificación sintáctica.
- Sonnara et al. (2025): presupuesto de latencia de 100 ms e inferencia en Jetson con TensorRT.
- Xu et al. (2026): comparación de motores OCR (Tesseract, EasyOCR, PaddleOCR) y *Edge Caching*.
