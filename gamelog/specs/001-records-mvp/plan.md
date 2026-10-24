# Plan 001 — MVP web local de GameLog

- **Estado:** aprobado el 23 de octubre de 2026 (Python y Flask confirmados por el jugador)
- **Fecha:** 23 de octubre de 2026
- **Base:** [spec 001 v2 aprobada](spec.md), [constitución](../../docs/constitution.md) y [QA resuelto](qa-review.md)
- **Alcance:** diseño técnico; todavía no se implementa código.

## 1. Decisiones técnicas

| ID | Decisión y motivo | Alternativa descartada |
| --- | --- | --- |
| DT-1 | Python 3.12+ y Flask 3.1.x para una sola aplicación web local con plantillas HTML. Es un lenguaje para interfaz de servidor, aplicación y dominio; Flask permite probar las rutas sin levantar un servidor real. [RNF-5, RNF-6; principios 1 y 6] | React más una API: dos aplicaciones y más herramientas para el mismo MVP. |
| DT-2 | `sqlite3` de la biblioteca estándar y un único archivo SQLite en la carpeta personal del jugador. Sus transacciones y relaciones permiten borrar un juego y sus resultados como una operación. [RF-1.6, RF-2.7, RNF-2, RNF-7; principio 5] | Archivo JSON: las relaciones, la unicidad y los borrados completos requerirían más código propio. |
| DT-3 | HTML generado en el servidor, formularios y CSS local; sin framework de JavaScript. Las tres opciones de orden son enlaces que vuelven a cargar el historial. [RF-4.5, RNF-5; principio 1] | Aplicación de una sola página: añade estado y herramientas de cliente sin necesidad del MVP. |
| DT-4 | Guardar valores como enteros exactos: puntos enteros, tiempos en centésimas y tipos propios en centésimas. Nunca usar `float` para decidir un récord. [RF-3.4 a RF-3.6, RF-4.2, RF-4.3] | `float`: puede representar mal algunos decimales y alterar comparaciones o empates. |
| DT-5 | Leer la fecha local una vez al confirmar un resultado mediante un reloj inyectable. [RF-3.2, CL-19; principios 3 y 4] | Llamar al reloj desde la regla de negocio: haría las pruebas dependientes del día real. |
| DT-6 | Todos los cambios usan transacciones; los borrados relacionados usan claves foráneas con `ON DELETE CASCADE`. El récord se calcula desde los resultados vigentes y no se guarda como dato separado. [RF-1.6, RF-2.7, RF-3.9, RNF-7] | Guardar el récord en otra columna: podría quedar desactualizado tras editar o borrar. |
| DT-7 | El servidor escucha solo en `127.0.0.1`. Todos los formularios de cambio usan POST y un token CSRF; los GET solo leen. [RNF-6, RNF-7] | Escuchar en todas las interfaces o permitir borrados por GET: ampliaría el acceso y permitiría cambios involuntarios. |

**Dependencias:** Flask para web y plantillas; `pytest` solo para desarrollo y pruebas. `sqlite3`, `decimal`, `datetime`, `secrets` y `unicodedata` vienen con Python. No hay base de datos como servicio, red externa ni servicio de cuentas. Flask y pytest se declararán en `pyproject.toml`; antes de implementar se fijarán versiones compatibles para reproducir la instalación. [Principio 1]

## 2. Estructura y responsabilidades

```text
gamelog/
├── pyproject.toml
├── gamelog/
│   ├── __init__.py        # crea y configura la aplicación Flask
│   ├── domain.py          # nombres, valores, récords y orden; funciones puras
│   ├── service.py         # casos de uso; coordina reloj, reglas y persistencia
│   ├── storage.py         # único acceso a SQLite, esquema y transacciones
│   ├── web.py             # rutas, formularios y respuestas HTTP
│   ├── clock.py           # fecha local; sustituible en pruebas
│   ├── __main__.py        # inicia el servidor solo en 127.0.0.1
│   ├── messages.py        # textos de éxito y error en español
│   ├── migrations/        # esquema inicial y cambios numerados posteriores
│   ├── templates/         # páginas HTML en español
│   └── static/style.css   # estilos locales
└── tests/
    ├── test_domain.py
    ├── test_storage.py
    ├── test_service.py
    └── test_web.py
```

