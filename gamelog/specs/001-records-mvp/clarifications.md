# Aclaraciones iniciales de GameLog

Decisiones tomadas con el propietario del proyecto el 23 de octubre de 2026, antes y durante la redacción de la especificación del MVP.

| ID | Decisión |
| --- | --- |
| D-1 | El registro es personal: una persona anota únicamente sus propios resultados. |
| D-2 | Se pueden incluir varios videojuegos. |
| D-3 | Un juego puede tener varios tipos de récord; cada tipo compara sus resultados por el valor más alto o el más bajo. |
| D-4 | Cada resultado se registra por separado, aunque proceda de la misma partida. |
| D-5 | Hay tipos predefinidos, como puntuación y tiempo, y se pueden crear tipos propios. «Score» y «puntuación» se tratan como el mismo tipo. |
| D-6 | La fecha de un resultado es automáticamente el día en que se registra; no se elige otra fecha. |
| D-7 | Los tiempos se introducen en formato `mm:ss.cc`, por ejemplo `03:42.58`. |
| D-8 | La puntuación acepta enteros desde `0`; los tipos propios aceptan números desde `0` con hasta dos decimales. |
| D-9 | Los nombres de juegos y los nombres de tipos dentro de un juego se comparan ignorando mayúsculas y espacios al principio o al final; los duplicados se rechazan. |
| D-10 | Un resultado registrado por error se puede editar o borrar; después de borrarlo se puede registrar otro resultado. |
| D-11 | Los juegos también se pueden editar y borrar. |
| D-12 | Los tipos de récord creados por el jugador se pueden editar y borrar; los tipos predefinidos «Puntuación» y «Tiempo» no se pueden editar ni borrar. |
| D-13 | Borrar un juego requiere confirmación y elimina todos sus tipos y resultados asociados. |
| D-14 | Borrar un tipo propio requiere confirmación y elimina sus resultados; el juego y los demás tipos y resultados se conservan. |
| D-15 | Al editar un resultado solo cambia su valor; se conservan el juego, el tipo y la fecha original de registro. |
| D-16 | El historial de cada juego y tipo muestra primero la mejor marca; el jugador también puede ordenarlo por fecha, desde el más antiguo o desde el más nuevo. |

Los detalles que estas decisiones no resuelven se identificarán en `spec.md` con `[NECESITA ACLARACIÓN]`.
