# Kiroween 2026 — Project Proposal

**Documento a rellenar entre los dos.** Cada campo lleva marcado quién lo
contesta, para que podáis escribir en paralelo sin pisaros:

- `[M1]` lo contesta el miembro 1 (quien tiene este repo en local)
- `[M2]` lo contesta el miembro 2
- `[ambos]` hay que decidirlo juntos

Lo que no sepáis, dejadlo con `?`. Una propuesta con interrogantes sirve; una
propuesta inventada no.

> **Estado:** idea y categoría cerradas (**KUR**, categoría Resurrection).
> Pendiente: datos de equipo, decisión de público objetivo, stack, alcance del
> MVP y plan de Kiro.
> **Fecha límite para cerrar lo que falta:** domingo 8 de noviembre de 2026

---

## 0. Equipo

**Representative** (quien entrega la submission en Devpost, debe ser mayor de
edad donde reside) `[ambos]`:

| Campo | Miembro 1 `[M1]` | Miembro 2 `[M2]` |
|---|---|---|
| Nombre completo | | |
| Usuario de GitHub | `ccrishg` | |
| Perfil de LinkedIn | | |
| Email de estudiante | | |
| Año y grado | | |
| Universidad | | |
| Kiro user ID (para los créditos) | | |
| Zona horaria | KST (Seúl) | |
| Horas/semana realistas | | |

> El email de git de este repo es `1709896@uab.cat`, lo que apunta a la
> Universitat Autònoma de Barcelona. Confirmad si la universidad que declaráis
> para el premio de estudiante es la de origen o la de intercambio en Corea, y
> usad el email institucional que podáis demostrar con matrícula.
>
> Para pedir los créditos hay **verificación de identidad** (selfie y documento
> oficial). El nombre del perfil de Devpost debe coincidir **exactamente** con
> el del documento: es la causa más común de que el check falle.

Preguntas de elegibilidad `[ambos]`:

- ¿Somos **todos** estudiantes en una institución académica reconocida?
  (requisito duro para Best Student Project) → `sí / no`
- ¿Podemos aportar prueba de matrícula si nos la piden? → `sí / no`
- ¿Alguno se registró en **Kiroween 2025**? (abre el bonus Ghost of Kiroween
  Past, $2.000) → `sí / no / quién`
- ¿Alguno es fundador de una startup con web y LinkedIn? (abre Best Startup
  Project, $10.000) → `sí / no`
- ¿Seremos 2 o 3 personas? (el máximo es 3) → `2 / 3`

---

## 1. Categoría

**Categoría elegida: `Resurrection`** ✅

