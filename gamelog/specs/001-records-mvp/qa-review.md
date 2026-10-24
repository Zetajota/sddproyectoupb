# Revisión de QA — Spec 001 de GameLog

- **Documento revisado:** `spec.md`, borrador del 23 de octubre de 2026 (decisiones D-1 a D-16).
- **Referencia:** `docs/constitution.md` y `clarifications.md`.
- **Alcance:** detección de problemas; este documento no propone soluciones.

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
