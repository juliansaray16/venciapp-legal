# VenciApp — Documentos legales

Repositorio fuente de los documentos legales de **VenciApp**, marca de
**AKRUX S.A.S.** (NIT 902.086.312-4).

## Dónde vive cada cosa

| Archivo | Qué es |
|---------|--------|
| `VenciApp_Terminos_y_Condiciones.md` | Fuente única de los T&C (v2.0). |
| `VenciApp_Politica_Tratamiento_Datos.md` | Fuente única de la política de datos (v2.0). |
| `NOTAS_LEGALES_INTERNAS.md` | Notas de trabajo. **No es público.** |
| `historico/` | Versiones sin vigencia, conservadas como registro. |
| `index.md`, `privacy-policy.md`, `_config.yml` | Sitio de GitHub Pages (hoy, solo redirección). |

## Publicación

Los dos documentos de arriba son la **fuente**. El sitio público los renderiza
directamente desde copias en `venciapp-web/content/legal/` — no se transcriben a
JSX, precisamente porque esa transcripción se desincronizó una vez y llegó a
declarar un responsable del tratamiento distinto.

Flujo al modificar un documento:

1. Editar el `.md` en este repositorio.
2. Copiarlo a `venciapp-web/content/legal/` (`terminos.md` / `privacidad.md`).
3. Si el cambio es sustancial, subir `VERSION_TERMINOS` / `VERSION_POLITICA` en
   `venciapp/src/config/constants.ts` — de ahí sale la versión que se graba como
   prueba de aceptación en el registro de cada usuario.

URLs vigentes:

- https://venciapp.co/privacidad
- https://venciapp.co/terminos

GitHub Pages (https://juliansaray16.github.io/venciapp-legal/) se mantiene vivo
solo porque esa fue la URL registrada originalmente ante las tiendas; hoy
redirige al sitio oficial.

## Contacto

Protección de datos personales: **admin@venciapp.co**
