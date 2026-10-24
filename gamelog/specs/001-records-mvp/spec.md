# Spec 001 — Registro personal de resultados de GameLog

- **Estado:** borrador para revisión
- **Fecha:** 23 de octubre de 2026
- **Origen:** [constitución](../../docs/constitution.md) y [decisiones D-1 a D-16](clarifications.md)

## 1. Contexto y objetivo

Una persona puede jugar a varios videojuegos y querer recordar sus puntuaciones, tiempos y otras marcas. Si anota resultados en lugares separados, le cuesta comparar intentos y saber cuál es su récord en cada juego.

El objetivo del MVP es registrar resultados personales, conservar su historial y mostrar el mejor resultado según la regla de comparación de cada tipo de récord.

## 2. Usuarios

- **Jugador:** única persona que utiliza su registro y anota solo sus propios resultados. No necesita registrar perfiles de otros jugadores.

## 3. Historias de usuario

- **HU-1:** Como jugador, quiero añadir varios juegos para mantener separados sus resultados.
- **HU-2:** Como jugador, quiero usar tipos de récord predefinidos o crear otros para medir distintas marcas de un mismo juego.
- **HU-3:** Como jugador, quiero guardar cada resultado con la fecha de registro para consultar mis intentos anteriores.
- **HU-4:** Como jugador, quiero ver mi mejor resultado por juego y tipo de récord para reconocer mi progreso.

## 4. Requisitos funcionales

Los criterios de aceptación usan «Cuando… el sistema deberá…», «Mientras… el sistema deberá…» y «Si… entonces el sistema deberá…». Los asuntos todavía no decididos figuran en la sección 10.

### RF-1 — Juegos

- **RF-1.1.** Cuando el jugador añada un juego con un nombre que, tras quitar espacios al principio y al final, no esté vacío ni coincida con otro juego ignorando mayúsculas, el sistema deberá conservar el nombre sin esos espacios para registrar resultados asociados a él.
- **RF-1.2.** Cuando el jugador consulte sus juegos, el sistema deberá mostrar todos los juegos registrados.
- **RF-1.3.** Si el jugador intenta registrar un resultado para un juego inexistente, entonces el sistema deberá rechazarlo y explicar el error sin guardar el resultado.
- **RF-1.4.** Si el nombre de un juego queda vacío tras quitar los espacios extremos o coincide con otro juego ignorando mayúsculas, entonces el sistema deberá rechazar el alta y explicar el error.
- **RF-1.5.** Cuando el jugador cambie el nombre de un juego existente por otro válido y no duplicado, el sistema deberá mostrar el nombre nuevo y mantener asociados sus tipos y resultados.
- **RF-1.6.** Cuando el jugador confirme el borrado de un juego existente, el sistema deberá eliminarlo junto con todos sus tipos propios y resultados asociados; dejarán de aparecer en las consultas.
- **RF-1.7.** Si el jugador cancela el borrado de un juego, entonces el sistema deberá conservar el juego, sus tipos y todos sus resultados.

### RF-2 — Tipos de récord

- **RF-2.1.** Para cada juego, el sistema deberá ofrecer los tipos predefinidos «Puntuación» (mejora al subir) y «Tiempo» (mejora al bajar). «Score» se entiende como puntuación, no como un tercer tipo.
- **RF-2.2.** Cuando el jugador cree un tipo propio para un juego, el sistema deberá guardar su nombre sin espacios extremos y si mejora al subir o al bajar.
- **RF-2.3.** Cuando el jugador consulte los tipos de un juego, el sistema deberá mostrar los predefinidos y los tipos propios creados para ese juego.
- **RF-2.4.** Si el jugador intenta registrar un resultado para un tipo que no pertenece al juego elegido, entonces el sistema deberá rechazarlo sin guardar el resultado.
- **RF-2.5.** Si el nombre de un tipo propio queda vacío tras quitar los espacios extremos o coincide, ignorando mayúsculas, con otro tipo propio o predefinido del mismo juego, entonces el sistema deberá rechazarlo y explicar el error.
- **RF-2.6.** Cuando el jugador edite un tipo propio, el sistema deberá permitir cambiar su nombre por otro válido y no duplicado, o cambiar si mejora al subir o al bajar. Los resultados existentes seguirán asociados a ese tipo.
- **RF-2.7.** Cuando el jugador confirme el borrado de un tipo propio, el sistema deberá eliminar ese tipo y sus resultados asociados, conservando el juego, sus demás tipos y los resultados de estos.
- **RF-2.8.** Si el jugador intenta editar o borrar «Puntuación» o «Tiempo», entonces el sistema deberá rechazar la acción sin cambiar el tipo ni sus resultados.
- **RF-2.9.** Si el jugador cancela el borrado de un tipo propio, entonces el sistema deberá conservar el tipo y todos sus resultados.

