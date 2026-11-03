# Kiroween 2026 — reglas destiladas

Documento de solo lectura. Resumen de <https://kiroween.devpost.com> y
<https://kiroween.devpost.com/rules>, consultadas el 3 de noviembre de 2026.
Si hay discrepancia, **manda el reglamento oficial**.

---

## 1. Lo que hay que construir

Una aplicación de software **funcionando**, construida con Kiro (IDE, CLI, Web
y/o Kiro Crew), presentada en **una** de las cuatro categorías. El reglamento
dice literalmente que un uso insuficiente de Kiro es motivo de descalificación a
discreción de los jueces.

### Las cuatro categorías

| Categoría | Definición oficial | Qué implica para nosotros |
|---|---|---|
| **Resurrection** | Devolver a la vida tu tecnología muerta favorita. Reimaginar tecnología obsoleta con innovación actual o resolviendo problemas de mañana. | Hay que elegir una tecnología muerta concreta y defender qué le aporta 2026. Narrativa de vídeo muy potente. |
| **Frankenstein** | Coser una quimera de tecnologías en una sola app. Unir elementos aparentemente incompatibles para construir algo inesperadamente potente. | Puntúa alto en Innovation. El riesgo es que la unión parezca gratuita. |
| **Skeleton Crew** | Construir una plantilla de código esqueleto, lo bastante escueta para ser clara y lo bastante flexible para soportar varios casos de uso. Demostrar su versatilidad con **dos aplicaciones distintas** construidas sobre esa base. | **El repo debe contener 2 carpetas separadas, una por aplicación.** Es la categoría con más carga de trabajo. |
| **Costume Contest** | Cualquier app, pero con una interfaz espeluznante, pulida e inolvidable. Elementos de diseño spooky que **mejoren la función** de la app. | Mejor ratio esfuerzo/premio si hay ojo para el diseño. Lo spooky no puede ser solo decoración. |

### Requisitos del proyecto

- **Solo proyectos nuevos.** Creados por nosotros después del inicio del
  Submission Period (30 de octubre de 2026, 09:00 PT). Los proyectos de
  Kiroween 2025 no son elegibles. Se pueden usar frameworks, librerías y
  starter templates estándar, pero **cualquier otro código preexistente debe
  declararse**.
- **Funcionalidad.** Debe instalarse y correr de forma consistente en la
  plataforma a la que va destinado, y comportarse como se muestra en el vídeo y
  se describe en el texto.
- **Integraciones de terceros.** Si se usa un SDK, API o dataset de terceros,
  hay que estar autorizado según sus términos y licencia.
- **Propiedad.** Trabajo original, de propiedad exclusiva nuestra, sin violar
  derechos de terceros.

---

## 2. Requisitos de entrega

Todos obligatorios. Los dos primeros son los que descalifican en seco.

- [ ] **Repo público en GitHub.** No GitLab, no Bitbucket. Con todo el código
      fuente, assets e instrucciones necesarias para que el proyecto funcione.
- [ ] **Licencia open source** en un fichero, detectable y **visible en la
      sección "About"** de la portada del repo.
- [ ] **Directorio `/.kiro` en la raíz del proyecto**, mostrando el uso de
      specs, hooks y steering. **Nunca** añadir `/.kiro` ni sus subcarpetas al
      `.gitignore`: el reglamento dice que eso puede descalificar la entrega.
- [ ] **URL de la aplicación funcionando.** Si necesita login, las credenciales
      e instrucciones de prueba van en el formulario de Devpost (ese campo no es
      público, solo lo ven los jueces).
- [ ] **Descripción de texto** que explique features y funcionalidad.
- [ ] **Vídeo demo**: menos de **3 minutos** (los jueces no están obligados a
      ver más allá), mostrando el proyecto funcionando en el dispositivo para el
      que se construyó, **subido a YouTube y público**, sin marcas registradas
      ni música ni material con copyright de terceros.
- [ ] **Write-up de cómo se usó Kiro.** Ver sección 4.
- [ ] **Categoría y bonus prizes** identificados en el formulario.
- [ ] Todo el material **en inglés** (o con traducción al inglés adjunta).