| Parte | Responsabilidad y cobertura |
| --- | --- |
| `domain.py` | Normalizar y comparar nombres, validar entradas, convertir valores, elegir mejor resultado y ordenar el historial. No importa Flask, SQLite, archivos ni reloj. [RF-1.1, RF-1.4, RF-2.1, RF-2.5, RF-3.4 a RF-3.8, RF-4.1 a RF-4.5; principio 3] |
| `service.py` | Crear, consultar, editar y borrar juegos, tipos y resultados; comprobar que cada tipo pertenece al juego; leer la fecha una vez; traducir errores esperados a códigos sin imprimir mensajes. [RF-1 a RF-4] |
| `storage.py` | Leer y escribir la base, mantener claves únicas y foráneas, confirmar o deshacer operaciones completas. Ninguna otra parte ejecuta SQL. [RF-1.6, RF-2.7, RF-3.9, RNF-2, RNF-7; principio 5] |
| `web.py` y plantillas | Mostrar juegos, tipos, resultados, formularios, confirmaciones y errores; procesar POST y protegerlos con CSRF. No calculan récords ni abren la base. [RF-1 a RF-4, RNF-1, RNF-5, RNF-6] |
| `clock.py` | Proveer la fecha local al servicio y una fecha fija en pruebas. [RF-3.2, CL-19] |
| `messages.py` | Centralizar mensajes al jugador en español; código e identificadores en inglés. [RNF-1; principio 6] |

El servicio recibe objetos `GameRepository`, `RecordTypeRepository`, `ResultRepository`, `UnitOfWork` y `Clock` por su constructor. Sus contratos se definen sin importar `sqlite3`, para que las reglas se prueben con memoria o con una base temporal. Los nombres exactos de clases pueden ajustarse al implementar sin cambiar las responsabilidades ni los RF.

Desde `gamelog/` se instalarán las dependencias declaradas y se iniciará la app con `python -m gamelog`; el navegador abrirá la dirección local que muestre el programa. El modo de depuración queda desactivado al usar datos personales reales. [RNF-5, RNF-6]

## 3. Datos y persistencia

**Ubicación:** un solo archivo en `~/.gamelog/gamelog.sqlite3`, obtenido desde la carpeta personal del sistema, sin depender de la carpeta desde la que se inicia la aplicación. La base de pruebas vive en una carpeta temporal. El archivo de datos no se añade a Git. [RNF-2, RNF-7]

**Esquema inicial, versión 1:**

| Tabla | Campos principales | Restricciones |
| --- | --- | --- |
| `games` | `id`, `name`, `name_key` | `id` identifica un juego; `name_key` es único. [RF-1] |
| `record_types` | `id`, `game_id`, `name`, `name_key`, `kind`, `direction` | `game_id` referencia `games` con borrado en cascada; `UNIQUE(game_id, name_key)`; `kind` es `score`, `time` o `custom`; `direction` es `high` o `low`. [RF-2] |
| `results` | `id`, `type_id`, `value_units`, `recorded_on` | `type_id` referencia `record_types` con borrado en cascada; `value_units >= 0`; `recorded_on` es fecha ISO `AAAA-MM-DD`; `id` conserva el orden original de registro. [RF-3, RF-4] |

Cada juego nuevo recibe dentro de **la misma transacción** los tipos `Puntuación` (`score/high`) y `Tiempo` (`time/low`). `Score` es un alias reservado: no crea una fila adicional y su nombre clave no se admite para un tipo propio. Los tipos `score` y `time` no se editan ni borran individualmente; al borrar el juego, sus resultados desaparecen junto con todos sus tipos. [RF-2.1, RF-2.5, RF-2.8, RF-1.6]

`name_key` se obtiene normalizando Unicode a NFC, quitando espacios Unicode extremos, aplicando `casefold()` y normalizando otra vez a NFC. El nombre visible conserva sus mayúsculas. La capa de dominio rechaza vacío, caracteres de control (`Cc`) y más de 80 caracteres Unicode. La base impide duplicados incluso si dos formularios llegan a la vez. [RF-1.1, RF-1.4, RF-2.2, RF-2.5, CL-9, CL-16]

