# Integración automática de pedidos online en el TPV de una pizzería

Proyecto Intermodular de DAM · CFGS DAM (2º DAM Virtual) · Curso 2026/2027

> Este README es el **registro vivo de decisiones, fases y estructura** del proyecto. Sirve de base para la memoria final (entrega: enero). Cada decisión importante se anota aquí con su motivo.

---

## 1. Resumen

Plataforma de pedidos online para una pizzería pequeña que **introduce automáticamente cada pedido en el TPV del local**, aunque este sea un programa cerrado sin API. El sistema se hace pasar por la impresora de tickets del local, reenvía el ticket a la impresora real (la cocina lo sigue recibiendo), extrae los datos del pedido y los teclea en el TPV simulando el teclado. Después verifica que el pedido se registró y reintenta si falla.

Alcance: todo el flujo desde que el cliente pide hasta que el pedido queda en el TPV de demostración. **Fuera de alcance:** pagos reales (se simulan), Glovo/Just Eat, TPV comercial concreto.

## 2. Componentes

| # | Componente | Tecnología | Carpeta |
|---|---|---|---|
| 1 | Web de pedidos (clientes) | React + Vite | `web/` |
| 2 | App de pedidos (Android) | React Native + Expo | `mobile/` |
| 3 | Backend (API REST) + BBDD | Spring Boot + PostgreSQL + JWT | `backend/` |
| 4 | Módulo de integración (núcleo) | Java | `integrador/` |
| 5 | Panel de seguimiento y administración | Rutas protegidas de la web React | `web/` |
| 6 | TPV de demostración | JavaFX | `tpv-demo/` |

Actores: **cliente**, **encargado** (gestiona pedidos) y **administrador** (carta, usuarios, configuración).

## 3. Estructura del repositorio

```
pizzeria-tpv/                  monorepo, un solo repositorio git
├── settings.gradle            Gradle multiproyecto (backend, integrador, tpv-demo)
├── backend/                   Spring Boot + PostgreSQL + JWT
├── integrador/                impresora falsa + parser + teclado (Robot) + verificación
├── tpv-demo/                  TPV "antiguo" en JavaFX, manejado solo por teclado
├── web/                       React + Vite (clientes, panel, administración)
├── mobile/                    React Native + Expo
├── docker-compose.yml         PostgreSQL para desarrollo
└── docs/                      material de la memoria (diagramas, modelo de datos, pruebas)
```

## 4. Flujo del sistema

1. El cliente hace un pedido desde la web o la app.
2. El backend lo guarda en PostgreSQL y "imprime" el ticket por TCP a la dirección configurada como impresora (en realidad, el integrador).
3. El integrador reenvía el ticket sin modificar a la impresora real (en pruebas, un simulador que guarda los tickets en archivos).
4. El integrador interpreta el ticket y teclea el pedido en la ventana del TPV.
5. Verifica en el archivo de datos del TPV que el pedido está y coincide. Si no, reintenta.
6. Informa al backend del resultado (`PENDIENTE` → `INTRODUCIDO` o `ERROR`); el panel lo muestra en tiempo real.
7. El encargado puede corregir y reenviar los pedidos con error.

**Evitar duplicados y pérdidas:** el integrador guarda qué pedidos ha procesado; antes de reintentar comprueba si el pedido ya está en el TPV; el reenvío desde el panel solo se permite en pedidos no introducidos.

## 5. Decisiones tomadas

| Decisión | Elegido | Motivo |
|---|---|---|
| Backend | Spring Boot + Gradle | Preferencia del alumno; coherente con el resto del stack Java |
| Base de datos | PostgreSQL (vía Docker Compose) | Conocimiento previo del alumno; estándar en proyectos DAM |
| Web | React + Vite | Requisito del anteproyecto |
| App móvil | React Native + Expo | Requisito del anteproyecto; se probará en emulador y móvil real |
| Integrador | Java | Un único lenguaje con backend y TPV; menos fricción |
| Simulación de teclado | `java.awt.Robot` con portapapeles | `Robot` falla con `ñ`, tildes y `@` según el teclado; se pega cada campo con Ctrl+V y solo se usan Tab/Enter/teclas F directamente |
| TPV demo | JavaFX | Preferencia del alumno; interfaz de escritorio moderna con Gradle |
| Panel | Ruta protegida de la web por rol | Comparte login, API y componentes; menos código y despliegues |
| Repositorio | Monorepo | Un solo sitio para código y documentación |
| Verificación | Lectura del archivo de datos del TPV | Un TPV cerrado no tiene API, pero guarda sus datos en disco |

### Supuestos (del anteproyecto)
Impresora de red redirigible; TPV en Windows manejado por teclado; el local autoriza la instalación; máximo ~100 pedidos/día.

## 6. Fases

| Fase | Contenido | Estado |
|---|---|---|
| 1 | **Entorno:** Gradle wrapper, Docker Compose con PostgreSQL, repositorio git | Hecho (falta arrancar Docker Desktop para probar Postgres) |
| 2 | **Núcleo:** TPV demo (JavaFX) + integrador (Java), probados con un ticket de ejemplo sin backend | Pendiente |
| 3 | **Backend:** carta, clientes, pedidos, impresión por TCP, estados en tiempo real, endpoint para el integrador. Prueba del flujo completo | Pendiente |
| 4 | **Web:** carta, carrito, registro, confirmación; después panel y administración | Pendiente |
| 5 | **App móvil:** Expo (requiere Android Studio y emulador) | Pendiente |
| 6 | **Memoria:** se redacta en `docs/` desde la fase 2 (diagramas, modelo de datos, casos de uso, pruebas, mejoras futuras) | Pendiente |

## 7. Registro de cambios y decisiones

- **2026-11-11** — Definición de stack, estructura y fases tras revisar el anteproyecto. Se descartó un primer enfoque en Node para alinear todo con Java/Spring Boot.

## 8. Mejoras futuras (propuestas)
Pagos reales; integración con Glovo/Just Eat; conexión con un TPV comercial concreto.
