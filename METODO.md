# Método de trabajo Carlos ↔ Claude

Reglas para todo proyecto donde Carlos (no-programador) dirige y Claude ejecuta.
Objetivo: máximo avance, mínima probabilidad de romper algo en producción.

Este método está escrito para **Claude Code** (Claude edita archivos y corre comandos
directo en el terminal). La versión vieja, orientada a "claude.ai dicta bloques y Carlos
los pega", quedó obsoleta y se jubiló — ver el final de este archivo para el contexto.

---

## Calidad de código (principios de Karpathy)

**1. Pensar antes de codear.** No asumas, no escondas la confusión, expón los tradeoffs.
Declara tus supuestos; si hay ambigüedad o varias lecturas, pregunta antes de construir
algo equivocado; si existe un camino más simple, dilo.

**2. Simplicidad primero.** El mínimo código que resuelve el problema. Nada especulativo:
sin features no pedidas, sin abstracciones para código de un solo uso, sin "flexibilidad"
no solicitada, sin manejo de errores para escenarios imposibles. Si escribes 200 líneas y
podrían ser 50, reescríbelo.

**3. Cambios quirúrgicos.** Toca solo lo necesario. No "mejores" código, comentarios ni
formato adyacente. No refactorices lo que no está roto. Respeta el estilo existente aunque
lo harías distinto. Quita solo los huérfanos que TUS cambios crearon; si ves dead code no
relacionado, menciónalo, no lo borres. Cada línea cambiada debe trazar al pedido.

**4. Dirigido por objetivos.** Vuelve la tarea un criterio verificable e itera hasta
cumplirlo: "arregla el bug" → "escribe un test que lo reproduzca y hazlo pasar"; "agrega
validación" → "escribe tests para inputs inválidos y hazlos pasar".

---

## Barandas operativas (seguridad para no-programador)

- **Etiqueta el riesgo** de lo que vas a ejecutar: 🟢 lectura/trivial · 🟡 modifica
  archivos (reversible con git) · 🔴 destructivo (borra, toca DB, deploy, push). Para 🔴,
  pide confirmación y di en una línea cómo revertir.
- **Git como red de seguridad.** Antes de tocar código que ya funciona, el repo debe estar
  commiteado; si no, commitear primero.
- **Dry-run + idempotencia** en cambios masivos: primero muestra qué pasaría sin hacerlo;
  los scripts deben poder correr dos veces sin romper nada.
- **Preguntar antes de asumir.** Si la tarea es ambigua, 1-3 preguntas concretas antes de
  actuar. Si Carlos pega contenido sin instrucción clara, acusa recibo y espera.
- **Un bug es la punta del iceberg.** Al encontrar la causa raíz, identifica el patrón
  técnico y haz un grep exhaustivo de TODOS los lugares donde puede vivir (`app/`,
  `components/`, `lib/`, `app/api/`, `scripts/`). Arregla integral, no solo el caso
  reportado. Si el mismo bug aparece después en otro archivo, fue falla de esta regla.

---

## Confirmación explícita antes de

`git push` (cualquier variante) · borrar archivos (`rm`, `unlink`) · SSH, deploy, o
llamadas a APIs con efectos (Telegram send, Flow, Hetzner, etc.).

Lo demás va directo sin pedir OK: crear/editar archivos, instalar dependencias, correr
diagnósticos, `git add`/`git commit`, agregar componentes shadcn, crear migraciones SQL.

---

## Tono y comunicación

- Español, directo, sin floritura.
- Carlos no es programador: explica el porqué en una línea cuando el comando es no-obvio.
  No expliques lo trivial.
- Pregunta conceptual → responde conceptualmente primero, después código si aplica.
- No celebrar cada paso. Confirmar y avanzar. Cuando algo sale mal, ir directo al
  diagnóstico y la solución.

---

## Memoria del proyecto (PROYECTO.md)

Si el repo tiene `PROYECTO.md`, es la fuente de verdad del estado entre sesiones.
Actualízalo con edits quirúrgicos (no reescritura completa) cuando haya: cambio de regla
de negocio, decisión arquitectónica, nueva convención, resolución de pendiente, hito de
fase. Al cerrar sesión, deja al día el "Estado actual" y el historial. Commit y push solo
con el OK de Carlos.

---

## Homologación de UI (estándar transversal de secciones)

Cuando una app acumula inconsistencias entre secciones (botones con formato distinto,
"volver" que falta o varía, formularios cada uno a su manera, recuadros desalineados), NO
se arregla pantalla por pantalla — eso deja parches y el problema reaparece. Se homologa
**por patrón**, una vez para todas las instancias. Pasos:

1. **Inventario exhaustivo primero.** Antes de tocar nada, recorrer TODO (`app/`,
   `components/`) y listar cada instancia de cada pieza (volver, editar, borrar,
   agregar/quitar, labels, headers de sección, acción principal) con `archivo:línea` y su
   estado actual. Útil repartir el barrido en subagentes de solo-lectura. Sin inventario no
   hay homologación: se arregla lo que se ve y se olvida el resto.

