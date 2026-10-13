# Resumen de código - Tutorial 01 Django SOLID

## Evidencia de auditoría

El archivo [pagos_locales_JERONIMO_RAMIREZ.log](./pagos_locales_JERONIMO_RAMIREZ.log)
contiene tres transacciones exitosas generadas por el gateway de pagos.

## Separación de responsabilidades

- `views.py`: recibe la petición HTTP, delega al servicio y devuelve la respuesta.
- `services.py`: consulta el libro, valida el inventario, calcula el total, procesa el pago y descuenta el stock.
- `domain/logic.py`: contiene el cálculo del IVA.
- `infra/gateways.py`: implementa el procesador de pagos y escribe el log de auditoría.

## Código de `services.py`

La clase usada por la vista de Compra Rápida es `CompraRapidaService`:

```python
from django.shortcuts import get_object_or_404

from .domain.logic import CalculadorImpuestos
from .infra.factories import PaymentFactory
from .models import Inventario, Libro


class CompraRapidaService:
    def __init__(self, procesador_pago):
        self.procesador_pago = procesador_pago

    @classmethod
    def crear(cls):
        return cls(PaymentFactory.get_processor())

    def obtener_detalle(self, libro_id):
        libro = get_object_or_404(Libro, id=libro_id)
        total = CalculadorImpuestos.obtener_total_con_iva(libro.precio)
        return {"libro": libro, "total": total}

    def procesar(self, libro_id):
        libro = get_object_or_404(Libro, id=libro_id)
        inv = get_object_or_404(Inventario, libro=libro)

        if inv.cantidad <= 0:
            raise ValueError("No hay existencias.")

        total = CalculadorImpuestos.obtener_total_con_iva(libro.precio)

        if self.procesador_pago.pagar(total):
            inv.cantidad -= 1
            inv.save()
            return total

        return None
```

> Nota: en el archivo real `services.py` también se encuentra `CompraService`,
> utilizado por la vista y la API de compra avanzada. La clase anterior es la
> implementación correspondiente al flujo `CompraRapidaView` del tutorial.

## Código de `views.py`

La vista de Compra Rápida delega la lógica al servicio:

```python
class CompraRapidaView(View):
    template_name = "tienda_app/compra_rapida.html"

    def setup_service(self):
        return CompraRapidaService.crear()

    def get(self, request, libro_id):
        contexto = self.setup_service().obtener_detalle(libro_id)
        return render(request, self.template_name, contexto)

    def post(self, request, libro_id):
        libro = get_object_or_404(Libro, id=libro_id)
        try:
            total = self.setup_service().procesar(libro_id)
        except ValueError as error:
            return HttpResponse(str(error), status=400)
        return HttpResponse(
            f"Compra exitosa: {libro.titulo}. Total: ${total}"
        )
```

La vista no calcula impuestos, no modifica el inventario y no escribe archivos
de log. Esas responsabilidades fueron extraídas al servicio y a la
infraestructura, respectivamente.

## Flujo de ejecución

1. El navegador solicita `/compra-rapida/<libro_id>/`.
2. `CompraRapidaView` crea el servicio mediante `PaymentFactory`.
3. `CompraRapidaService` valida el stock y calcula el total con IVA.
4. El gateway `BancoNacionalProcesador` registra la transacción en el archivo
   `.log`.
5. El servicio descuenta una unidad del inventario.
6. La vista devuelve el resultado de la compra.
