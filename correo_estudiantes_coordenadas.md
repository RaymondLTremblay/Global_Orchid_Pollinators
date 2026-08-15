**Asunto:** Coordenadas de polinizadores de orquídeas — arrancamos el lunes 17 de agosto

Hola Naan, Natalia y Caleb:

Les escribo para arrancar formalmente la fase de georreferenciación del proyecto de polinizadores de orquídeas. Adjunto el archivo **`coordinate_assignment_tracker.xlsx`**, que tiene la asignación completa de cada uno.

## De qué se trata

La base de datos tiene ~3,127 especies de orquídeas con registros de polinización tomados de la literatura, pero solo **171 tienen coordenadas**. Sin coordenadas no podemos mapear nada, así que ese es el cuello de botella del proyecto entero. La meta de esta fase es cerrar esa brecha empezando por las Américas; Asia queda para una fase posterior.

## Tu asignación

Cada uno tiene su propia pestaña en el archivo:

| | Asignación | Total |
|---|---|---|
| **Naan** | Las 60 especies de Vanilloideae (semanas 1–3), luego se suma a las Américas (semanas 4–14) | 286 especies |
| **Natalia** | Especies de las Américas (semanas 1–14) | 280 especies |
| **Caleb** | Especies de las Américas (semanas 1–14) | 286 especies |

Son **852 especies en total**, a un ritmo de **18–22 especies por persona por semana durante 14 semanas**. Arrancamos el **lunes 17 de agosto** y la primera entrega es el **viernes 21 de agosto**.

Las asignaciones están repartidas por género completo a propósito: las especies de un mismo género suelen venir del mismo artículo y de la misma región, así que investigándolas juntas se avanza mucho más rápido.

Naan: empiezas con Vanilloideae porque es la única subfamilia que no tiene ni un solo punto georreferenciado y por eso está ausente de todos los mapas. Es la pieza que más falta hace.

## Cómo llenarlo

1. Abre **tu propia pestaña** (Naan / Natalia / Caleb) y trabaja de arriba hacia abajo, una semana a la vez.
2. Borra la fila de ejemplo antes de empezar. Está ahí solo para mostrar el formato esperado.
3. Empieza por la columna **`Reference(s) in dataset`**. Ese es el artículo del que salió el registro de polinización y es el primer lugar donde buscar la localidad de la especie.
4. Llena **únicamente las columnas amarillas**: Latitude, Longitude, Coordinate source, Precision, Date done, Status, Notes. Las columnas blancas (taxón, región, pista de localidad, referencia) no se tocan.
5. Usa **grados decimales**: `-22.9068, -43.1729`. Sur y Oeste van en negativo. Nada de grados-minutos-segundos.
6. En **`Coordinate source`** pon la cita exacta o el URL de donde sacaste las coordenadas — no basta con decir "GBIF". Necesitamos poder reconstruir de dónde salió cada punto.
7. En **`Precision`** indica si el punto es de sitio, localidad o región.
8. Usa el menú desplegable de **`Status`**: Not started / In progress / Done / Cannot find.

La pestaña `Progress` se actualiza sola y lleva la cuenta de todos.

## Sobre "Cannot find"

Si el artículo no da la localidad, o no encuentras la especie, o la referencia no está disponible: **marca `Cannot find` y explica por qué en `Notes`**. No inventes un punto ni pongas el centroide del país para llenar la casilla.

Una coordenada mal puesta es peor que una casilla vacía, porque después nadie sabe que hay que revisarla. Un "no encontré la localidad, el artículo solo dice 'Ecuador'" es información útil y la vamos a usar. Ya nos pasó: hay dos registros en la base (*Govenia bella* y *Acianthera ochreata*) con coordenadas que caen en el país equivocado, y costó más trabajo detectarlas que si nunca se hubieran llenado.

Si una localidad es ambigua pero tienes una interpretación razonable, ponla y explícala en `Notes`. Yo la reviso.

## Control de versiones — importante

Esto es lo que más dolores de cabeza nos ha dado, así que va en serio:

- **Un solo archivo por persona.** Trabaja siempre sobre el mismo, no hagas copias paralelas.
- **No le cambies el nombre.** Ni le agregues `_v2`, `_final`, `_1`, ni la fecha.
- **No lo exportes desde Numbers ni desde Google Sheets.** Guárdalo como `.xlsx` de Excel. Las exportaciones de Numbers agregan pestañas extra y cambian el formato de las celdas, y eso rompe el código que lee el archivo.
- **No agregues, borres ni reordenes filas o columnas**, aunque parezca que sobra alguna.
- **Cuando me lo mandes, dime la fecha del último cambio y hasta qué semana llegaste.** Las fechas de los archivos adjuntos no son confiables y no me dejan saber cuál versión es la más reciente.
- **Mándalo cada dos semanas**, aunque no hayas terminado el bloque completo.

## Preguntas

Si algo no está claro, pregunten antes de llenar veinte filas con el criterio equivocado. Prefiero contestar diez correos ahora que rehacer un bloque después.

Saludos,
Ray
