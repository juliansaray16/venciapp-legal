# Notas legales internas — NO PUBLICAR

Documento de trabajo. Registra (a) los datos que faltan por completar, (b) las
**contradicciones entre los documentos legales y lo que el software realmente
hace**, y (c) los cambios de código que deben quedar listos **antes** de publicar
la versión 2.0 de los Términos y de la Política.

Levantado el 2026-07-27 contra el código de `proyectos/venciapp`.

---

## 1. Datos pendientes de completar (bloquean la publicación)

| Campo | Dónde aparece | Estado |
|---|---|---|
| NIT y dígito de verificación de AKRUX S.A.S. | T&C §1, §30 · Política §1, §20 | **Falta** |
| Dirección y ciudad del domicilio principal | T&C §1, §30 · Política §1, §20 | **Falta** |
| Teléfono de contacto | Política §1, §20 | **Falta** |
| Fecha de publicación | encabezado y cierre de ambos | **Falta** |
| Razón social exacta y país de cada subprocesador | Política §10.3 | **Falta** |
| Tope de responsabilidad (se propuso 1 SMMLV) | T&C §20.4 | **Confirmar** |
| Plazo de retención tras eliminar cuenta (se propuso 90 días) | T&C §23.4 · Política §13.3 | **Confirmar** |
| Ventana de rotación de respaldos (se propuso 30 días) | Política §13.4 | **Confirmar contra la configuración real de Supabase** |

---

## 2. Contradicciones halladas entre los borradores v1.0 y el código

Las tres primeras eran **afirmaciones falsas** en un documento destinado a
publicarse. Ya quedaron corregidas en la v2.0, pero explican por qué cambió el
texto.

### 2.1. La telemetría NO es anónima 🔴

El borrador v1.0 afirmaba en cuatro lugares que la telemetría es "anónima y
agregada", que "no identifica al Usuario" y —lo más grave— que **"no constituye
dato personal en los términos de la Ley 1581 de 2012"** (Política v1.0 §14.4).

El código hace lo contrario:

```
src/services/telemetria.ts:100-103
  export function fijarUsuario(userId, email) {
    Sentry.setUser({ id: userId, ...(email ? { email } : {}) });
  }
```

invocado desde `app/_layout.tsx:196` con `session.user.id` y `session.user.email`
en cada cambio de sesión. **El correo electrónico es dato personal sin discusión
posible.** Publicar lo contrario expone a tres frentes a la vez: sanción de la
SIC por información engañosa al titular, rechazo o retiro en las tiendas por
discrepancia con la ficha de privacidad (App Privacy / Data Safety), y pérdida de
credibilidad de todo el resto del documento en un eventual litigio.

Corregido en Política §12.2 y T&C §16.2, que ahora lo declaran abiertamente y
lo justifican por la finalidad de diagnóstico. **Decirlo cuesta menos que
ocultarlo:** con la finalidad declarada y el opt-out disponible, el tratamiento
es perfectamente lícito.

**Acción de código pendiente:** la etiqueta del interruptor en
`src/screens/AjustesScreen.tsx:438` dice **"Reporte de errores anónimo"** y la
entrada de changelog `src/data/changelog.ts:269` repite lo mismo. Ambas son
inexactas. Dos caminos:

- **(a) Cambiar la etiqueta** a "Reporte de errores" o "Diagnóstico de errores",
  con subtítulo "Incluye tu correo para poder reproducir el fallo. No se envían
  datos de tus contribuyentes." — cambio de una línea, coherente con el documento.
- **(b) Cambiar el código** para no enviar el correo (`Sentry.setUser({ id })`),
  quedándose solo con el UUID. Sigue siendo dato personal (es identificable),
  pero reduce el dato expuesto al proveedor. La etiqueta seguiría sin poder decir
  "anónimo".

Recomendado: **(a) + quitar el email**, es decir ambas. El UUID basta para
agrupar eventos de un mismo usuario, que es para lo que sirve en la práctica.

### 2.2. No existe registro de la aceptación de los términos 🔴

El borrador afirmaba que la autorización "queda registrada como prueba"
(Política v1.0 §6.2). **No es cierto:** no hay casilla de aceptación en
`src/screens/RegisterScreen.tsx` ni columna alguna que registre la aceptación.

El artículo 12 de la Ley 1581 y el artículo 2.2.2.25.2.4 del Decreto 1074
imponen al Responsable el **deber de conservar prueba de la autorización**. Sin
ella, ante un reclamo la posición es indefendible: el tratamiento se presume no
autorizado.

**Acción de código pendiente (bloqueante):**

1. Casilla de aceptación **no premarcada** en el registro, con enlaces a ambos
   documentos.