`value_units` significa puntos para `score`, centésimas de segundo para `time` y centésimas del valor para `custom`. Por ejemplo:

| Juego | Tipo | Entrada | `value_units` | Fecha |
| --- | --- | --- | ---: | --- |
| Tetris | Puntuación | `12500` | `12500` | `2026-10-23` |
| Tetris | Tiempo | `03:42.58` | `22258` | `2026-10-23` |
| Tetris | Precisión (`high`) | `98.75` | `9875` | `2026-10-23` |

Los tres son **resultados separados**. Un valor `custom` como `7` se guarda en `700` unidades y se muestra como `7.00`; la comparación sigue siendo exacta. `results.id` nunca se modifica al editar el valor, de modo que los empates y el orden dentro de una fecha son estables. [RF-3.3, RF-3.8, RF-4.1, RF-4.5, CL-18]

En cada conexión se activa la comprobación de claves foráneas. Cada escritura valida primero, inicia una transacción, guarda todos los cambios y confirma; cualquier error deshace la transacción y se muestra sin alterar los datos previos. El esquema tiene una versión registrada; los cambios futuros se harán con migraciones numeradas, después de actualizar la spec. [RNF-7; principio 5]

## 4. Algoritmos en pseudocódigo

### 4.1 Valores de resultado [RF-3.4 a RF-3.8]

```text
parse_score(text):
    exigir 1..12 caracteres ASCII '0'..'9'
    devolver entero(text)

parse_time(text):
    exigir 2..6 dígitos de minutos + ':' + 2 dígitos de segundos + '.' + 2 de centésimas
    exigir segundos <= 59 y total > 0
    devolver (minutos * 60 + segundos) * 100 + centésimas

parse_custom(text):
    exigir 1..12 dígitos enteros, opcionalmente '.' y 1..2 dígitos decimales
    completar decimales a dos cifras y devolver unidades enteras
```

El mismo analizador valida altas y ediciones. Si falla, devuelve un error comprensible y no se inicia ninguna escritura. Al mostrar un tiempo, se reconstruyen minutos, segundos y centésimas con el formato aceptado. [RF-3.7, RF-3.8, CL-7, CL-8, CL-20]

### 4.2 Mejor resultado e historial [RF-2.6, RF-3.9, RF-4.1 a RF-4.5]

```text
results = leer todos los resultados de un tipo
si results está vacío: mostrar "Aún no hay récord"
si direction == high: best = máximo(result.value_units)
si direction == low:  best = mínimo(result.value_units)

orden "mejor": ordenar por valor descendente si high, ascendente si low;
               en empate, id descendente
orden "antiguo": ordenar por recorded_on ascendente, id ascendente
orden "nuevo":  ordenar por recorded_on descendente, id descendente
```

El dominio calcula el récord y los órdenes con los datos cargados, sin E/S. Después de editar o borrar un resultado, o cambiar la dirección de un tipo propio, la siguiente consulta vuelve a calcularlo; no hay un récord guardado que pueda quedar obsoleto. [RF-2.6, RF-3.8, RF-3.9, CL-14, CL-15, CL-21]

### 4.3 Operaciones con fecha y borrado [RF-1.6, RF-2.7, RF-3.2, RNF-7]

```text
registrar_resultado(juego, tipo, valor):
    comprobar juego, pertenencia del tipo y formato del valor
    hoy = clock.local_date()  # una sola lectura al confirmar
    en una transacción: insertar resultado con hoy

borrar_juego(juego, confirmación):
    si se cancela: no escribir
    en una transacción: borrar juego; sus tipos y resultados se borran en cascada

borrar_tipo_propio(tipo, confirmación):
    si se cancela: no escribir
    rechazar si es predefinido
    en una transacción: borrar tipo; sus resultados se borran en cascada
```

## 5. Contrato de la página web

Cada página usa texto y formularios en español. Los juegos y resultados muestran un identificador inequívoco mediante sus enlaces de edición y borrado, aunque dos entradas tengan el mismo valor y fecha. Los errores de validación conservan el formulario y los datos ya guardados. Todos los POST llevan token CSRF. [RNF-1, RNF-5, RF-3.11]

