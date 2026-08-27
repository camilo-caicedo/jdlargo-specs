# Historias de usuario

Estructura: una carpeta por épica, las historias dentro.

```
04-historias/
  EP-001-nombre-de-la-epica/
    EP-001-nombre-de-la-epica.md   ← la épica
    HU-001-nombre-de-la-historia.md
    HU-002-...
  backlog.md                        ← índice de todo
```

Las historias se escriben con la skill **`/user-story-writing`**
(plugin `pm-skills@product-on-purpose`). Plantilla de respaldo, por si se necesita
editar a mano: `../_plantillas/historia-de-usuario.md`.

Antes de crear una historia: revisar que la épica exista, que el actor esté en
`00-contexto/actores-y-roles.md` y que los términos estén en el glosario.

**No se abre una historia** que dependa de una `PA-xxx` sin responder, salvo que se
registre el `SUP-xxx` correspondiente.