2. Columnas en `usuarios` (o tabla `aceptaciones_legales`): `acepto_terminos_en`
   (timestamptz), `version_terminos` (text), `version_politica` (text). Guardar
   la versión permite acreditar **qué texto** aceptó cada usuario, que es lo que
   realmente importa cuando el documento cambia.
3. Para los usuarios **ya registrados**: pedir la aceptación una sola vez al
   abrir la app tras la actualización, y registrarla igual.

### 2.3. La eliminación de cuenta deja archivos huérfanos 🟠

`eliminar_mi_cuenta()` (mig `20260602000000`) borra la organización en cascada y
`auth.users`, y el cliente borra el logo del bucket `branding-logos`
(`src/services/cuenta.ts`). Pero **no borra los objetos del bucket
`contribuyente-archivos`**: las filas de `archivos_contribuyente` desaparecen y
los archivos quedan en Storage sin referencia.

La Política §13.2 promete que la eliminación "conlleva la supresión de la
información de Contribuyentes, notas y archivos". Hoy eso es falso para los
archivos.

**Acción de código pendiente:** que la eliminación recoja los `storage_path` de
la organización antes del borrado y los elimine del bucket, o una rutina
programada que barra objetos sin fila asociada. Mientras no exista, la promesa
del §13.2 es un incumplimiento documentado por escrito — que es la peor clase.

### 2.4. Wompi / PSE no existe en el código 🟢

El borrador no lo mencionaba y hice bien en no agregarlo: no hay una sola
referencia a Wompi en el repositorio. La v2.0 usa una redacción prospectiva
neutra (T&C §11.2 y §11.4: "cuando el Titular lo habilite", suscripción
"prepagada y no automáticamente renovable"), que cubre el canal web futuro sin
afirmar que hoy existe.

### 2.5. RevenueCat está inerte 🟢

`src/services/revenuecat.ts:54-60` no inicializa el SDK si faltan las llaves. Hoy
la app **no cobra**. La redacción de T&C §11 describe el mecanismo sin afirmar
que esté activo, así que no requiere cambios cuando se enciendan las llaves.

### 2.6. No se valida la mayoría de edad 🟠

No hay fecha de nacimiento ni validación en el registro. Se resolvió por
declaración del usuario (T&C §2.3.a), que es el estándar de la industria y
suficiente para este tipo de producto. No requiere cambio de código.

---

## 3. Lo que se agregó y por qué (protección de la empresa)

Los borradores v1.0 estaban bien orientados pero eran genéricos: servían para
cualquier app. Lo que faltaba era blindar los riesgos **propios de este
producto**.

### 3.1. Cláusula 6 de los T&C — recordatorios y notificaciones

**El riesgo número uno del negocio.** La propuesta de valor es "te avisamos antes
del vencimiento"; el daño típico del cliente es una sanción de la DIAN por
presentar tarde. Si un usuario alega que no le llegó el aviso, el borrador v1.0
solo ofrecía una exclusión genérica de responsabilidad.

Un dato técnico decisivo: **los push son locales**, programados en el dispositivo
con `expo-notifications`. No hay servidor que garantice la entrega. Si el usuario
reinstala, cambia de teléfono, restaura un backup, revoca el permiso o el sistema
mata la app en segundo plano, **las alertas simplemente dejan de existir sin que
nadie se entere** — ni el usuario ni nosotros.

La nueva §6 enumera esas dependencias una por una, declara que no hay SLA y
—lo más importante— hace que el usuario **acepte expresamente que no debe
depender exclusivamente de VenciApp**. Una exclusión genérica se lee como cláusula
de estilo; una enumeración concreta acredita que el riesgo fue informado.

### 3.2. Cláusula 10 — contenido del usuario

El bucket `contribuyente-archivos` **no restringe tipos MIME**: acepta PDF, Word,
Excel, imágenes, cualquier cosa. En la práctica los contadores subirán RUT,
cédulas, estados financieros y declaraciones. Es decir: datos de terceros, en
volumen, sin filtro, y potencialmente datos sensibles.

Se agregó: licencia limitada de alojamiento, prohibición expresa de cargar datos
sensibles y de menores, facultad de retiro sin obligación de supervisión (evitar
el deber de vigilancia), y la advertencia de que el usuario debe mantener sus
propias copias.

### 3.3. Cláusula 8 y Política §15 — equipos y auxiliares

Los borradores ignoraban por completo el sistema multiusuario, que sí existe
(`usuarios_organizaciones`, `invitaciones`, permisos granulares). Sin esta
cláusula quedaba sin resolver quién responde cuando un auxiliar despedido sigue
teniendo acceso, o cuando un contador invita a alguien con un correo que no le
pertenece. Ahora responde el titular de la cuenta.

### 3.4. Cláusula 20 — limitación de responsabilidad

