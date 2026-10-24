# Spec 001 — Registro personal de resultados de GameLog

- **Estado:** versión 2 para revisión
- **Fecha:** 23 de octubre de 2026
- **Origen:** [constitución](../../docs/constitution.md), [decisiones D-1 a D-18](clarifications.md) y [revisión de QA](qa-review.md)

## 1. Contexto y objetivo

Una persona puede jugar a varios videojuegos y querer recordar sus puntuaciones, tiempos y otras marcas. Si anota resultados en lugares separados, le cuesta comparar intentos y saber cuál es su récord en cada juego.

El objetivo del MVP es registrar resultados personales desde una página web, conservar su historial y mostrar el mejor resultado según la regla de comparación de cada tipo de récord.

## 2. Usuarios

- **Jugador:** única persona que utiliza su registro desde el navegador de su computadora y anota solo sus propios resultados. No necesita registrar perfiles de otros jugadores.

## 3. Historias de usuario

- **HU-1:** Como jugador, quiero añadir varios juegos para mantener separados sus resultados.
- **HU-2:** Como jugador, quiero usar tipos de récord predefinidos o crear otros para medir distintas marcas de un mismo juego.
- **HU-3:** Como jugador, quiero guardar cada resultado con la fecha de registro para consultar mis intentos anteriores.
- **HU-4:** Como jugador, quiero ver mi mejor resultado por juego y tipo de récord para reconocer mi progreso.
- **HU-5:** Como jugador, quiero corregir o borrar datos registrados por error para mantener mi historial exacto.

## 4. Requisitos funcionales

Los criterios de aceptación usan «Cuando… el sistema deberá…», «Mientras… el sistema deberá…» y «Si… entonces el sistema deberá…». Los asuntos todavía no decididos figuran en la sección 10.

Un nombre se normaliza a Unicode NFC, se le quitan los espacios Unicode de los extremos y se compara sin distinguir mayúsculas. Los acentos sí distinguen nombres. Se rechazan nombres vacíos, con caracteres de control o de más de 80 caracteres Unicode después de quitar espacios extremos. Los espacios interiores normales se conservan.

### RF-1 — Juegos

- **RF-1.1.** Cuando el jugador añada un juego con un nombre válido que no coincida con otro juego según la regla de nombres, el sistema deberá conservar el nombre normalizado para registrar resultados asociados a él.
- **RF-1.2.** Cuando el jugador consulte sus juegos, el sistema deberá mostrar todos los juegos registrados o indicar claramente que aún no hay juegos.
- **RF-1.3.** Si el jugador intenta registrar un resultado para un juego inexistente, entonces el sistema deberá rechazarlo y explicar el error sin guardar el resultado.
- **RF-1.4.** Si el nombre de un juego no cumple la regla de nombres o coincide con otro juego, entonces el sistema deberá rechazar el alta o la edición y explicar el error sin modificar los datos.
- **RF-1.5.** Cuando el jugador cambie el nombre de un juego existente por otro válido y no duplicado, el sistema deberá mostrar el nombre nuevo y mantener asociados sus tipos y resultados.
- **RF-1.6.** Cuando el jugador confirme el borrado de un juego existente, el sistema deberá eliminarlo junto con todos sus tipos propios y resultados asociados; dejarán de aparecer en las consultas.
- **RF-1.7.** Si el jugador cancela el borrado de un juego, entonces el sistema deberá conservar el juego, sus tipos y todos sus resultados.
- **RF-1.8.** Si el jugador intenta editar o borrar un juego que no existe, entonces el sistema deberá mostrar un error sin modificar los datos.
- **RF-1.9.** Si el jugador intenta consultar un juego que no existe, entonces el sistema deberá mostrar un error sin modificar los datos.

### RF-2 — Tipos de récord

