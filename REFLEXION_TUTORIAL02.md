# Reflexión - Tutorial 02: Patrones Creacionales

## Factory Method

`PaymentFactory` centraliza la decisión de qué procesador de pagos utilizar.
La vista y el servicio no necesitan conocer si se está usando el banco real o
un procesador simulado. El comportamiento se cambia mediante la variable de
entorno `PAYMENT_PROVIDER`, sin modificar el código de la vista.

## Builder

`OrdenBuilder` reduce el riesgo de errores porque permite construir una orden
paso a paso mediante una interfaz fluida (`con_usuario`, `con_libro`,
`con_cantidad` y `para_envio`). Además, concentra las validaciones y el cálculo
del total en un único componente. Si la orden se construyera directamente en
la vista, la vista tendría que conocer todos los campos obligatorios, calcular
el total y validar los datos; eso aumentaría el acoplamiento y facilitaría que
se creen órdenes incompletas o inconsistentes. Con el Builder, la vista solo
coordina el flujo y el proceso de construcción puede reutilizarse desde otros
servicios o entradas de la aplicación.

## Archivos fuente entregados

- `tienda_app/infra/factories.py`: implementación de Factory Method.
- `tienda_app/domain/builders.py`: implementación de Builder.
- `tienda_app/services.py`: integración de ambos patrones en el flujo de compra.