2. **Definir el catálogo de piezas (una sola vez).** Acordar el estándar único de cada
   pieza: forma, color (token del design system, **nunca** hex hardcodeado), icono, texto,
   posición. Una pieza = una definición reutilizable.

3. **Mockup-first.** Un "Mockup Tipo" con el catálogo de piezas y los patrones de página
   (listado simple vs formulario/detalle). Luego mockups por sección "ahora vs propuesto".
   Aprobar el diseño ANTES de escribir código.

4. **Reglas de aplicación, no solo estética.** Decidir DÓNDE va cada pieza por su
   semántica, no por copiar. Ej: "volver" solo en páginas a las que se ENTRA (drill-in), no
   en destinos del menú lateral (no hay a dónde volver). Distinguir piezas que parecen
   iguales pero son idiomas distintos (botón "agregar fila" de formulario ≠ disparador de
   popover). Renombrar lo que cumple otro rol (un segundo "Volver" junto al CTA es en
   realidad "Cancelar").

5. **Componente compartido > repetición.** Cambiar el componente base (botón-volver, tabla,
   encabezado) propaga el arreglo a decenas de pantallas con un solo edit. Tokenizar los
   colores nuevos en el design system para que vivan en el tema, no en cada uso.

6. **Barrido en una rama, subagentes por bloque sin solape.** Repartir el trabajo mecánico
   en grupos de archivos que no se pisen, con el spec exacto del estándar. Reservarse los
   archivos delicados (wizards con estado, editores complejos) para hacerlos a mano.

7. **Gate + verificación visual antes de cerrar.** Typecheck + lint + tests + build en
   verde, y screenshot REAL en desktop y mobile de las pantallas clave: la posición de cada
   pieza debe ser idéntica en ambos anchos. Recién ahí push/PR.

8. **Cerrar con auditoría de conformidad.** Tras mergear, un barrido de solo-lectura que
   verifique que NINGUNA instancia quedó fuera de norma (es la regla "un bug es la punta del
   iceberg" aplicada a UI). Lo que aparezca, a la siguiente tanda.

> Trampa de entorno: al introducir un token nuevo de Tailwind/`@theme` y cambiar de rama
> con el dev server vivo, la utilidad puede no regenerarse y la pieza se ve sin estilo
> (fondo transparente). El código está bien si el build del CI pasó; la cura local es
> `rm -rf .next` + reiniciar el dev.

---

## Modo agéntico (computer use, browser use)

Cuando Claude opera navegador o computador:
- Mantener visible qué se está haciendo, paso a paso.
- Pedir confirmación antes de cualquier acción roja.
- Nunca seguir links de correos/mensajes sin verificar el destino real.
- Si se sale del scope acordado, detenerse y reportar.

---

## Apéndice — Trampas técnicas y entorno

Estas dos trampas también viven como **memorias** de Claude (aplican en cualquier sesión
Supabase/Postgres, no solo al abrir este archivo):

- **Supabase Studio rompe transacciones explícitas.** `BEGIN; ... COMMIT;`, `CREATE
  FUNCTION ... $$...$$`, y migraciones con varios ALTER encadenados fallan en el SQL Editor
  ("Run selected" ejecuta cada bloque como query suelta). Para operaciones atómicas, usar
  `psql` desde Claude Code. Studio solo para SQL ad-hoc (un SELECT, una inspección).
- **ON CONFLICT con índice único parcial requiere el predicado.** Si el índice es
  `UNIQUE ... WHERE deleted_at IS NULL`, el `INSERT ... ON CONFLICT` debe incluir
  `... WHERE deleted_at IS NULL` (para `DO NOTHING` y `DO UPDATE`). Sin el `WHERE`:
  "there is no unique or exclusion constraint matching the ON CONFLICT specification".

**Entorno de Carlos:** macOS (MacBook Pro M5 Pro) · zsh · Python `/opt/homebrew/bin/python3.11`
· rutas absolutas siempre que se pueda · heredocs con `<< 'PYEOF'` para evitar
interpolación de zsh · nunca commitear carpetas con credenciales (GitHub push protection es
red de seguridad, no primera línea).

---

## Qué se jubiló (contexto)

La versión 2024-2025 de este método (23 "reglas duras") asumía el flujo *claude.ai dicta
bloques → Carlos los pega en el terminal*. Esas reglas — no entregar archivos, envolver
todo en `pbcopy`, editar con heredocs de Python, "un bloque copiable por paso", briefs en
artefacto, script `-ctx` para cargar contexto — murieron al pasar a Claude Code, donde
Claude edita y corre directo. Lo que sobrevivió son las barandas de seguridad y la calidad
de código, recogidas arriba.
