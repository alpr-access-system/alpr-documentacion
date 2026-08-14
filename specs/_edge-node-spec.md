# Especificación — Nodo de Borde (Edge Node)

> ⏳ **PENDIENTE DE COMPLETAR** — Se trabajará en el Sprint 1 / Sprint 2.

---

## Descripción

Servicio de inferencia local que captura video de una cámara, detecta placas patentes,
extrae texto con OCR y valida el acceso contra la base de datos local (SQLite).
Opera de forma independiente a la conectividad de red (Offline-First).

## Componentes

- [ ] Pipeline de captura de video (OpenCV + RTSP/webcam)
- [ ] Módulo de detección (YOLOv12s)
- [ ] Módulo de OCR (EasyOCR)
- [ ] Módulo de validación (SQLite)
- [ ] Módulo de sincronización (HTTP hacia API Central)
- [ ] Dashboard local (React lite)

## Requisitos de Rendimiento

- Latencia total pipeline: < 100 ms
- Funcionamiento sin red (Offline-First): ✅ obligatorio
- Sincronización automática al recuperar red: ✅ obligatorio

## Formatos de Patente Soportados

*(5 formatos chilenos — 1985 a hoy. Por documentar en detalle.)*