Cambios respecto de la v1.0:

- Exclusión expresa de **lucro cesante y daños indirectos**, que es donde se
  concentra el valor de una demanda de este tipo.
- Tope cuantitativo con **doble límite** (lo pagado en 12 meses **o** un tope
  absoluto, el menor). Con el tope de 12 meses solo, un cliente Empresarial anual
  podría reclamar un múltiplo del valor del plan.
- **Salvedad de dolo, culpa grave y derechos del consumidor** (§20.5). Es
  contraintuitivo, pero *fortalece* la cláusula: una limitación absoluta es
  cláusula abusiva bajo el artículo 42 de la Ley 1480 y un juez la anula entera;
  una limitación con salvedades se sostiene en lo demás. Se agregó además
  divisibilidad expresa dentro de la propia cláusula.
- Declaración de que la asignación de riesgo es **elemento determinante del
  precio**, que es el argumento que sostiene el tope frente a un juez.

### 3.5. Cláusula 25 — cesión

El titular puede ceder el contrato libremente, incluso por fusión o venta de
activos, con consentimiento anticipado del usuario. Sin esto, **una eventual
venta de la empresa exigiría renegociar con cada usuario uno por uno.** Es la
cláusula más barata de escribir y la más cara de omitir.

### 3.6. Política §10 — transferencia internacional

La v1.0 la despachaba en dos párrafos. Toda la infraestructura está fuera de
Colombia (Supabase, Sentry, Resend, Expo, RevenueCat) y **Estados Unidos no
figura en la lista de países con nivel adecuado de protección** de la Circular
Externa 005 de 2017 de la SIC. Por eso la base de legitimación tiene que ser la
**autorización expresa del titular** (literal a, artículo 26 de la Ley 1581), y
tiene que estar escrita como tal, con la tabla de subprocesadores a la vista.

### 3.7. Otras adiciones

Fuerza mayor · supervivencia de cláusulas · acuerdo íntegro · divisibilidad · no
renuncia · comunicaciones electrónicas con presunción de recibo · prohibición de
scraping y de reconstrucción del catálogo (protege el activo real: los
calendarios) · propiedad de la retroalimentación · derecho a exigir versión
mínima de la app · cláusula de versiones beta (hay testers en TestFlight ahora
mismo) · degradación de plan · mora y suspensión · área responsable de habeas
data (exigida por el Decreto 1074) · reclamo incompleto y traslado por falta de
competencia (artículo 15 de la Ley 1581) · notificación de incidentes a la SIC ·
RNBD.

---

## 4. Publicación: hay dos políticas vivas y divergentes 🟠

- `privacy-policy.md` de este repositorio se publica en
  `https://juliansaray16.github.io/venciapp-legal/` (ver `README.md`, `_config.yml`,
  `index.md`) y trae el correo personal `julian14-04@outlook.com`.
- La app apunta a **otra** dirección: `src/config/constants.ts:11` →
  `https://venciapp.co/privacidad` (y `URL_TERMINOS`), que vive en el repositorio
  `venciapp-web`.

**Tener dos políticas de privacidad publicadas y distintas es un riesgo en sí
mismo**: ante un reclamo, se aplica la que le sea más favorable al titular, y la
discrepancia sugiere descuido en el cumplimiento. No borré `privacy-policy.md`
porque es la que está viva hoy y la que probablemente se registró ante las
tiendas. Decidir una de dos:

- Publicar la v2.0 en `venciapp.co/privacidad` y `/terminos`, y dejar
  `privacy-policy.md` como una redirección a esa URL.
- O mantener GitHub Pages como sede oficial y cambiar `constants.ts`.

Recomendado: lo primero. El dominio propio da mejor impresión ante Apple/Google
y ante un cliente empresarial, y ya está referenciado desde la app.

---

## 5. Orden de ejecución sugerido

1. Completar los datos del §1 (NIT, domicilio, teléfono).
2. Implementar el registro de aceptación (§2.2) — **bloqueante**, es deber legal.
3. Corregir la etiqueta "anónimo" y quitar el email de Sentry (§2.1).
4. Arreglar el borrado de archivos huérfanos (§2.3).
5. Revisión del abogado sobre este texto ya corregido, no sobre el borrador.
6. Publicar en `venciapp.co/privacidad` y `/terminos`; redirigir GitHub Pages.
7. Conectar la fila "Términos y privacidad" de Ajustes
   (`src/screens/AjustesScreen.tsx:399`, hoy no-op) con `Linking.openURL`.
8. Alinear la ficha de privacidad de App Store y Play Store con la Política ya
   corregida — en particular declarar que los informes de fallos se asocian a la
   identidad del usuario.

---

*Este documento es de uso interno. No debe publicarse ni distribuirse a usuarios.*