**Por qué esta y no otra.** El proyecto no reinterpreta tecnología muerta por
analogía: resucita dos artefactos concretos cuyas instrucciones de uso estuvieron
ilegibles durante milenios y hoy están descifradas. Eso encaja literalmente con
la definición oficial de la categoría ("devolver a la vida tu tecnología muerta
favorita"), y da una narrativa de vídeo que no necesita adornos. El premio de
estudiante también se otorga por categoría, así que competimos dos veces con el
mismo trabajo.

**Lo que descartamos y por qué:**

- *Frankenstein*: la unión de juego, música y datos aquí no es un injerto
  forzado, es histórica. Presentarla como quimera restaría credibilidad.
- *Skeleton Crew*: exige dos aplicaciones distintas en 31 días con dos personas.
- *Costume Contest*: nos quedaríamos solo con la capa visual y perderíamos la
  parte de ingeniería musical y datos, que es donde está nuestro interés.

**Bonus prizes a los que nos comprometemos** `[ambos]`:

- [x] **Best Student Project: Resurrection** — gratis si los dos somos
      estudiantes, solo hay que rellenar datos. $1.500 + Kiro Pro Max 1 año
- [ ] **Bonus Blog Prize** — ~1h de trabajo, 100 plazas. Quién lo escribe:
- [ ] **Social Blitz** — ~30min. El hashtag `#KiroweenHackathon26` va **al
      principio** del post. Quién lo publica:
- [ ] **Best Kiro Crew App** — trabajo de ingeniería real, y solo se evalúa
      entre el top 50. Decidir en la semana 3, no antes
- [ ] **Best Ghost of Kiroween Past** — solo si alguien se registró en 2025
- [ ] **Best Startup Project** — no aplica
- [x] **Best Demo** — sin pasos extra, sale solo si el vídeo es bueno

---

## 2. La idea

**Nombre del proyecto: `KUR`**

`KUR` es el nombre sumerio del inframundo. Cuatro letras, se pronuncia igual en
cualquier idioma, y nombra exactamente la premisa: juegas contra los muertos.

**Pitch en una frase:**

> El juego de mesa más antiguo del mundo estuvo cuatro mil años sin reglas
> legibles. Las descifraron. Ahora juégalo contra los muertos, con la música más
> antigua que se conserva y con la probabilidad a la vista.

**Problema que resuelve.** Hay un montón de conocimiento arqueológico
descifrado que no se puede *experimentar*. Las reglas del Juego Real de Ur
existen desde 2007 y el Himno Hurrita se transcribió en los años 60, pero para
una persona normal siguen siendo un PDF académico y un vídeo de YouTube. KUR
convierte dos reconstrucciones académicas en algo que se juega y se oye.

**Qué hace que no se pueda hacer hoy.** Hay implementaciones del Juego de Ur
jugables en web, y hay grabaciones del Himno Hurrita. Lo que no existe es la
unión: una partida donde el estado del juego genere la música en el sistema de
afinación antiguo, y donde la probabilidad exacta de cada tirada sea parte de la
interfaz en vez de estar escondida. La aportación nuestra son esas dos capas
sobre el juego, no el juego.

### Decisión abierta: público objetivo `[ambos]`

Hay que cerrarla antes del domingo 8, porque cambia el diseño de la interfaz.

| Opción | Público | Qué implica | Riesgo |
|---|---|---|---|
| **A. Solo jugadores** | gente a la que le gustan los juegos de mesa y la historia | interfaz centrada en la partida, la viz como adorno informativo | **Potential Impact es el 20% de la nota** y "un juego antiguo bonito" no demuestra que alguien lo usaría |
| **B. Educativo** (recomendada) | profesores de secundaria y estudiantes de probabilidad | la visualización en vivo de la binomial es la herramienta: se juega para desarrollar intuición de probabilidad. Añade un modo "predice tu tirada" y un resumen de partida | más trabajo de interfaz, y hay que ser honestos en que no lo hemos validado con profesores |

**Argumento para B:** los jueces preguntan literalmente si alguien usaría esto
de verdad. Con B hay una respuesta concreta y un usuario con nombre. Con A la
respuesta es "a quien le guste". Y B no obliga a renunciar a nada: el juego
sigue siendo el mismo, solo cambia qué se enseña en pantalla.

**Decisión:** `A / B` → _pendiente_

**Lo que NO vamos a hacer.** Escribid aquí con generosidad; esto es lo que salva
el proyecto en la semana 3 `[ambos]`:

- Sin multijugador online ni cuentas de usuario
- Sin backend con estado persistente si se puede evitar
- Sin aplicación móvil nativa
- Sin reconstruir el resto de los himnos hurritas: solo el h.6
- Sin modelo 3D del tablero: 2D bien hecho antes que 3D regular
- _(añadid más a medida que aparezcan tentaciones)_

---

## 3. Encaje con el tema

### La tecnología muerta nº 1: el Juego Real de Ur

Juego de mesa sumerio de hace unos 4.500 años, excavado por Leonard Woolley en
las tumbas reales de Ur, en el actual Irak, entre 1926 y 1927. Sus reglas
estuvieron perdidas hasta que Irving Finkel, conservador del British Museum,
las reconstruyó a partir de una tablilla cuneiforme babilónica del 177 a.C.
(BM 33333B) escrita por el escriba Itti-Marduk-balāṭu. El conjunto de reglas
resultante se conoce como *Finkel ruleset* y es el que se juega hoy.

Mecánica: dos jugadores, siete piezas cada uno, una carrera por un recorrido
compartido, casillas con rosetas que funcionan como refugio seguro.

**El gancho de Halloween no hay que inventarlo.** El propio British Museum
señala que las casillas centrales del tablero se usaban además para
**adivinación**. El juego ya era un oráculo. De ahí sale toda la ambientación:
cada tirada es un augurio, y el oponente son los muertos.

### La tecnología muerta nº 2: el Himno Hurrita nº 6

Datado hacia el 1400 a.C., hallado en tablillas cuneiformes en el Palacio Real
de Ugarit (Ras Shamra, Siria) en excavaciones de los años 50. Es la obra de
música notada **sustancialmente completa más antigua que se conserva**, y
Guinness lo reconoce como la canción escrita con notación más antigua.

Lo relevante para nosotros: **su notación describe intervalos, no notas**. No es
una melodía que se copia y se pega, es un sistema que hay que implementar. Eso
convierte la capa musical en ingeniería de verdad (ratios de afinación justa,
síntesis de timbres de lira) y no en reproducir un MP3.

### La tercera capa: los datos

Sale del propio juego, no la añadimos nosotros. Se tira con **cuatro dados
tetraédricos con dos vértices marcados cada uno**, así que el resultado va de 0
a 4 siguiendo una binomial B(4, ½):

| Tirada | 0 | 1 | 2 | 3 | 4 |
|---|---|---|---|---|---|
| Probabilidad | 1/16 | 4/16 | 6/16 | 4/16 | 1/16 |

Un espacio de probabilidad exacto, pequeño y verificable. Se puede visualizar en
vivo durante la partida sin inventar ninguna métrica, y es un caso de libro para
**property-based testing**: los invariantes del tablero y la distribución de los
dados son propiedades comprobables, no ejemplos.

### Cómo las tres capas son un solo sistema

El estado del juego alimenta el motor de audio y la visualización a la vez. La
música no es una pista de fondo: la tensión de la partida (piezas en riesgo,
distancia a la meta, probabilidad de la tirada que necesitas) determina qué
intervalos suenan. Esa es la frase del pitch: **los datos y la música son el
mismo sistema.**

### Aviso legal importante

Las tablillas son antiguas, pero **una transcripción o una grabación moderna del
himno puede estar protegida por copyright**, y el reglamento prohíbe música con
copyright en el vídeo de demo. Generamos el audio nosotros a partir de los datos
de intervalos. **No sampleamos grabaciones existentes.** Lo mismo con imágenes
del tablero: usar fotos propias o de dominio público verificado.

### Fuentes

Verificar cada afirmación antes de ponerla en el vídeo o en la descripción.
Contenido reformulado para cumplir con las restricciones de licencia.

- [Royal Game of Ur — Wikipedia](https://en.wikipedia.org/wiki/Royal_Game_of_Ur)
- [Top 10 historical board games — British Museum](https://www.britishmuseum.org/blog/top-10-historical-board-games)
  (origen del dato sobre el uso del tablero para adivinación)
- [On the Rules for the Royal Game of Ur](https://www.academia.edu/15173145/On_the_Rules_for_the_Royal_Game_of_Ur)
  (tablilla BM 33333B)
- [Rules of the Royal Game of Ur — royalur.net](https://royalur.net/rules)
  (Finkel ruleset)
- [Hurrian songs — Wikipedia](https://en.wikipedia.org/wiki/Hurrian_songs)
- [Oldest written song with notation — Guinness World Records](https://www.guinnessworldrecords.com/world-records/729727-oldest-written-song-with-notation)

---

## 4. Alcance de la demo

> **Bloqueado** hasta cerrar la decisión de público objetivo de la sección 2.
> El vídeo son 3 minutos y los jueces no están obligados a ver más, así que el
> alcance se define desde el vídeo hacia atrás.

**Recorrido de usuario end-to-end que se verá en el vídeo, paso a paso**
`[ambos]`:

1.
2.
3.
4.

**3 features imprescindibles** — si falta una, no hay proyecto `[ambos]`:

| # | Feature | Dueño |
|---|---|---|
| 1 | | |
| 2 | | |
| 3 | | |

**3 features deseables** — sacrificables el 22 de noviembre si vamos justos:

| # | Feature | Dueño |
|---|---|---|
| 1 | | |
| 2 | | |
| 3 | | |

**Criterio de "terminado" para el MVP** — una frase comprobable `[ambos]`:

---

## 5. Técnica

**Stack** `[ambos]`:

| Capa | Elección | Por qué |
|---|---|---|
| Frontend | | |
| Motor de audio | | |
| Visualización | | |
| Estado / datos | | |
| Despliegue | | |

**Experiencia de cada uno con ese stack** (para repartir trabajo con sentido):

- `[M1]` Cómodo con: / Nunca he usado:
- `[M2]` Cómodo con: / Nunca he usado:

**Dónde se despliega y con qué URL pública** — requisito de entrega, los jueces
necesitan probarlo `[ambos]`:

**Si necesita login, credenciales de prueba para los jueces** (van en el
formulario de Devpost, que no es público) `[ambos]`:

**Datos y fuentes de terceros, y su licencia de uso** `[ambos]`:

| Fuente | Para qué | Licencia / términos | ¿Verificada? |
|---|---|---|---|
| Finkel ruleset | reglas del juego | reconstrucción académica, comprobar atribución | |
| Datos de intervalos del h.6 | motor de audio | comprobar qué transcripción usamos y su estado | |
| | | | |

**Código preexistente que incorporamos** — el reglamento obliga a declarar
cualquier cosa que no sea un framework o starter template estándar `[ambos]`:

**Riesgo técnico mayor y plan B** `[ambos]`:

- Riesgo probable: que el motor de audio generativo suene mal. Plan B: reducir a
  un conjunto fijo de motivos pregenerados en la afinación correcta, disparados
  por eventos del juego.

**Coste de infraestructura** — debe ser gratis y sin restricciones para los
jueces hasta el 30 de enero de 2027 `[ambos]`:

**Disponibilidad regional** — ¿depende de features o modelos de Kiro que puedan
no estar disponibles desde Corea? `[ambos]`:

---

## 6. Estrategia de uso de Kiro

Esto es el 20% de la nota (Technological Implementation) y alimenta el write-up
obligatorio. Los jueces preguntan si hubo una estrategia pensada de desarrollo
con IA y si se experimentó.

**Superficies que vamos a usar y para qué** `[ambos]`:

| Superficie | ¿La usamos? | Para qué |
|---|---|---|
| Kiro IDE | | |
| Kiro CLI | | |
| Kiro Web | | |
| Kiro Crew | | |

**Specs previstas en `.kiro/specs/`** `[ambos]`:

**Steering docs previstos en `.kiro/steering/`** `[ambos]`:

**Hooks previstos y qué automatizan** `[ambos]`:

| Hook | Trigger | Qué automatiza |
|---|---|---|
| | | |

**Custom agents previstos** `[ambos]`:

**Servidores MCP que vamos a conectar y qué desbloquean** `[ambos]`:

**Property-based testing: qué invariantes del dominio cubrimos** `[ambos]`:

Candidatos obvios, que es una ventaja de esta idea:

- la suma de las cuatro casillas de probabilidad siempre da 1
- ninguna pieza puede ocupar una casilla ya ocupada por una pieza propia
- el total de piezas en juego, en mano y retiradas es siempre 7 por jugador
- una partida válida siempre termina
- toda frecuencia generada por el motor de audio cae dentro del rango audible

**Kiro powers que pensamos usar** `[ambos]`:

**¿Construimos una Kiro Crew app para el bonus?** → decidir en la semana 3

> Desde el primer día, cada uno anota en `docs/kiro-journal.md` qué feature de
> Kiro usó, qué funcionó y qué no. El write-up se escribe copiando de ahí.

---

## 7. Diseño y presentación

El 20% de la nota es Presentation y otro 20% es Design. Un proyecto
técnicamente bueno con un vídeo malo pierde contra lo contrario.

**Dirección de arte.** Punto de partida: barro, cuneiforme, lámpara de aceite.
Oscuro pero no genérico de Halloween; el terror es arqueológico, no de
calabazas. `[M2]`

**Referencias visuales** (3 enlaces o descripciones) `[ambos]`:

1.
2.
3.

**Paleta, tipografía y sonido** `[M2]`:

**Quién graba y edita el vídeo** `[ambos]`:

**Guion del vídeo en 3 actos, máximo 3 minutos** `[ambos]`:

| Tiempo | Qué se ve | Qué se dice |
|---|---|---|
| 0:00–0:20 | | el juego más antiguo del mundo, 4.000 años sin reglas legibles |
| 0:20–2:20 | | partida completa: tirada, augurio, música reaccionando, viz de probabilidad |
| 2:20–3:00 | | cómo usamos Kiro y por qué importa |

**Plataforma del blog post y quién lo escribe** — debe mencionar que es parte
del Kiroween 2026 hackathon `[ambos]`:

**Quién publica el post social** con `#KiroweenHackathon26` al inicio `[ambos]`:

---

## 8. Reparto de trabajo

**Miembro 1 — motor de juego y datos** `[M1]`

- Reglas del Juego de Ur según el Finkel ruleset, como motor puro y testeable
- Oponente de la máquina ("los muertos")
- Modelo de probabilidad de los dados y estadísticas de partida
- Property-based testing de los invariantes del tablero y de los dados
- Specs en `.kiro/specs/` y despliegue

**Miembro 2 — audio y visualización** `[M2]`

- Motor de audio: afinación e intervalos del h.6, síntesis, reacción al estado
  de la partida
- Visualización de la probabilidad en vivo y del historial de partidas
- Dirección de arte e interfaz
- Grabación y edición del vídeo

**Frontera entre los dos.** El motor de juego expone un único flujo de eventos
de estado (`tirada`, `movimiento`, `captura`, `roseta`, `pieza retirada`,
`fin de partida`) y **ni el audio ni la visualización leen el estado interno del
motor**. Esa es la única interfaz que hay que acordar, y conviene acordarla el
primer día: con eso los dos podéis trabajar en paralelo sin bloquearos.

**Cómo revisamos** `[ambos]`:

- Ramas: `feature/...`, `docs/...`, `fix/...`
- Commits: `acción: explicación` en minúscula (`add: new proposal`,
  `fix: broken deploy`, `docs: update calendar`)
- ¿PRs obligatorios o push directo a main?
- ¿Quién mergea?

**Cuándo sincronizamos** (día y hora fijos, en KST) `[ambos]`:
