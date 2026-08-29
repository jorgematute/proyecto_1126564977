# Plan de Proyecto de Software — Gestor de Presupuesto del Hogar

> **Nota:** este es un borrador de trabajo. Los valores numéricos (montos, días, porcentajes) son ejemplos razonables — debes ajustarlos a un caso real o a lo que tenga sentido para tu hogar, y ser capaz de justificar por qué elegiste esos números y no otros.

## 1. La situación actual

Hoy, el control del presupuesto de un hogar con varios miembros se hace de forma dispersa: cada persona conoce sus propios gastos, pero nadie tiene una vista conjunta de cuánto entra y cuánto sale del hogar en total. Los pagos recurrentes (arriendo, servicios públicos, suscripciones) se recuerdan de memoria o mediante alarmas sueltas en el celular de quien paga habitualmente. No existe un registro centralizado de quién gastó qué, en qué categoría, ni de cuándo vence cada obligación. Cuando alguien olvida una fecha de pago, se entera por el corte del servicio o por un recargo, no antes.

## 2. El problema

- Se pierden fechas de pago porque dependen de la memoria de una sola persona, generando recargos por mora o suspensión de servicios.
- No hay visibilidad de cuánto ha gastado cada miembro del hogar ni en qué categorías, así que las decisiones de gasto se toman sin saber si ya se superó el presupuesto disponible.
- El dinero disponible del hogar (ingresos menos compromisos fijos) no se conoce con certeza en un momento dado, lo que lleva a gastos que comprometen el pago de obligaciones próximas.

A quién le afecta: a todos los miembros del hogar que aportan o gastan del presupuesto compartido, y en particular a quien administra los pagos recurrentes.

## 3. Qué hará el sistema

1. Registrar un movimiento (ingreso o gasto) asociado a un miembro del hogar y una categoría.
2. Registrar una obligación de pago con su fecha límite y su monto.
3. Calcular el saldo disponible del presupuesto del hogar en un momento dado.
4. Alertar sobre pagos próximos a vencer, con un nivel de urgencia según los días restantes.
5. Alertar cuando una categoría de gasto supera un porcentaje de su límite mensual.
6. Consultar el resumen de gastos por miembro y por categoría en un rango de fechas.

## 4. Entradas, procesos y salidas

| Funcionalidad | Entradas | Proceso | Salidas |
|---|---|---|---|
| Registrar movimiento | miembro, tipo (ingreso/gasto), categoría, monto, fecha | valida monto > 0 y categoría existente; actualiza saldo del hogar | confirmación + saldo actualizado |
| Registrar obligación de pago | nombre del pago, monto, fecha límite, ¿recurrente? | valida fecha límite futura; la agrega a la lista de pagos pendientes | confirmación + pago agregado a la lista |
| Calcular saldo disponible | (usa movimientos e ingresos registrados) | suma ingresos del mes, resta gastos y obligaciones pendientes del mes | saldo disponible actual |
| Alertar vencimientos | fecha actual, lista de pagos pendientes | calcula días restantes por pago y asigna nivel de urgencia | lista de pagos con su color/nivel |
| Alertar límite de categoría | gastos acumulados del mes por categoría, límite de la categoría | compara gasto acumulado contra el % del límite | aviso si se cruza el umbral |
| Resumen por miembro/categoría | rango de fechas, filtro opcional de miembro o categoría | agrupa y suma movimientos según filtros | tabla o totales agregados |

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

**Pago pendiente**
- id (entero)
- nombre (texto)
- monto (decimal)
- fecha_límite (fecha)
- recurrente (booleano)
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
- **Saldo negativo**: si al registrar un gasto el saldo disponible del hogar quedaría en negativo, el sistema no bloquea el registro, pero marca el movimiento con una advertencia de "sobregiro". (Alternativa a decidir: bloquear el registro por completo — debes elegir una y justificarla.)
- Un pago pendiente pasa a estado "pagado" solo cuando se registra explícitamente como tal; no cambia de estado automáticamente por tener un movimiento de gasto asociado.

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

---

### Antes de entregar (checklist del curso)
- [ ] Renombrar/mover este archivo a `proyecto/PLAN.md` en tu repositorio.
- [ ] Ajustar los montos, porcentajes y días de las reglas a valores que puedas justificar (no dejarlos como están aquí).
- [ ] Decidir la regla del "sobregiro" (bloquear vs. advertir) y dejarla escrita.
- [ ] Revisar que cada funcionalidad del punto 3 empiece con un verbo y sea programable.
- [ ] Confirmar que todos los casos límite del punto 6 están resueltos.
