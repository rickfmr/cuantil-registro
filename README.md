# Registro de pronósticos de Cuantil

Este repositorio guarda, cada vez que cambia, la **huella digital** del registro de pronósticos de Cuantil.
No contiene los pronósticos: solo prueba que existían en una fecha.

## Cómo funciona

- Cada pronóstico se guarda en un archivo de solo agregar (`pronosticos.jsonl`). Cada línea incluye el
  SHA-256 de la línea anterior (`hash_anterior`) y su propio `hash`, calculado sobre el JSON con las llaves
  ordenadas.
- `cabezas.csv` tiene una fila por publicación: fecha UTC, número de registros y el `hash` de la última línea.
  Como cada hash depende de todos los anteriores, ese único valor amarra el registro completo hasta ese momento.
- El historial de commits de GitHub fija cuándo se publicó cada fila.

## Cómo verificar

Con el archivo `pronosticos.jsonl` (lo publica Cuantil):

1. Recalcula la cadena: para cada línea, quita `hash`, serializa con llaves ordenadas
   (`json.dumps(registro, sort_keys=True, ensure_ascii=False)`), calcula SHA-256 y comprueba que coincide con
   `hash` y con el `hash_anterior` de la línea siguiente.
2. Toma las primeras *N* líneas, donde *N* es la columna `registros` de una fila de `cabezas.csv`; el `hash` de
   la línea *N* debe ser el `ultimo_hash` de esa fila.

Si coincide, esos *N* pronósticos ya existían, sin cambios, en la fecha de esa fila.

## Sello independiente (OpenTimestamps)

Cada fila nueva de `cabezas.csv` también se guarda en `sellos/<fecha>_<hash>.txt` y se sella con
[OpenTimestamps](https://opentimestamps.org/): su hash queda anclado en la blockchain de Bitcoin, así que la
fecha no depende de confiar en Cuantil ni en GitHub. El sello tarda unas horas en completarse (el archivo
`.ots` se actualiza solo en las corridas siguientes).

Para verificar: arrastra el `.txt` y su `.ots` a https://opentimestamps.org, o con el cliente oficial
`ots verify sellos/<archivo>.txt.ots`. Debe mostrar la hora del bloque de Bitcoin que lo contiene.
