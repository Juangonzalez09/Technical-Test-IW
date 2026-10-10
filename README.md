# Presupuesto familia Miranda

Prueba técnica IW (Builder). Cruza los gastos de papá, mamá e hijo contra el presupuesto de agosto y septiembre de 2023.

## Notebooks

| Notebook | Qué hace | Abrir |
|---|---|---|
| `01_etl` | Lee los archivos, los limpia, los carga al modelo y crea las vistas. Al final, los problemas del cargue. | [Colab](https://colab.research.google.com/github/Juangonzalez09/Technical-Test-IW/blob/main/notebooks/01_etl.ipynb) |
| `02_reportes` | Responde las preguntas de la prueba. | [Colab](https://colab.research.google.com/github/Juangonzalez09/Technical-Test-IW/blob/main/notebooks/02_reportes.ipynb) |

Los dos quedan guardados con sus resultados, así que se pueden leer sin ejecutarlos. Si se ejecutan en Colab, la primera celda clona este repo para leer los datos. `02_reportes` corre `01_etl` si la base no existe.

## Dónde está cada punto (Builder Senior)

| Punto | Tema | Dónde |
|---|---|---|
| 1 | Arquitectura y modelo de datos | [docs/arquitectura.md](docs/arquitectura.md) y `01_etl` §3 |
| 2 | Cargue desde Colab | `01_etl` §3 |
| 3 | Ejecución presupuestal por rubro y miembro | `02_reportes` · Punto 3 |
| 4 | Problemas del cargue | `01_etl` §5 |
| 5 y 6 | ¿Gasta más de lo que gana? ¿Alcanza para ahorrar? | `02_reportes` · Puntos 5 y 6 |
| 7 | Rubros por encima del presupuesto | `02_reportes` · Punto 7 |
| 8 | Forma de pago preferida | `02_reportes` · Punto 8 |
| 9 | Gastos no presupuestados | `02_reportes` · Punto 9 |
| 10 | Rubros sin uso | `02_reportes` · Punto 10 |

## Arquitectura

```
bronze   data/bronze/     archivos originales sin modificar
   │     ETL (01_etl)
   ▼
silver   base de datos    modelo relacional (3FN) con los datos limpios
   │     vistas SQL
   ▼
gold     reportes         consultas para 02_reportes y Power BI
```

Detalle del diseño, el modelo y el crecimiento a 5 años: [docs/arquitectura.md](docs/arquitectura.md).

La base es Azure SQL Database (oferta gratuita). La conexión se lee del secreto `DATABASE_URL` de Colab o de la variable de entorno con el mismo nombre. **Las credenciales no están en el repo**: sin ellas los notebooks usan un archivo SQLite local y dan los mismos resultados, porque el código y el SQL son los mismos.

## Estructura

```
data/bronze/     datos originales
notebooks/       ETL y reportes
docs/            arquitectura
```

## Cómo ejecutar en local

```bash
pip install -r requirements.txt
jupyter notebook notebooks/
```

## Decisiones de limpieza

- Rubros: se quitan espacios de más y se pasan a mayúsculas. Los nombres que cambian entre archivos se unifican con un diccionario de alias (por ejemplo, `TRANSPORTE CARRO / MOTO` → `TRANSPORTE CARRO/MOTO/GASOLINA`).
- `8MIL PESOS` se carga como 8000.
- Los contratos que vienen en los archivos de gastos se cargan como ingresos.
- Lo que no tiene rubro en el presupuesto (retiros de cajero, cuotas de manejo, la compra del carro) se carga igual y se reporta como gasto no presupuestado.
- Los gastos que aparecen repetidos entre miembros no se borran. Se señalan en `01_etl` y se muestra cuánto cambiaría el resultado.

## Tiempos

| Ejercicio | Minutos |
|---|---|
| | |
