# Plan de Proyecto de Software — Gestor de Presupuesto del Hogar

## 1. La situación actual

Hoy, el control del presupuesto de un hogar con varios miembros se hace de forma dispersa: cada persona conoce sus propios gastos, pero nadie tiene una vista conjunta de cuánto entra y cuánto sale del hogar en total. Los pagos recurrentes (arriendo, servicios públicos, suscripciones) se recuerdan de memoria o mediante alarmas sueltas en el celular de quien paga habitualmente. No existe un registro centralizado de quién gastó qué, en qué categoría, ni de cuándo vence cada obligación. Cuando alguien olvida una fecha de pago, se entera por el corte del servicio o por un recargo, no antes.

## 2. El problema

- Se pierden fechas de pago porque dependen de la memoria de una sola persona, generando recargos por mora o suspensión de servicios.
- No hay visibilidad de cuánto ha gastado cada miembro del hogar ni en qué categorías, así que las decisiones de gasto se toman sin saber si ya se superó el presupuesto disponible.
- El dinero disponible del hogar (ingresos menos compromisos fijos) no se conoce con certeza en un momento dado, lo que lleva a gastos que comprometen el pago de obligaciones próximas.

A quién le afecta: a todos los miembros del hogar que aportan o gastan del presupuesto compartido, y en particular a quien administra los pagos recurrentes.

## 3. Qué hará el sistema

1. Registrar un movimiento (ingreso o gasto) asociado a un miembro del hogar y una categoría, opcionalmente vinculado a un pago pendiente.
2. Registrar una obligación de pago con su fecha límite, su monto y si admite abonos parciales.
3. Calcular el saldo disponible del presupuesto del hogar en un momento dado.
4. Alertar sobre pagos próximos a vencer, con un nivel de urgencia según los días restantes.
5. Alertar cuando una categoría de gasto supera un porcentaje de su límite mensual.
6. Consultar el resumen de gastos por miembro y por categoría en un rango de fechas.
7. Marcar un gasto como "gasto hormiga" y alertar cuando su acumulado mensual supera un umbral definido.

## 4. Entradas, procesos y salidas

| Funcionalidad | Entradas | Proceso | Salidas |
|---|---|---|---|
| Registrar movimiento | miembro, tipo (ingreso/gasto), categoría, monto, fecha, pago_pendiente_id (opcional) | valida monto > 0 y categoría existente; si viene vinculado a un pago no abonable, el monto se autocompleta con el monto pendiente de ese pago (no editable); si el pago es abonable, el monto es libre; actualiza saldo del hogar; si el pago vinculado queda saldado, lo marca "pagado" | confirmación + saldo actualizado + estado del pago vinculado (si aplica) |
| Registrar obligación de pago | nombre del pago, monto, fecha límite, ¿recurrente?, ¿abonable? | valida fecha límite futura; la agrega a la lista de pagos pendientes | confirmación + pago agregado a la lista |
| Calcular saldo disponible | (usa movimientos e ingresos registrados) | suma ingresos del mes, resta gastos y obligaciones pendientes del mes | saldo disponible actual |
| Alertar vencimientos | fecha actual, lista de pagos pendientes | calcula días restantes por pago y asigna nivel de urgencia | lista de pagos con su color/nivel |
| Alertar límite de categoría | gastos acumulados del mes por categoría, límite de la categoría | compara gasto acumulado contra el % del límite | aviso si se cruza el umbral |
| Resumen por miembro/categoría | rango de fechas, filtro opcional de miembro o categoría | agrupa y suma movimientos según filtros | tabla o totales agregados |
| Observar gastos hormiga | movimientos marcados como "hormiga" del mes en curso | suma el monto de todos los movimientos marcados como hormiga en el mes; compara contra el umbral mensual | total acumulado del mes + aviso si supera el umbral |

## 5. Datos que se manejan

**Miembro del hogar**
- id (entero)
- nombre (texto)

**Categoría**
- id (entero)
- nombre (texto)
- límite mensual (decimal)