### RF-3 — Registro de resultados

- **RF-3.1.** Cuando el jugador registre un resultado válido, el sistema deberá asociarlo a un único juego y a un único tipo de récord.
- **RF-3.2.** Cuando el jugador registre un resultado, el sistema deberá asignarle automáticamente la fecha de ese día; el jugador no deberá poder escoger ni cambiar la fecha de registro.
- **RF-3.3.** Mientras un juego tenga varios tipos de récord, el sistema deberá conservar cada resultado como una entrada independiente, incluso si varios proceden de la misma partida.
- **RF-3.4.** Cuando el jugador registre un tiempo, el sistema deberá aceptarlo en formato `mm:ss.cc`, con minutos no negativos, segundos entre `00` y `59` y centésimas entre `00` y `99`. El tiempo total deberá ser mayor que cero.
- **RF-3.5.** Cuando el jugador registre una puntuación, el sistema deberá aceptar únicamente un número entero mayor o igual a `0`.
- **RF-3.6.** Cuando el jugador registre un resultado de tipo propio, el sistema deberá aceptar únicamente un número mayor o igual a `0` con un máximo de dos cifras decimales.
- **RF-3.7.** Si un valor no cumple las reglas de su tipo (RF-3.4 a RF-3.6), entonces el sistema deberá explicar el error y no guardar el resultado.
- **RF-3.8.** Cuando el jugador edite el valor de un resultado existente y el nuevo valor cumpla las reglas de su tipo, el sistema deberá conservar el nuevo valor en lugar del anterior, manteniendo su juego, tipo y fecha original de registro. No se podrán cambiar esos otros datos mediante la edición.
- **RF-3.9.** Cuando el jugador borre un resultado existente, el sistema deberá quitarlo del historial y dejar de considerarlo para el mejor resultado; el jugador podrá registrar un resultado nuevo después.

### RF-4 — Historial y mejor resultado

- **RF-4.1.** Cuando el jugador consulte los resultados de un juego y un tipo, el sistema deberá mostrar todos los resultados registrados para esa combinación, con su valor y fecha, ordenados por mejor marca de forma predeterminada: de mayor a menor si el tipo mejora al subir y de menor a mayor si mejora al bajar. En un empate, el resultado registrado más tarde aparecerá primero.
- **RF-4.2.** Cuando el jugador consulte el récord de un tipo que mejora al subir, el sistema deberá mostrar el mayor valor registrado para ese juego y tipo.
- **RF-4.3.** Cuando el jugador consulte el récord de un tipo que mejora al bajar, el sistema deberá mostrar el menor valor registrado para ese juego y tipo.
- **RF-4.4.** Si aún no hay resultados para un juego y tipo, entonces el sistema deberá indicar que todavía no existe un récord.
- **RF-4.5.** Cuando el jugador elija el orden por fecha, el sistema deberá mostrar los mismos resultados desde el más antiguo o desde el más nuevo, según su elección. Si dos resultados tienen la misma fecha, se ordenarán por el momento en que fueron registrados.

## 5. Requisitos no funcionales