| Ruta / acción | Entrada y resultado visible | Éxito / error principal | Cobertura |
| --- | --- | --- | --- |
| `GET /` | Lista de juegos o aviso «Aún no hay juegos»; enlace para añadir. | `200` | RF-1.2 |
| `GET/POST /games/new` | Nombre; crea juego y tipos predefinidos; abre su ficha. | `200` formulario, `303` éxito, `400/409` nombre inválido/duplicado | RF-1.1, RF-1.4, RF-2.1 |
| `GET /games/<id>` | Ficha con tipos y mejor valor de cada uno, o estado sin récord. | `200`, `404` si no existe | RF-1.9, RF-2.3, RF-4.2 a RF-4.4 |
| `GET/POST /games/<id>/edit` | Cambia solo nombre y conserva tipos/resultados. | `303` éxito, `400/409/404` error | RF-1.5, RF-1.8 |
| `GET/POST /games/<id>/delete` | Página de confirmación; el POST confirmado borra todo lo asociado. Volver atrás cancela. | `303` éxito, `404` si no existe | RF-1.6 a RF-1.8 |
| `GET/POST /games/<id>/types/new` | Nombre y dirección `high/low` de tipo propio. | `303` éxito, `400/409/404` error | RF-2.2, RF-2.5 |
| `GET/POST /games/<id>/types/<id>/edit` | Edita solo tipo propio y recalcula orden/récord. | `303` éxito, `400/409/403/404` error | RF-2.6, RF-2.8, RF-2.10 |
| `GET/POST /games/<id>/types/<id>/delete` | Confirma borrado de tipo propio y sus resultados. | `303` éxito, `403/404` error | RF-2.7 a RF-2.10 |
| `GET /games/<id>/types/<id>?sort=best|oldest|newest` | Historial completo, mejor resultado y selector de orden. | `200`, `400` orden inválido, `404` tipo inexistente | RF-2.11, RF-4.1 a RF-4.5 |
| `GET/POST /games/<id>/types/<id>/results/new` | Un valor, fecha asignada automáticamente; vuelve al historial. | `303` éxito, `400/404` error | RF-1.3, RF-2.4, RF-3.1 a RF-3.7 |
| `GET/POST /results/<id>/edit` | Cambia únicamente el valor; mantiene juego, tipo, fecha y posición. | `303` éxito, `400/404` error | RF-3.8, RF-3.10, RF-3.11 |
| `POST /results/<id>/delete` | Borra solo el resultado y vuelve al historial. | `303` éxito, `404` error | RF-3.9 a RF-3.11 |

En esta tabla `200` muestra una página, `303` redirige tras un cambio, `400` indica entrada inválida, `403` acción prohibida sobre un tipo predefinido, `404` elemento inexistente y `409` nombre duplicado. Un fallo de guardado muestra un error en español y devuelve `500`; el estado previo se conserva. Un POST sin token válido devuelve `403` y no cambia datos. Los códigos HTTP son decisiones de interfaz; el jugador ve textos claros en español. [RF-1 a RF-4, RNF-1, RNF-7]

La confirmación de borrado de juegos y tipos tiene una pantalla propia; los controles «Cancelar» vuelven sin enviar un POST de borrado. Se usa un token aleatorio guardado en la sesión firmada de Flask y un campo oculto en cada formulario, comparados en el servidor. La clave de sesión se genera al iniciar la app local y no se guarda en el repositorio. [RF-1.7, RF-2.9, RNF-6]

## 6. Estrategia de pruebas

