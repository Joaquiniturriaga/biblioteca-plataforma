# Topología · biblioteca

| Exchange | Tipo | Cola | Binding | Caso de uso |
|---|---|---|---|---|
| `biblioteca.eventos` | topic | `notificaciones` | `prestamo.*` | Avisar al lector de lo que pasó con su préstamo |
| `biblioteca.eventos` | topic | `auditoria` | `#` | Guardar todo hecho en `eventos_auditoria` |
| `biblioteca.comandos` | direct | `correos` | `correo.enviar` | «Enviar» el correo y guardar la `Notificacion` |
| `biblioteca.dlx` | direct | `notificaciones.dlq` · `auditoria.dlq` · `correos.dlq` | el nombre de su cola de trabajo | Apartar lo que no se pudo procesar |

| Routing key | Quién la publica | Payload |
|---|---|---|
| `prestamo.creado` | `prestamos.mjs` | `{ prestamoId, libroId, usuarioSub, hasta }` |
| `prestamo.devuelto` | `prestamos.mjs` | `{ prestamoId, libroId, usuarioSub }` |
| `correo.enviar` | `NotificacionesConsumidor` | `{ para, asunto, cuerpo, origen, eventoId }` |
