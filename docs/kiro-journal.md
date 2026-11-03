# Bitácora de uso de Kiro

**Para qué sirve este fichero.** El write-up de cómo usamos Kiro vale el 20% de
la nota (Technological Implementation) y es obligatorio. Escribirlo de memoria
la última semana sale mal; anotarlo cada día sale gratis.

**Cómo usarlo.** Una entrada por sesión de trabajo, al terminar, en dos minutos.
No hace falta que esté bien escrito: es materia prima, no prosa final. Lo que
importa son los detalles concretos (qué feature, qué pasó, qué número) porque
eso es lo que no se recuerda en diciembre.

**Reglas:**
- Anotad también lo que **no** funcionó. Los jueces preguntan si hubo
  experimentación, y un "probamos X y lo descartamos porque Y" puntúa.
- Poned rutas de ficheros reales (`.kiro/specs/...`), para poder enlazarlas desde
  el write-up.
- Si una sesión no usó nada interesante de Kiro, no hace falta entrada.

---

## Plantilla de entrada

```markdown
### YYYY-MM-DD — [iniciales]

- **Qué hice:**
- **Features de Kiro usadas:** (IDE / CLI / Web / Crew / specs / steering /
  hooks / custom agents / MCP / PBT / powers)
- **Qué funcionó bien:**
- **Qué no funcionó o tuve que corregir a mano:**
- **Ficheros relevantes:**
- **Cita para el write-up:** (una frase concreta y reutilizable)
```

---

## Entradas

### 2026-11-03 — [M1]

- **Qué hice:** monté la base del repo conforme a las reglas del hackathon
  (licencia MIT, `.kiro/` tracked, README) y los documentos de planificación:
  reglas destiladas, calendario en hora de Seúl, plantilla de propuesta,
  checklist de entrega y esta bitácora.
- **Features de Kiro usadas:** Kiro IDE en modo agente, con búsqueda web para
  leer las reglas oficiales de Devpost y convertirlas en un brief interno.
- **Qué funcionó bien:** delegar la lectura del reglamento completo (47 KB) y
  obtener de vuelta solo los requisitos que descalifican, ya convertidos a
  checklist. La conversión de husos horarios (PT → KST, con el cambio de horario
  del 1 de noviembre) salió correcta a la primera.
- **Qué no funcionó:** pendiente.
- **Ficheros relevantes:** `docs/hackathon-brief.md`, `docs/calendar.md`,
  `docs/project-proposal.md`, `docs/submission-checklist.md`
- **Cita para el write-up:** antes de escribir una línea de producto, usamos
  Kiro para convertir el reglamento del hackathon en una checklist ejecutable y
  un calendario en nuestra zona horaria, de modo que los requisitos que
  descalifican estuvieran cerrados en el primer commit.

---

## Temas que el write-up tiene que cubrir

Checklist de lo que el reglamento sugiere tratar. Marcad cuando tengáis material
suficiente anotado arriba.

- [ ] **Superficies:** IDE, CLI, Web, Crew. Si probamos varias, en qué se
      diferenciaron
- [ ] **Custom agents:** si creamos o usamos alguno, en qué app, y qué impacto
      tuvo
- [ ] **Agent hooks:** qué workflows concretos automatizamos y cómo mejoró el
      proceso
- [ ] **Spec-driven development:** cómo estructuramos la spec, cómo mejoró el
      proceso, y en qué se diferenció del vibe coding
- [ ] **Steering docs:** cómo mejoraron las respuestas y qué estrategia marcó la
      diferencia
- [ ] **MCP:** qué desbloqueó, y qué habría sido difícil o imposible sin ello
- [ ] **Property-based testing:** si detectó algo que los tests unitarios
      dejaban pasar, y para qué requisitos lo activamos
- [ ] **Kiro powers:** cuáles usamos, y si trajeron herramientas de terceros que
      no habríamos usado
