# Motor de renovaciones reutilizable

## Objetivo

Una sola ventana de renovación atiende dos entidades:

- `client`: renueva acceso vendido, reactiva el estado, registra ingreso y prepara el mensaje de agradecimiento.
- `account`: renueva solo la fecha de vencimiento de la cuenta madre. No toca clientes, perfiles, ingresos ni WhatsApp.

## Decisión de diseño

La ventana conserva los mismos atajos (1 mes, 3 meses, 1 año, duración personalizada y fecha exacta), pero usa un destino tipado:

```js
{ kind: 'client' | 'account', id: '...' }
```

Esto evita tener dos modales casi iguales y centraliza las reglas de fecha en `renewalBaseDate` y `renewalDateFor`.

## Reglas de seguridad contra cruces

1. La cuenta madre solo guarda `dueDate` y `updatedAt` en `accounts`.
2. Una renovación de cuenta madre no cambia el estado de sus clientes ni sus perfiles libres.
3. Una renovación de cuenta madre no crea ingresos de cliente ni abre WhatsApp.
4. La renovación de cliente conserva su comportamiento previo: estado activo, combo, finanzas y mensaje de agradecimiento.
5. La fecha se calcula desde el vencimiento existente; si no existe, parte de hoy. Los meses usan ajuste al último día válido del mes.
6. Cerrar el modal borra el destino actual para que una acción posterior no reutilice una cuenta o cliente anterior.

## Cómo reutilizar en otro proyecto

1. Mantén el destino como `{ kind, id }`, no como una variable global por tipo de entidad.
2. Centraliza la aritmética de fechas en funciones puras y probables.
3. Define una rama explícita por `kind` solo para los efectos secundarios.
4. Mantén el formulario común y oculta los campos que no aplican (por ejemplo, el precio de renovación de cliente para una cuenta madre).
5. Persiste el cambio antes de actualizar la interfaz local; si falla, no muestres éxito.

## Pruebas mínimas antes de publicar

- Renovar una cuenta madre 1 mes y verificar que solo cambie su vencimiento.
- Renovar una cuenta madre con fecha exacta y verificar que no aparezca un movimiento financiero.
- Renovar un cliente y verificar que mantenga el flujo de ingresos/WhatsApp existente.
- Abrir una renovación de cuenta, cerrarla y renovar un cliente: confirmar que el destino no se mezcla.
- Probar meses de 28, 29, 30 y 31 días para confirmar el ajuste de fecha.
