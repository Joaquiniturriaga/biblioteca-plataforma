# biblioteca-plataforma

El `compose.yml` que levanta el sistema entero —RabbitMQ, Postgres, el worker, libros, prestamos, el gateway, el BFF y el Angular—, la topología en `docs/topologia.md` y el modelo de datos en `docs/modelo-de-datos.md`.

## Levantar

Los repositorios van uno al lado del otro, en la misma carpeta que este. Cada uno necesita su `.env`, y este, el suyo: cópialo de `.env.example` y rellénalo.

```bash
docker compose up -d --build --wait
docker compose ps
```

## Mensaje envenenado

Desde `biblioteca-plataforma`, con el sistema arriba:

```bash
docker compose run --rm --no-deps eventos node herramientas/emitir.mjs 1 --roto
docker compose exec rabbit1 rabbitmqctl list_queues name messages
```

Qué se tiene que ver: en los logs del worker (`docker compose logs eventos`), un `ERROR` de `AuditoriaConsumidor` y otro de `NotificacionesConsumidor` que lo mandan a la DLQ, porque `prestamo.creado` calza con `#` y con `prestamo.*`. Después, `CartasMuertasConsumidor` registra las dos copias:

```bash
docker compose exec postgres psql -U biblioteca -d biblioteca -c "select cola_origen, motivo, intentos, left(payload,30) as payload from mensajes_muertos order by recibido_en desc limit 2;"
```

Dos filas, una con `cola_origen` `auditoria` y otra con `notificaciones`, las dos con motivo `rejected` y el payload `{ esto no es JSON valido`. Las colas de trabajo quedan en 0: el mensaje roto no se reencola.
