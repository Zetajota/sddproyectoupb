# Revisión de QA — Spec 001 de GameLog

- **Documento revisado:** `spec.md`, versión 1 del 23 de octubre de 2026 (decisiones D-1 a D-16).
- **Referencia:** `docs/constitution.md` y `clarifications.md`.
- **Alcance:** las secciones 1 a 4 solo detectan problemas. La sección 5 registra sus resoluciones posteriores en la versión 2 de la spec.

## 1. Ambigüedades restantes

1. **RF-1.1, RF-1.4, RF-2.2 y RF-2.5 — nombres:** «espacios» y «mayúsculas» no indican cómo tratar tabuladores, saltos de línea, caracteres de control, acentos o caracteres Unicode equivalentes. Tampoco hay longitud máxima. Dos implementaciones podrían aceptar nombres diferentes.
2. **RF-2.1 y RF-2.5 — «Score»:** se declara sinónimo de «Puntuación», pero RF-2.5 solo prohíbe duplicar el nombre visible del tipo predefinido. No queda claro si se puede crear un tipo propio llamado «Score».
3. **RF-3.4 — formato de tiempo:** `mm:ss.cc` no aclara si los minutos exigen exactamente dos cifras, si se aceptan más de 99 minutos, ni si `3:42.58` es equivalente a `03:42.58`.
4. **RF-3.5 y RF-3.6 — números:** no se define el separador decimal, la interpretación de ceros a la izquierda ni un límite máximo de magnitud. Un resultado como `1,50` puede aceptarse en una interfaz y rechazarse en otra.
5. **RF-3.2 — fecha actual:** «ese día» no identifica la zona horaria que rige el cambio de fecha ni qué ocurre si el registro empieza antes de medianoche y termina después.
6. **RF-1.5, RF-2.6 y RF-3.8 — errores al editar:** las reglas de error para alta y registro no dicen expresamente qué ocurre si una edición introduce un nombre duplicado o un valor inválido; tampoco cómo se identifica una entrada concreta cuando hay resultados iguales.
7. **RF-2.6 frente a RF-4.1 a RF-4.3 — cambio de dirección:** se conservan los resultados al cambiar «mejora al subir» por «mejora al bajar», pero no se explicita cuándo se recalculan el récord y el orden del historial.
8. **RF-4.1 y RF-4.5 — empates de fecha:** se exige ordenar por «momento en que fueron registrados», mientras que RF-3.2 solo exige asignar una fecha. No se define qué dato permite distinguir el orden de dos entradas del mismo día.
9. **RF-4.1 — tamaño del historial:** «todos los resultados» no fija qué se muestra si el historial es muy grande ni si el jugador puede recorrerlo completo.
10. **RNF-2 y RF-1.6/RF-2.7/RF-3.9 — fallos al guardar:** no se indica qué ve el jugador ni en qué estado quedan los datos si la escritura falla durante un alta, edición o borrado.

## 2. Contradicciones entre requisitos

No se detectaron contradicciones directas entre los RF de esta versión. La relación entre los tipos predefinidos protegidos por **RF-2.8** y el borrado de su juego en **RF-1.6** necesita precisión, aunque ambas reglas pueden coexistir.

## 3. Casos límite no cubiertos

11. **RF-1.2, RF-2.3, RF-4.4:** no se cubre la consulta cuando aún no existe ningún juego, ni cuando un juego solo tiene los tipos predefinidos y no tiene resultados.
12. **RF-1.5, RF-1.6, RF-2.6, RF-2.7, RF-3.8 y RF-3.9:** no se cubre editar o borrar un juego, tipo o resultado que ya no existe, ni repetir un borrado.
13. **RF-3.9, RF-4.2 a RF-4.4:** no se cubre borrar el resultado que era récord, borrar el último resultado o editar un récord hasta que otro pase a ser el mejor.
14. **RF-2.6, RF-4.1 a RF-4.3:** no se cubre cambiar la dirección de un tipo propio con resultados existentes y consultar inmediatamente el nuevo orden y récord.
15. **RF-1.6 y RF-2.7:** no se cubre un fallo durante el borrado de un juego o tipo con muchos resultados, cuando podría quedar solo una parte eliminada.
16. **RF-3.2 y RF-4.5:** no se cubren dos registros hechos alrededor de medianoche o varios registros idénticos el mismo día al alternar los tres órdenes del historial.

## 4. Conflictos con la constitución

No se detectaron conflictos directos. La constitución exige pruebas para cada RF y caso límite, y centralizar la persistencia; la spec todavía no precisa el comportamiento ante fallos de persistencia (hallazgo 10), lo que puede dificultar una verificación completa.

## 5. Resoluciones aplicadas a la versión 2

Las decisiones de alcance D-17 (página web) y D-18 (uso local) fueron dadas por el jugador después de la revisión. Las demás resoluciones precisan comportamientos necesarios para que los requisitos se puedan probar; no añaden funciones principales nuevas.

| Hallazgo | Resolución en la spec versión 2 |
| --- | --- |
| 1 | Regla común de nombres: Unicode NFC, espacios extremos, comparación sin mayúsculas, controles prohibidos y máximo de 80 caracteres; RF-1.1, RF-1.4 y RF-2.5. |
| 2 | «Score» queda como alias de «Puntuación» y no puede crearse como tipo propio; RF-2.1 y RF-2.5. |
| 3 | Tiempo con dos a seis cifras de minutos, dos de segundos y dos de centésimas; RF-3.4. |
| 4 | Dígitos ASCII, punto decimal para tipos propios, reglas de ceros y límite de doce cifras enteras; RF-3.5 y RF-3.6. |
| 5 | La fecha local de la computadora se lee una vez al confirmar el registro; RF-3.2 y CL-19. |
| 6 | Las ediciones inválidas no cambian datos y cada entrada puede identificarse para editarla o borrarla; RF-1.4, RF-2.5, RF-3.8, RF-3.10 y RF-3.11. |
| 7 | Cambiar la dirección recalcula inmediatamente récord y orden con los resultados existentes; RF-2.6 y CL-15. |
| 8 | Se conserva la posición original de registro para desempatar entradas del mismo día; RF-3.8 y RF-4.5. |
| 9 | El historial completo se muestra sin paginación en el MVP; RF-4.1. |
| 10 | Un fallo de guardado deja intactos los datos anteriores y muestra un error; RNF-7 y CL-17. |
| 11 | Se define el estado sin juegos y el estado sin récord; RF-1.2 y RF-4.4. |
| 12 | Ediciones, borrados y consultas de entradas inexistentes dan error sin cambios; RF-1.8, RF-1.9, RF-2.10, RF-2.11 y RF-3.10. |
| 13 | Se comprueba el recálculo tras borrar o editar un récord y el estado tras borrar el último; CL-14 y CL-21. |
| 14 | Se comprueba el cambio de dirección de un tipo propio con historial existente; CL-15. |
| 15 | El borrado con datos asociados es completo o no ocurre; RNF-7 y CL-17. |
| 16 | Se comprueban registros del mismo día y alrededor de medianoche en los distintos órdenes; CL-18 y CL-19. |

La protección de «Puntuación» y «Tiempo» se precisó en RF-2.8: no se pueden editar ni borrar por separado, pero sus resultados desaparecen cuando se borra el juego completo. No quedan conflictos directos con la constitución en esta versión.
