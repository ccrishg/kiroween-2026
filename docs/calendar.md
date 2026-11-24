# Kiroween 2026 — calendario en hora de Seúl

Todas las horas en **KST (UTC+9)**. Seúl no tiene horario de verano; California
sí, y lo terminó el 1 de noviembre de 2026. La diferencia con la hora del
Pacífico es **+17h** para todo lo que queda.

> **CORRECCIÓN 24/11/2026.** Una versión anterior de este documento daba por
> hecho que estábamos a 3 de noviembre. Era falso. El plan semanal original
> (semanas 0 a 5 desde el 31 de octubre) ya no aplica y queda sustituido por el
> plan de abajo.

---

## Fechas oficiales que quedan

| Hito | Hora Pacífico | **Hora Seúl (KST)** | Día |
|---|---|---|---|
| **Límite para pedir los 4.000 créditos** | 1 dic, 12:00 PST | **2 dic, 05:00** | miércoles |
| Límite para canjear los códigos de crédito | 4 dic | antes del 5 dic, 07:00 | sábado |
| **DEADLINE DE ENTREGA** | 4 dic, 14:00 PST | **5 dic, 07:00** | **sábado por la mañana** |
| Periodo de juicio | 16 dic – 29 ene | 17 dic 06:00 – 30 ene 06:00 | — |
| Ganadores anunciados | 3 feb 2027, 09:00 PST | 4 feb 2027, 02:00 | jueves |

**No hay extensiones. Las entregas tardías no se aceptan.**

- **Deadline interno duro:** viernes 4 de diciembre por la noche.
- **Entrega real objetivo:** jueves 3 de diciembre.

---

## Estado actual

> **Hoy: martes 24 de noviembre de 2026, 14:21 KST.**
>
> **Quedan 10 días** hasta el deadline interno del viernes 4 de diciembre.
>
> Descontando los dos últimos días, que son para vídeo, write-up y entrega,
> quedan **unos 8 días de desarrollo** para dos personas: alrededor de 16
> jornadas-persona.
>
> **Nada está implementado todavía.** Lo que existe es el repositorio, la
> documentación y la spec completa de KUR (requisitos, diseño y plan de tareas).

### Urgente hoy

- [ ] **Pedir los 4.000 créditos los dos.** El límite es el miércoles 2 de
      diciembre a las 05:00 KST, pero la revisión tarda **hasta 48 horas
      hábiles** y además hay **verificación de identidad** (selfie y documento).
      Si se pide el 1 de diciembre, no llega. Hay que hacerlo hoy.
- [ ] Registrarse los dos en el hackathon y crear el equipo en Devpost
- [ ] Abrir el borrador de la entrega en Devpost
- [ ] Rellenar los datos de estudiante de la sección 0 de
      `docs/project-proposal.md` (son $1.500 y 5 minutos)

---

## Plan de los 10 días que quedan

| Día | Objetivo | Salida verificable |
|---|---|---|
| **mar 24 nov** (hoy) | Administrativo. Cadena de herramientas, contrato de eventos `events.ts` congelado, esqueleto desplegado | URL pública con un "hello world" |
| **mié 25 – vie 27 nov** | Dos carriles en paralelo: M1 el motor de reglas y los dados, M2 el tablero en Canvas y el motor de audio | Motor que juega solo una partida en test; tablero que dibuja y suena contra eventos falsos |
| **sáb 28 – dom 29 nov** | Unir los dos carriles | **Partida jugable de punta a punta en el navegador** |
| **lun 30 nov** | Visualización de probabilidad, panel de procedencia, pase de dirección de arte | Producto que se ve terminado |
| **mar 1 dic** | Property tests (4 o 5, no 15) y desbloqueo de audio. **Congelación de features al final del día** | Suite verde, audio que arranca en el navegador de un juez |
| **mié 2 dic** | Grabar y editar el vídeo de 3 minutos. Escribir el write-up de Kiro | Vídeo subido a YouTube como público |
| **jue 3 dic** | **Entregar en Devpost** | Entrega enviada |
| **vie 4 dic** | Colchón. Solo arreglos | — |

---

## Hitos de control

| Fecha | Control | Si no se cumple |
|---|---|---|
| **hoy, mar 24** | Créditos pedidos y contrato de eventos cerrado | Sin créditos no hay herramienta; sin contrato los dos se bloquean mutuamente |
| **vie 27 nov** | Los dos carriles funcionan por separado | Recortar el audio a unos pocos motivos fijos en lugar de generativo |
| **dom 29 nov** | **Partida jugable de punta a punta** | **Recorte de emergencia:** quitar el oponente inteligente y dejarlo aleatorio, y quitar la reactividad del audio |
| **mar 1 dic** | Feature freeze | Lo que no esté, no entra. El vídeo no se recorta |
| **jue 3 dic** | Entrega enviada | Queda el viernes 4 y nada más |

**La regla que manda sobre todas las demás:** el vídeo y el write-up valen el
40% de la nota y se hacen el 2 de diciembre pase lo que pase. Si el 29 de
noviembre no hay partida jugable, se recorta producto, nunca presentación.
