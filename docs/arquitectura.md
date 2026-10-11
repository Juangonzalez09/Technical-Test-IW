# Arquitectura

Diagrama interactivo: [arquitectura.html](arquitectura.html) (abrir en el navegador; tiene tema claro y oscuro y exportación a PNG/SVG).

## Flujo

```
 Archivos de cada miembro         Google Colab              Azure SQL                    Power BI
 (txt, xlsx) + presupuesto   ─►   01_etl.ipynb   ─►   silver: tablas 3FN   ─►   gold: vistas   ─►   tablero
        bronze                  extract/transform/load                                  02_reportes.ipynb
```

## Capas

| Capa | Dónde | Qué guarda | Regla |
|---|---|---|---|
| Bronze | `data/bronze/` | Los archivos tal como llegan | No se modifican nunca |
| Silver | Tablas en la base de datos | Datos limpios, tipados y normalizados (3FN) | Solo entra lo que pasó por Transform |
| Gold | Vistas en la base de datos | La lógica de cada reporte | Es lo único que leen los reportes y Power BI |

Por qué separar en capas:

- Si cambia una regla de limpieza, se reprocesa desde bronze sin pedir los archivos otra vez.
- Un error se ubica rápido: o viene en el archivo, o en la limpieza, o en la consulta.
- Power BI no depende del formato de los archivos. Si el hijo pasa de Excel a CSV, solo cambia Extract.
- Las reglas del negocio (qué es "no presupuestado", cómo se calcula el ahorro) están en un solo lugar: las vistas.

## Modelo

```
miembro ─┐
         ├─< movimiento >─ rubro ─< presupuesto >─ periodo
forma_pago ┘
```

- `movimiento`: un gasto o ingreso de un miembro. Su periodo se obtiene de la fecha.
- `presupuesto`: valor planeado por periodo y rubro.
- `rubro` tiene tipo (ingreso, ahorro, egreso). Los rubros que no están en el presupuesto también entran al catálogo, para poder reportarlos.

## Crecimiento

**5 años de información.** Hoy son unos 190 movimientos por mes. En 60 meses son unos 12.000 registros, y aunque la familia crezca a 10 miembros no pasa de 100.000. Una base relacional como Azure SQL (incluso la capa gratuita, de 32 GB) lo maneja sin problema. Herramientas como Spark o Databricks serían sobredimensionar.

**Más miembros.** Un miembro nuevo es una fila en `miembro` y un archivo `Gastos_<Nombre>_<AAAAMM>`. El ETL lee todos los archivos que sigan ese patrón y el reporte por miembro arma las columnas según los datos. No hay que cambiar código.

**Más meses.** Se agrega la hoja del mes al presupuesto y los archivos del mes. `periodo` se crea a partir de lo que llegue.

## Qué cambiaría para producción

Esto queda fuera del alcance de la prueba, pero es el siguiente paso natural:

- **Cargue incremental**: hoy se recrean las tablas en cada ejecución (son pocos datos). Con más volumen se cargaría solo el periodo nuevo.
- **Programar el ETL**: correrlo automáticamente cuando llega un archivo, por ejemplo con Azure Data Factory o un pipeline de Fabric.
- **Validar en la entrada**: la mayoría de problemas vienen de texto libre (rubros mal escritos, `8MIL PESOS`). Un formulario con listas desplegables, en Power Apps o Google Forms, los evita desde el origen.