- **RF-2.1.** Para cada juego, el sistema deberá ofrecer los tipos predefinidos «Puntuación» (mejora al subir) y «Tiempo» (mejora al bajar). «Score» identifica «Puntuación» y no se puede crear como un tercer tipo.
- **RF-2.2.** Cuando el jugador cree un tipo propio para un juego, el sistema deberá guardar su nombre sin espacios extremos y si mejora al subir o al bajar.
- **RF-2.3.** Cuando el jugador consulte los tipos de un juego, el sistema deberá mostrar los predefinidos y los tipos propios creados para ese juego.
- **RF-2.4.** Si el jugador intenta registrar un resultado para un tipo que no pertenece al juego elegido, entonces el sistema deberá rechazarlo sin guardar el resultado.
- **RF-2.5.** Si el nombre de un tipo propio no cumple la regla de nombres o coincide con otro tipo del mismo juego —incluido el alias «Score» de «Puntuación»—, entonces el sistema deberá rechazar el alta o la edición, explicar el error y conservar los datos anteriores.
- **RF-2.6.** Cuando el jugador edite un tipo propio, el sistema deberá permitir cambiar su nombre por otro válido y no duplicado, o cambiar si mejora al subir o al bajar. Los resultados existentes seguirán asociados a ese tipo; si cambia la dirección, el récord y el orden por mejor marca se recalcularán inmediatamente con todos esos resultados.
- **RF-2.7.** Cuando el jugador confirme el borrado de un tipo propio, el sistema deberá eliminar ese tipo y sus resultados asociados, conservando el juego, sus demás tipos y los resultados de estos.
- **RF-2.8.** Si el jugador intenta editar o borrar por separado «Puntuación» o «Tiempo», entonces el sistema deberá rechazar la acción sin cambiar el tipo ni sus resultados. Si se borra el juego completo, también desaparecen los resultados de sus tipos predefinidos.
- **RF-2.9.** Si el jugador cancela el borrado de un tipo propio, entonces el sistema deberá conservar el tipo y todos sus resultados.
- **RF-2.10.** Si el jugador intenta editar o borrar un tipo propio que no existe en el juego, entonces el sistema deberá mostrar un error sin modificar los datos.
- **RF-2.11.** Si el jugador intenta consultar un tipo que no existe en el juego, entonces el sistema deberá mostrar un error sin modificar los datos.

### RF-3 — Registro de resultados

- **RF-3.1.** Cuando el jugador registre un resultado válido, el sistema deberá asociarlo a un único juego y a un único tipo de récord.
- **RF-3.2.** Cuando el jugador confirme el registro de un resultado, el sistema deberá leer una sola vez la fecha local de su computadora y asignarla al resultado. El jugador no deberá poder escoger ni cambiar esa fecha, aunque el registro cruce la medianoche.
- **RF-3.3.** Mientras un juego tenga varios tipos de récord, el sistema deberá conservar cada resultado como una entrada independiente, incluso si varios proceden de la misma partida.
- **RF-3.4.** Cuando el jugador registre un tiempo, el sistema deberá aceptarlo en formato `mm:ss.cc`: de dos a seis cifras de minutos, exactamente dos de segundos (`00` a `59`) y dos de centésimas (`00` a `99`), separados por `:` y `.`. El tiempo total deberá ser mayor que cero.
- **RF-3.5.** Cuando el jugador registre una puntuación, el sistema deberá aceptar únicamente de uno a doce dígitos `0` a `9` que representen un entero mayor o igual a `0`, sin signo ni separadores.
- **RF-3.6.** Cuando el jugador registre un resultado de tipo propio, el sistema deberá aceptar únicamente de uno a doce dígitos `0` a `9` que representen un número mayor o igual a `0`, con un punto opcional seguido de una o dos cifras decimales; no se aceptan signos, comas ni exponentes. Los ceros a la izquierda se aceptan y no alteran el valor.
- **RF-3.7.** Si un valor no cumple las reglas de su tipo (RF-3.4 a RF-3.6), entonces el sistema deberá explicar el error y no guardar el resultado.
- **RF-3.8.** Cuando el jugador edite el valor de un resultado existente y el nuevo valor cumpla las reglas de su tipo, el sistema deberá conservar el nuevo valor en lugar del anterior, manteniendo su juego, tipo, fecha y posición original de registro. No se podrán cambiar esos otros datos mediante la edición. Si el valor es inválido, deberá conservarse el valor anterior y mostrarse un error.
- **RF-3.9.** Cuando el jugador borre un resultado existente, el sistema deberá quitarlo del historial y dejar de considerarlo para el mejor resultado; el jugador podrá registrar un resultado nuevo después.
- **RF-3.10.** Si el jugador intenta editar o borrar un resultado que no existe, entonces el sistema deberá mostrar un error sin modificar los datos.
- **RF-3.11.** Cuando el jugador consulte el historial, el sistema deberá permitir identificar inequívocamente cada resultado para poder editarlo o borrarlo, incluso si varios tienen el mismo valor y la misma fecha.

### RF-4 — Historial y mejor resultado