Extras por categoría o premio:

- **Skeleton Crew:** 2 carpetas de repo separadas para las 2 aplicaciones.
- **Best Student Project:** ver sección 5.

El proyecto debe estar accesible **gratis y sin restricciones** para que los
jueces lo prueben hasta que termine el Judging Period (29 de enero de 2027).

---

## 3. Criterios de puntuación

Cinco criterios, **20% cada uno, 10 puntos cada uno**. Antes hay una fase
pass/fail que solo comprueba que el proyecto encaja razonablemente con el tema y
usa razonablemente las herramientas exigidas.

| Criterio | Qué preguntan los jueces |
|---|---|
| **Potential Impact** | ¿Hay un caso creíble de que resuelve un problema real para una audiencia real? ¿Es útil, intuitivo y accesible? ¿Alguien lo usaría de verdad? |
| **Technological Implementation** | ¿Con cuánta profundidad y habilidad se usa Kiro? ¿Hubo una estrategia pensada de desarrollo con IA? ¿Se usó una variedad de herramientas de Kiro con impacto? ¿Se experimentó? |
| **Design** | ¿Entrega una experiencia de producto completa y coherente, no solo una prueba de concepto técnica? ¿El diseño es intuitivo, pulido e intencionado? |
| **Innovation/Idea** | ¿Es creativo y novedoso? ¿Se diferencia de lo que ya existe? ¿Se combinan tecnologías o recursos públicos (datasets) de forma única? |
| **Presentation** | ¿El vídeo demuestra el proyecto funcionando de punta a punta? ¿El pitch comunica qué problema resuelve, para quién y por qué importa? ¿Es fácil de seguir, creativo y memorable? |

**Lectura estratégica:** el 40% de la nota (Technological Implementation +
Presentation) no depende de cuánto código escribamos, sino de cómo documentemos
el uso de Kiro y de cómo grabemos el vídeo. Son las dos partes que nunca se
recortan.

**Desempate:** se reevalúa de cero con máximo de 20 puntos por criterio en vez
de 10, y se repite subiendo 10 puntos por ronda hasta que haya ganador.

---

## 4. El write-up de Kiro

Los jueces deben entender cuán efectivamente usamos Kiro. El reglamento sugiere
cubrir estos puntos, y conviene tratarlos como una checklist:

- **Superficies:** ¿IDE, CLI, Web, Crew? Si probamos varias, en qué se
  diferenciaron.
- **Custom agents:** ¿creamos o usamos agentes personalizados? ¿En qué app de
  Kiro? ¿Qué impacto tuvieron en el build?
- **Agent hooks:** ¿qué workflows concretos automatizamos? ¿Cómo mejoraron el
  proceso de desarrollo?
- **Spec-driven development:** ¿cómo estructuramos la spec para que Kiro la
  implementara? ¿Cómo mejoró el proceso? ¿Qué diferencia hubo frente al vibe
  coding?
- **Steering docs:** ¿cómo mejoraron las respuestas de Kiro? ¿Qué estrategia
  marcó la mayor diferencia?
- **MCP:** ¿qué desbloqueó extender las capacidades de Kiro? ¿Qué features o
  mejoras de workflow habrían sido difíciles o imposibles sin ello?
- **Property-based testing:** ¿detectó problemas que los tests unitarios habrían
  dejado pasar? ¿Para qué requisitos lo activamos?
- **Kiro powers:** ¿qué powers usamos? ¿Alguna herramienta o integración de
  terceros que no habríamos usado de otro modo?

> Por esto existe `docs/kiro-journal.md`. Escribir esto de memoria la última
> semana sale mal; anotarlo cada día sale gratis.

---

## 5. Premios

Un proyecto puede ganar **un premio Overall o uno de Categoría, no los dos**, y
además **cualquier número de bonus**.

### Overall y categoría

| Premio | Importe |
|---|---|
| 1st Place | $30.000 |
| 2nd Place | $20.000 |
| 3rd Place | $10.000 |
| Best Resurrection | $5.000 |
| Best Frankenstein | $5.000 |
| Best Skeleton Crew | $5.000 |
| Best Costume Contest | $5.000 |

