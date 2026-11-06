# Facturación Agua La Ciénaga

Aplicación web para llevar la facturación mensual del acueducto **Agua La Ciénaga**: suscriptores con su medidor, consumo de cada mes, cálculo automático de la factura, control de pagos y PDF para imprimir 3 o 4 facturas por hoja.

Es una sola página (`index.html`). No hay que instalar nada: se publica con **GitHub Pages** y guarda los datos en **Firebase** (Firestore + inicio de sesión).

## Qué hace

- **Suscriptores**: número, nombre, medidor, teléfono y dirección. Se pueden agregar uno a uno o **subir desde un CSV** (Excel → Guardar como → CSV). Ver `ejemplos/plantilla_suscriptores.csv`.
- **Facturación por periodo (mes)**:
  - Se escriben los m³ consumidos, o la lectura del medidor y el consumo se calcula solo.
  - Valor del m³, cargo fijo y recargo (por ejemplo, matrícula).
  - **Tarifa plena**: después de 14 m³ los metros adicionales se cobran a $3.000 (el límite, el valor y el modo se cambian en Configuración).
  - **Estado de cuenta**: si la factura del mes anterior no se pagó, pasa sola como saldo anterior; si se abonó, pasa solo lo que quedó debiendo.
- **Pagos**: quién pagó, quién abonó y quién no ha pagado; pagos con fecha y número de recibo; meses sin pagar; historial por suscriptor; lista de cobro en PDF.
- **PDF de facturas**: 3 o 4 por hoja, papel carta, oficio o A4, con líneas para recortar. Todas, solo las que deben, o por rango.
- **Resumen y cartera**, exportación a Excel (CSV) y **copia de seguridad** (descargar / restaurar en `.json`).

## Archivos

| Archivo | Para qué es |
|---|---|
| `index.html` | Toda la aplicación |
| `firebase-config.js` | Datos de tu proyecto de Firebase (se llena una sola vez) |
| `firestore.rules` | Reglas de seguridad: quién puede ver y modificar los datos |
| `ejemplos/plantilla_suscriptores.csv` | Formato para subir suscriptores |

## Puesta en marcha

### 1. Crear el proyecto de Firebase

1. Entra a <https://console.firebase.google.com> → **Agregar proyecto** (por ejemplo `agua-cienaga`). Se recomienda un proyecto nuevo, separado de otras apps.
2. **Compilación → Authentication → Comenzar → Correo electrónico/contraseña → Habilitar.**
3. En **Authentication → Usuarios → Agregar usuario**, crea el correo y la contraseña de cada persona que va a facturar. (La app no tiene registro público: solo entran los usuarios que crees aquí.)
4. **Compilación → Firestore Database → Crear base de datos** (modo producción, ubicación `southamerica-east1` o la más cercana).
5. En **Firestore → Reglas**, pega el contenido de `firestore.rules`, cambia los correos de ejemplo por los reales y dale **Publicar**.
6. En **Configuración del proyecto (⚙️) → Tus apps → Web (`</>`)**, registra la app y copia el bloque `firebaseConfig`.

### 2. Conectar la app

Edita `firebase-config.js` en GitHub (ícono del lápiz) y pega los valores:

```js
window.FIREBASE_CONFIG = {
  apiKey: "AIza...",
  authDomain: "agua-cienaga.firebaseapp.com",
  projectId: "agua-cienaga",
  storageBucket: "agua-cienaga.appspot.com",
  messagingSenderId: "1234567890",
  appId: "1:1234567890:web:abc123"
};
```

> Estos valores no son secretos: identifican el proyecto. La seguridad la dan el inicio de sesión y las reglas de `firestore.rules`.

### 3. Publicar con GitHub Pages

**Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)` → Save.**
En un par de minutos la app queda en `https://atipialab.github.io/Agua/`.

En Firebase, agrega ese dominio en **Authentication → Configuración → Dominios autorizados** (`atipialab.github.io`).

### 4. Cargar los datos

- **Desde la versión anterior de la app**: allá, *Copia de seguridad → Descargar copia (.json)*; aquí, *Copia de seguridad → Restaurar desde archivo…*.
- **Desde Excel**: *Suscriptores → Subir CSV…*.

## Uso mensual

1. **Facturación → + Nuevo periodo**: fechas, valor m³ y cargo fijo (se copian de Configuración).
2. Escribir el consumo (m³ o lecturas) de cada suscriptor.
3. **Generar PDF para imprimir** → 3 o 4 por hoja.
4. **Pagos**: registrar quién paga. Lo que quede debiendo aparece solo en la factura del mes siguiente.

> Antes de crear un periodo nuevo, registra los pagos del anterior. Si no, todo el mes anterior pasa como saldo pendiente (se corrige solo cuando registres el pago).

## Sin Firebase (modo local)

Si `apiKey` está vacío, la app funciona igual pero guarda los datos **solo en ese navegador**. Sirve para probar; para uso real configura Firebase y haz copias de seguridad.

## Estructura de datos (Firestore)

- `config/main` — título, valor m³, cargo fijo, tarifa plena, nota al pie, formato de impresión.
- `suscriptores/{id}` — `num`, `nombre`, `medidor`, `telefono`, `direccion`, `activo`.
- `periodos/{AAAA-MM}` — `nombre`, `desde`, `hasta`, `limite`, `valorM3`, `cargoFijo`, `limiteBasico`, `valorM3Pleno`, `modoPleno` e `items` por suscriptor: `m3`, `lectAnt`, `lectAct`, `recargo`, `ant` (saldo anterior manual), `pagos` [`{v, f, n}`].