- **RF-4.1.** Cuando el jugador consulte los resultados de un juego y un tipo, el sistema deberá mostrar todos los resultados registrados para esa combinación, con su valor y fecha, ordenados por mejor marca de forma predeterminada: de mayor a menor si el tipo mejora al subir y de menor a mayor si mejora al bajar. En un empate, el resultado registrado más tarde aparecerá primero. Se mostrará el historial completo, aunque tenga muchos resultados.
- **RF-4.2.** Cuando el jugador consulte el récord de un tipo que mejora al subir, el sistema deberá mostrar el mayor valor registrado para ese juego y tipo.
- **RF-4.3.** Cuando el jugador consulte el récord de un tipo que mejora al bajar, el sistema deberá mostrar el menor valor registrado para ese juego y tipo.
- **RF-4.4.** Si aún no hay resultados para un juego y tipo, entonces el sistema deberá indicar que todavía no existe un récord.
- **RF-4.5.** Cuando el jugador elija el orden por fecha, el sistema deberá mostrar los mismos resultados desde el más antiguo o desde el más nuevo, según su elección. Si dos resultados tienen la misma fecha, se ordenarán por su posición original de registro, que no cambia al editar el valor.

## 5. Requisitos no funcionales

- **RNF-1 — Comprensión:** los mensajes dirigidos al jugador deberán estar en español e indicar claramente si una operación tuvo éxito o por qué falló.
- **RNF-2 — Conservación:** al volver a usar GameLog, deberán seguir disponibles los juegos, tipos propios y resultados que se guardaron correctamente.
- **RNF-3 — Comprobación:** cada requisito funcional y cada caso límite acordado deberá tener al menos una prueba reproducible con fechas controladas.
- **RNF-4 — Simplicidad:** la primera versión deberá poder funcionar sin registrar cuentas de otros jugadores.
- **RNF-5 — Acceso web:** el jugador deberá poder realizar las acciones del MVP desde un navegador mediante páginas y formularios en español.
- **RNF-6 — Uso local:** la primera versión solo deberá estar accesible desde la computadora donde se ejecuta; no necesitará cuentas ni conexión a servicios externos.
- **RNF-7 — Integridad:** si falla el guardado de una operación, los datos anteriores deberán mantenerse completos y el jugador deberá ver un error en español. Una edición o un borrado con datos asociados se aplicará por completo o no se aplicará.

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
- **CL-14:** al borrar el resultado que era récord, el siguiente mejor pasa a ser el récord; al borrar el último, se muestra el estado sin récord (RF-3.9, RF-4.2 a RF-4.4).
- **CL-15:** cambiar la dirección de un tipo propio con resultados existentes cambia inmediatamente qué valor es el récord y el orden por mejor marca (RF-2.6, RF-4.1 a RF-4.3).
- **CL-16:** un nombre vacío, con saltos de línea o con más de 80 caracteres tras normalizar se rechaza; «Score» no se crea como tipo propio (RF-1.4, RF-2.5).
- **CL-17:** si falla el guardado durante el borrado de un juego con resultados, todos los datos anteriores siguen presentes (RNF-7).
- **CL-18:** dos resultados idénticos del mismo día conservan su orden original al cambiar entre «mejor marca», «más antiguo» y «más nuevo» (RF-4.1, RF-4.5).
- **CL-19:** si el registro se confirma antes de medianoche y termina después, se usa la fecha leída al confirmarlo (RF-3.2).
- **CL-20:** una puntuación de trece dígitos, un valor propio con trece cifras enteras o un tiempo con siete cifras de minutos se rechazan (RF-3.4 a RF-3.7).
- **CL-21:** editar el valor del resultado que era récord puede hacer que otro pase a ser el mejor; una edición con valor inválido conserva el récord anterior (RF-3.8, RF-4.2, RF-4.3).

## 7. Fuera de alcance del MVP

- Cuentas o resultados de otras personas; clasificaciones compartidas.
- Importación automática de datos desde videojuegos o plataformas externas.
- Sincronización entre dispositivos.
- Publicación en internet y acceso desde otras computadoras.
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
| D-6 y D-18 | RF-3.2, RNF-6 |
| D-7 y D-8 | RF-3.4 a RF-3.7 |
| D-9 | RF-1.1, RF-1.4, RF-2.5 |
| D-10 a D-15 | RF-1.5 a RF-1.8, RF-2.6 a RF-2.10, RF-3.8 a RF-3.10 |
| D-16 | RF-4.1, RF-4.5 |
| D-17 | RNF-5 |

## 10. Dudas abiertas

No quedan dudas abiertas que bloqueen el plan técnico. Las resoluciones de los hallazgos de QA figuran en `qa-review.md`.
