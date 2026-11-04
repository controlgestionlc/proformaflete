# Proformas de Fletes · La Cabaña Forestal

App web (HTML único) para generar proformas de flete desde el Excel semanal, con registro histórico en Firestore.

## Qué hace

- **Carga del Excel**: misma lectura y misma lógica de agrupación y cálculo que la v7 (flete, carguío incluido, líneas de carguío, detalle de guías).
- **Anti-duplicados**: cada registro (Proveedor + N° G/D + N° GMM + Concepto + UM) se compara con lo ya guardado. Los repetidos se excluyen y se informa en qué proforma están.
- **Folio automático**: correlativo desde el último registrado. En la vista previa se muestra como *provisional*; el número definitivo se reserva al guardar (transacción, sin choques entre usuarios).
- **Guardado manual**: botón "Guardar" con confirmación. Puedes desmarcar proformas para revisar tarifas antes.
- **Historial**: filtros por mes, semana, estado de factura y texto. Ver, imprimir, PDF, registrar factura y eliminar.
- **Eliminar**: borra la proforma y libera sus registros para volver a cargarlos. Si era el último folio, el número se reutiliza.
- **Facturas**: al ingresar se piden las facturas faltantes de proformas cuya semana comienza desde el **01-10-2026** (`FECHA_CORTE_FACTURA`). Las anteriores se pueden registrar desde el historial con el botón 🧾.
- **Informe Guías a Digitar**, **Resumen PDF** y **Excel con folios** (descarga el mismo Excel con la columna N° PROF completada).

> La columna **N° PROF** del Excel ya no es necesaria. Si viene con valores, solo se usa para separar grupos (por ejemplo, un carguío facturado aparte); el folio siempre lo asigna la app.

## Puesta en marcha

### 1. Firebase
1. En [console.firebase.google.com](https://console.firebase.google.com) crea un proyecto (o usa uno existente).
2. **Firestore Database** → Crear base de datos → modo producción → región `southamerica-east1`.
3. **Firestore → Reglas**: pega el contenido de `firestore.rules` y publica.
4. **Authentication** → Comenzar → habilita **Correo electrónico/contraseña** → pestaña Usuarios → agrega los usuarios que usarán la app.
5. **Configuración del proyecto → Tus apps → Web (`</>`)**: registra la app y copia el objeto `firebaseConfig`.
6. En **Authentication → Configuración → Dominios autorizados** agrega `TU_USUARIO.github.io`.

### 2. Configurar `index.html`
Reemplaza el bloque `firebaseConfig` al inicio del `<script>` con los datos copiados.

### 3. GitHub Pages
1. Crea un repositorio (ej. `proformas-fletes`) y sube `index.html` (y si quieres, `firestore.rules` y este README).
2. **Settings → Pages** → Source: *Deploy from a branch* → `main` / `root` → Save.
3. La app queda en `https://TU_USUARIO.github.io/proformas-fletes/`.

### 4. Primer uso
Al ingresar por primera vez la app pide el **último folio emitido** (el último número que asignaste a mano). La siguiente proforma toma ese número + 1. Se puede ajustar después con el botón **⚙ Folio** (nunca por debajo del mayor folio guardado).

## Estructura en Firestore

| Colección | Documento | Contenido |
|---|---|---|
| `config` | `folio` | `ultimo` (último folio emitido) |
| `proformas` | `{folio}` | Resumen: proveedor, semana, mes, totales, `nFactura`, `fechaFactura`, `fechaRef` (lunes de la semana) |
| `proformas_detalle` | `{folio}` | Líneas, detalle de guías y claves de registros |
| `registros` | `{clave}` | Un doc por fila del Excel → `folio` al que pertenece (control de duplicados) |

El historial lee solo `proformas` (liviano); el detalle se carga al ver, imprimir o eliminar.