1. **Dominio puro (`test_domain.py`):** nombres Unicode, vacíos, duplicados y alias `Score`; límites de números y tiempo; exactitud de centésimas; récord alto/bajo, empates, órdenes y recálculo. Fechas y datos fijos. [RF-1.1, RF-1.4, RF-2.1, RF-2.5, RF-3.4 a RF-3.8, RF-4; CL-2, CL-5, CL-7 a CL-9, CL-13 a CL-16, CL-18, CL-20, CL-21]
2. **Persistencia temporal (`test_storage.py`):** crear/reabrir la base, claves únicas, claves foráneas, cascadas, conservación del orden de registro y deshacer una transacción con fallo inducido. La carpeta personal real nunca se usa en tests. [RF-1.6, RF-2.7, RF-3.9, RNF-2, RNF-7; CL-11, CL-12, CL-17]
3. **Casos de uso (`test_service.py`):** una fecha leída por alta, pertenencia de tipos, prohibición de editar predefinidos, entradas inexistentes, edición sin cambiar fecha/id, cancelaciones y errores sin cambios. [RF-1 a RF-3; CL-3, CL-6, CL-10 a CL-12, CL-19]
4. **Web (`test_web.py`):** cliente de pruebas de Flask para cada formulario y ruta; mensajes en español, estados HTTP, redirecciones, CSRF, orden por los tres modos y enlaces que identifican cada entrada. [Todos los RF, RNF-1, RNF-5, RNF-6; CL-1 a CL-21]
5. **Comprobación manual local:** abrir la app en el navegador, crear dos juegos, anotar puntuación y tiempo, editar/borrar un resultado, cambiar el orden y comprobar que otro equipo no puede acceder al servidor local. Documentar el resultado antes de cerrar el MVP. [RNF-5, RNF-6]

Cada test llevará el identificador del RF o CL que cubre en su nombre o marcador. `pytest -q` debe pasar antes de cada commit de código. La fecha y la ruta de la base se inyectan en los tests; ningún test escribe en los datos reales del jugador. [Principio 4, RNF-3]

## 7. Trazabilidad resumida

| Requisitos | Plan y prueba principal |
| --- | --- |
| RF-1.1 a RF-1.9 | Nombres en §2/§3; juegos y cascada en §3/§4.3; rutas en §5; dominio, persistencia y web en §6. |
| RF-2.1 a RF-2.11 | Tipos predefinidos/propios y alias en §3; dirección en §4.2; rutas y prohibiciones en §5; dominio, servicio y web en §6. |
| RF-3.1 a RF-3.11 | Valores y fecha en §3/§4.1/§4.3; altas/ediciones/borrados en §5; los cuatro niveles de pruebas en §6. |
| RF-4.1 a RF-4.5 | Cálculo y órdenes en §4.2; ficha/historial en §5; dominio y web en §6. |
| RNF-1 a RNF-7 | Idioma y web en §2/§5; datos e integridad en §3; pruebas en §6; stack y uso local en §1. |

## 8. Comprobación contra la constitución

| Principio | Evidencia en el plan |
| --- | --- |
| 1. Stack simple | Un lenguaje, una aplicación, Flask como única dependencia de ejecución justificada (§1). |
| 2. Spec primero | Todos los comportamientos remiten a la spec 001 v2 aprobada; cambios futuros de comportamiento o formato exigirán revisión previa (§1, §3). |
| 3. Reglas separadas | `domain.py` no hace E/S ni lee el reloj; servicio y web reciben sus resultados (§2, §4). |
| 4. Pruebas obligatorias | Cada RF/CL aparece en la estrategia y en la matriz; suite completa antes de commits de código (§6, §7). |
| 5. Persistencia controlada | Solo `storage.py` usa SQLite; una base local y migraciones versionadas (§2, §3). |
| 6. Idioma | Identificadores y archivos en inglés; mensajes, páginas y documentación en español (§2, §5). |

## 9. Fuentes técnicas consultadas

- [Flask 3.1: instalación y documentación](https://flask.palletsprojects.com/en/stable/installation/).
- [Flask: uso de SQLite](https://flask.palletsprojects.com/en/stable/patterns/sqlite3/).
- [Flask: pruebas con pytest y cliente de pruebas](https://flask.palletsprojects.com/en/stable/testing/).
- [Flask: consideraciones de seguridad y CSRF](https://flask.palletsprojects.com/en/stable/web-security/).
- [Python: `sqlite3` y transacciones](https://docs.python.org/3/library/sqlite3.html).
- [SQLite: claves foráneas y borrados en cascada](https://www.sqlite.org/foreignkeys.html).