### Bonus prizes y su requisito exacto

| Premio | Importe | Requisito |
|---|---|---|
| **Best Student Project: _(por categoría)_** | $1.500 + Kiro Pro Max 1 año, uno por cada una de las 4 categorías | **Todos** los miembros del equipo deben ser estudiantes de una institución académica reconocida. Hay que indicar la categoría y aportar, de cada miembro: **LinkedIn, email de estudiante, año y grado, y nombre de la universidad**. Pueden pedir prueba de matrícula; si no se entrega a tiempo, se pierde la consideración. |
| **Best Startup Project** | $10.000 | Confirmar que eres fundador de la startup, con LinkedIn y web de la startup. |
| **Best Ghost of Kiroween Past** | $2.000 | Confirmar en el formulario que te registraste en Kiroween 2025. Lo verifican contra sus registros. El proyecto de 2026 debe ser nuevo. |
| **Best Kiro Crew App** | $1.000 | URL pública del repo de GitHub de la app de Kiro Crew + descripción de features. **Solo se evalúa entre el top 50** de entregas elegibles. La app debe crearse durante el Submission Period. |
| **Best Demo** | $1.000 | Sin pasos extra. Se da a la entrega con mejor nota en Presentation que además esté **en el top 10%** de la puntuación global. |
| **Bonus Blog Prize** | 2.000 créditos Kiro, **100 ganadores** | Post de blog público sobre Kiro y nuestro proyecto, en cualquier plataforma de terceros, publicado **dentro del Submission Period**, que **mencione explícitamente que se escribe como parte del Kiroween 2026 hackathon**. Se da a las 100 entregas elegibles con mejor nota. |
| **Social Blitz Prize** | Kiro Pro+ 3 meses, **5 ganadores** | Post en cualquier red social que **empiece** con el hashtag `#KiroweenHackathon26` (el hashtag va al principio, antes del resto del contenido), contando lo que más nos gusta de Kiro y cómo cambió nuestra forma de desarrollar. Publicado dentro del Submission Period. Lo elige el equipo de Kiro por calidad del contenido (ideas originales frente a texto escrito por IA) y de los assets. |

### Nuestra prioridad

1. **Best Student Project** de nuestra categoría. Mismo trabajo que la entrega
   normal, solo hay que rellenar datos. Mejor relación esfuerzo/probabilidad.
2. **Bonus Blog Prize.** Una hora de trabajo, 100 plazas.
3. **Social Blitz.** Media hora. Ojo al hashtag al principio del post.
4. **Best Demo** y los premios de categoría salen solos si el vídeo y el
   producto están bien.

---

## 6. Logística

- **Equipo:** máximo **3 personas**. Hay que designar un **Representative** que
  entrega la submission en nombre del equipo.
- **Créditos:** 4.000 créditos Kiro gratis por persona, una sola vez. Se piden
  por el Credit Request Form con el Kiro user ID o AWS account ID correcto (un
  ID equivocado puede no aprobarse). **Tardan hasta 48 horas hábiles** en
  revisarse. Los créditos canjeados expiran el 31 de diciembre de 2026.
- **Borradores:** se pueden guardar borradores de la entrega en Devpost tantas
  veces como se quiera hasta que cierre el plazo. Una vez cerrado, no se puede
  modificar nada.
- **Múltiples entregas:** permitidas, pero cada una debe ser única y
  sustancialmente distinta.
- **Restricciones regionales:** la disponibilidad de features y modelos de IA de
  Kiro varía por región, y es responsabilidad nuestra comprobarlo. Si el
  proyecto depende de algo no disponible en nuestra región, el problema es
  nuestro.
- **Discord de Kiro:** canal `#kiroween-hackathon`, con office hours cada dos
  semanas.
- **No elegibles** (relevante por si cambia nuestra residencia): residentes de
  Quebec, Argentina, Australia, Brasil, Indonesia, Italia, Filipinas, Rusia,
  Emiratos Árabes Unidos y los países sancionados por la OFAC. Corea del Sur y
  España no están en la lista.
