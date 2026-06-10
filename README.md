# metodo-carlos

Método de trabajo con Claude para proyectos donde Carlos (no-programador) dirige pero no
ejecuta código a mano. Modernizado para **Claude Code** (Claude edita archivos y corre
comandos directo en el terminal).

Archivo principal: [METODO.md](./METODO.md).

## Cómo se aplica

- **Claude Code (terminal):** cada repo lleva un `CLAUDE.md` con el núcleo del método
  embebido — Claude Code lo carga solo al abrir el repo. Es la vía principal. Una URL
  remota a este archivo **no** se auto-carga en Claude Code, por eso va embebido.
- **claude.ai (Project):** en las Instructions del Project, pegar:

      Aplica las reglas del método definido en:
      https://raw.githubusercontent.com/chbarrerab-cmd/metodo-carlos/main/METODO.md

      Al inicio de cada chat, si no tienes el contenido cargado, haz web_fetch de esa URL.

  Después agregar el contexto específico de ese proyecto.

Cuando cambie el método, se actualiza este METODO.md (canónico) y se propaga el núcleo a
los `CLAUDE.md` de los repos.