**Movimiento**
- id (entero)
- miembro_id (referencia a Miembro)
- categoría_id (referencia a Categoría)
- tipo (texto: "ingreso" o "gasto")
- monto (decimal)
- fecha (fecha)
- es_hormiga (booleano) — solo aplica a movimientos de tipo "gasto"; indica si el usuario lo marcó como gasto hormiga al registrarlo
- pago_pendiente_id (referencia a Pago pendiente, opcional) — solo aplica a movimientos de tipo "gasto"; vincula el movimiento con la obligación que está abonando o pagando

**Pago pendiente**
- id (entero)
- nombre (texto)
- monto (decimal)
- fecha_límite (fecha)
- recurrente (booleano)
- abonable (booleano) — si es true, el pago admite varios movimientos parciales; si es false, solo se salda con un único movimiento por el monto completo
- estado (texto: "pendiente" o "pagado")

## 6. Reglas del sistema

- **Niveles de urgencia de un pago pendiente**, según días restantes hasta la fecha límite:
  - Verde: 8 días o más.
  - Amarillo: entre 3 y 7 días (inclusive).
  - Rojo: 2 días o menos, incluyendo el día del vencimiento.
  - Vencido: fecha límite ya pasada y el pago sigue en estado "pendiente".
- **Caso límite**: un pago con exactamente 7 días restantes se clasifica como amarillo (no verde); un pago con exactamente 2 días restantes se clasifica como rojo.
- **Alerta de categoría**: cuando el gasto acumulado del mes en una categoría alcanza el 90 % de su límite mensual, se marca en amarillo; al llegar o superar el 100 %, se marca en rojo.
  - Caso límite: con el gasto acumulado en exactamente 90 % del límite, ya se considera amarillo (no verde).
- **Saldo negativo**: si al registrar un gasto el saldo disponible del hogar quedaría en negativo, el sistema **rechaza el registro** y no permite guardar el movimiento. El usuario debe ajustar el monto o registrar antes un ingreso que cubra la diferencia.
- **Vinculación de movimiento a pago pendiente**: al registrar un gasto, el usuario puede (opcionalmente) elegir a qué pago pendiente corresponde:
  - Si el pago **no es abonable**: el campo monto del movimiento se autocompleta con el monto restante del pago y no se puede modificar. Al guardar el movimiento, el pago pasa a estado "pagado" de inmediato.
  - Si el pago **es abonable**: el usuario ingresa libremente el monto que va a abonar (no tiene que ser el total). El sistema suma todos los movimientos históricos vinculados a ese pago; en cuanto la suma de abonos llega o supera el monto total del pago, este pasa a estado "pagado" automáticamente.
  - Un movimiento sin pago_pendiente_id no afecta el estado de ningún pago — sigue siendo un gasto normal.
  - Caso límite: si la suma de abonos de un pago abonable llega exactamente al monto total, se marca "pagado" en ese mismo momento (no se espera a que lo supere).
- **Umbral de gastos hormiga**: se suman todos los movimientos del mes marcados con `es_hormiga = true`, sin importar su categoría. Si ese acumulado llega o supera $100.000 en el mes, el sistema muestra un aviso.
  - Caso límite: con el acumulado en exactamente $100.000, ya se dispara el aviso (no se espera a superarlo).
  - Un gasto puede estar marcado como hormiga y seguir contando también para el límite normal de su categoría — las dos reglas son independientes y no se excluyen entre sí.

## 7. Qué queda fuera

- No se integra con bancos ni se importan movimientos automáticamente desde extractos bancarios.
- No se generan reportes en PDF ni se exportan a Excel en esta primera versión.
- No se manejan monedas distintas ni conversión de divisas.
- No hay roles de permisos distintos entre miembros (todos pueden ver y registrar todo).

## 8. Módulo mínimo y funcionalidades opcionales

**Módulo mínimo** (resuelve el problema central y se puede mostrar funcionando):
- Registrar movimiento (func. 1)
- Registrar obligación de pago (func. 2)
- Alertar vencimientos con niveles de urgencia (func. 4)

**Opcionales** (se construyen si el tiempo alcanza):
- Calcular saldo disponible del hogar (func. 3)
- Alertar límite de categoría (func. 5)
- Resumen por miembro/categoría (func. 6)
- Observar gastos hormiga (func. 7)