- **RNF-1 — Comprensión:** los mensajes dirigidos al jugador deberán estar en español e indicar claramente si una operación tuvo éxito o por qué falló.
- **RNF-2 — Conservación:** al volver a usar GameLog, deberán seguir disponibles los juegos, tipos propios y resultados que se guardaron correctamente.
- **RNF-3 — Comprobación:** cada requisito funcional y cada caso límite acordado deberá tener al menos una prueba reproducible con fechas controladas.
- **RNF-4 — Simplicidad:** la primera versión deberá poder funcionar sin registrar cuentas de otros jugadores.

## 6. Casos límite por comprobar

- **CL-1:** un juego no tiene resultados: se muestra el estado sin récord (RF-4.4).
- **CL-2:** dos resultados del mismo tipo tienen el mismo mejor valor: ambos permanecen en el historial (RF-3.3, RF-4.1).
- **CL-3:** el jugador registra varios resultados del mismo juego y tipo el mismo día: cada uno conserva esa fecha y se compara por separado (RF-3.2, RF-3.3).
- **CL-4:** un juego tiene a la vez puntuaciones, tiempos y tipos propios: los resultados y récords se calculan por tipo, sin mezclarlos (RF-2.1, RF-2.2, RF-4.2, RF-4.3).
- **CL-5:** un tipo propio mejora al bajar: su récord es el menor valor, igual que en «Tiempo» (RF-2.2, RF-4.3).
- **CL-6:** se intenta usar un tipo que pertenece a otro juego: se rechaza sin crear el resultado (RF-2.4).
- **CL-7:** se intenta guardar `00:00.00` o un tiempo con `60` segundos: se rechaza sin crear el resultado (RF-3.4, RF-3.7).
- **CL-8:** se intenta guardar una puntuación negativa, decimal o un valor propio con más de dos decimales: se rechaza sin crear el resultado (RF-3.5, RF-3.6, RF-3.7).
- **CL-9:** `Tetris`, `tetris` y ` Tetris ` no pueden registrarse como juegos distintos; tampoco `Puntuación` y ` puntuación ` como tipos distintos del mismo juego (RF-1.4, RF-2.5).
- **CL-10:** intentar borrar o editar «Puntuación» o «Tiempo» no cambia los tipos predefinidos ni sus resultados (RF-2.8).
- **CL-11:** borrar un juego con resultados elimina sus tipos y todos sus resultados; cancelar el borrado no cambia nada (RF-1.6, RF-1.7).
- **CL-12:** borrar un tipo propio con resultados elimina solo ese tipo y sus resultados; cancelar el borrado no cambia nada (RF-2.7, RF-2.9).
- **CL-13:** el historial de un tipo que mejora al bajar pone el tiempo menor primero; al elegir «más nuevo» o «más antiguo» ordena por registro sin cambiar qué resultado es el récord (RF-4.1, RF-4.3, RF-4.5).

## 7. Fuera de alcance del MVP

- Cuentas o resultados de otras personas; clasificaciones compartidas.
- Importación automática de datos desde videojuegos o plataformas externas.
- Sincronización entre dispositivos.
- Agrupar varios tipos de resultado en una única entrada de partida: cada resultado se registra por separado.

## 8. Criterios de finalización

1. Cada RF y CL acordado tiene al menos una prueba con resultado satisfactorio.
2. Los datos guardados siguen disponibles en un uso posterior.
3. La comparación del mayor y del menor valor funciona por juego y tipo, incluidos los tipos propios.
4. La fecha se asigna automáticamente al registrar un resultado y no puede sustituirse por otra elegida por el jugador.
5. No quedan dudas abiertas que alteren el comportamiento del MVP.

## 9. Trazabilidad de decisiones

| Decisión | Requisitos principales |
| --- | --- |
| D-1 y D-2 | RF-1, RNF-4 |
| D-3 y D-5 | RF-2, RF-4 |
| D-4 | RF-3.1, RF-3.3 |
| D-6 | RF-3.2 |

## 10. Dudas abiertas

No quedan dudas abiertas de producto identificadas en esta primera redacción. La revisión de QA puede detectar otras.
