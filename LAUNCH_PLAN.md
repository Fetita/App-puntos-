# 🚀 Anotador de Truco — Plan de lanzamiento (App Store + Google Play)

Este documento es tu guía completa para publicar la app en las dos tiendas.
La base técnica ya está armada en este repo (Capacitor + PWA + persistencia +
íconos). Lo que falta es, sobre todo, trabajo de cuentas, builds y formularios
de las tiendas — pasos que requieren tus credenciales y, para iOS, una Mac.

---

## 0. Qué ya está hecho en este repo ✅

| Área | Estado |
|---|---|
| App envuelta con **Capacitor** (web → nativo) | ✅ `capacitor.config.json`, proyecto `android/` generado |
| **Persistencia** de la partida (no se pierde al cerrar) | ✅ `localStorage`, ver `www/index.html` |
| **Funciona offline** (service worker) | ✅ `www/sw.js` + `www/manifest.webmanifest` |
| **Instalable como PWA** | ✅ manifest + íconos |
| **Ícono de la app** (todas las densidades Android + maskable) | ✅ `assets/icon.svg` → `www/icons/` y `android/.../mipmap-*` |
| **Capturas para las tiendas** (6.7", 5 pantallas) | ✅ `store/screenshots/` |
| **Feature graphic** de Play (1024×500) | ✅ `store/screenshots/feature-graphic-1024x500.png` |
| **Textos del listado** (ES + EN, ambas tiendas) | ✅ `store/listings.md` |
| **Política de privacidad** | ✅ `store/PRIVACY.md` |

> **Por qué Capacitor:** tu app ya es una excelente web app móvil. Capacitor la
> envuelve en un contenedor nativo para ambas tiendas sin reescribir nada. Es la
> opción con mejor relación esfuerzo/resultado.

---

## 1. Lo que necesitás conseguir vos (requisitos)

| Requisito | Costo | Para qué |
|---|---|---|
| **Cuenta Google Play Console** | USD 25 (pago único) | Publicar en Play |
| **Apple Developer Program** | USD 99 / año | Publicar en App Store |
| **Una Mac con Xcode** | — | Compilar y subir el build de iOS (obligatorio de Apple) |
| **Android Studio** | gratis | Compilar el build de Android (cualquier SO: Win/Mac/Linux) |
| **URL pública para la política de privacidad** | gratis | Requisito de ambas tiendas (ver §5) |

> ⚠️ **iOS exige una Mac.** No hay forma soportada de compilar/subir a la App
> Store desde Linux o Windows. Alternativas si no tenés Mac: pedir prestada una,
> un servicio de Mac en la nube (MacStadium, MacinCloud), o un CI con runners
> macOS (Codemagic, GitHub Actions `macos-latest`, Ionic Appflow).

---

## 2. Camino rápido: publicar en Google Play (Android) 🤖

Esto se puede hacer **desde cualquier computadora** (incluida la tuya).

### 2.1 Instalar herramientas (una sola vez)
1. Instalá **Android Studio**: https://developer.android.com/studio
2. Instalá un **JDK 17** (Android Studio ya trae uno).

### 2.2 Compilar el proyecto
```bash
npm install            # instala Capacitor
npx cap sync android   # copia la web app al proyecto Android
npm run open:android   # abre Android Studio
```
En Android Studio: esperá a que sincronice Gradle, luego **Build → Build Bundle(s) / APK(s) → Build Bundle**.

### 2.3 Firmar la app (clave de subida)
Play exige un **App Bundle (.aab) firmado**. Generá tu keystore (¡guardalo y
respaldalo, no se puede recuperar!):
```bash
keytool -genkey -v -keystore truco-upload.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias truco
```
Luego, en Android Studio: **Build → Generate Signed Bundle / APK → Android App
Bundle**, elegí tu `.jks`, y generá el `app-release.aab`.

> Recomendado: activá **Play App Signing** (Google guarda la clave de firma
> final; vos solo subís con tu clave de "upload"). Es la opción por defecto.

### 2.4 Crear la ficha en Play Console
1. Entrá a https://play.google.com/console → **Crear app**.
2. Nombre: **Anotador de Truco** · Idioma: Español · Tipo: Juego · Gratis.
3. Completá con los textos de `store/listings.md`:
   - Descripción corta y completa.
   - **Ícono:** `www/icons/icon-512.png`.
   - **Feature graphic:** `store/screenshots/feature-graphic-1024x500.png`.
   - **Capturas de teléfono:** las 5 de `store/screenshots/0*-*.png`.
4. **Política de privacidad:** pegá tu URL (ver §5).
5. Completá los cuestionarios obligatorios: **clasificación de contenido**,
   **público objetivo** (apto todo público), **seguridad de los datos**
   (declará: *no se recopilan datos* — coincide con `store/PRIVACY.md`),
   y **app de gobierno/finanzas** (No).
6. Subí el `.aab` en **Producción** (o probá primero en **Testing interno**).
7. Enviá a revisión. Suele aprobarse en **horas a pocos días**.

---

## 3. Camino iOS: publicar en la App Store 🍎 (requiere Mac)

### 3.1 Generar el proyecto iOS (en la Mac)
```bash
npm install
npx cap add ios          # crea la carpeta ios/ (corre pod install)
npx cap sync ios
npm run open:ios         # abre Xcode
```

### 3.2 Íconos y splash de iOS
El ícono maestro está en `assets/icon.png` (1024×1024). Para generar todo el
set de iOS, la forma más simple es la herramienta oficial de Capacitor en la Mac:
```bash
npm i -D @capacitor/assets
npx capacitor-assets generate --ios \
  --iconBackgroundColor '#06301f' --splashBackgroundColor '#06301f'
```
(En este entorno Linux no se pudo instalar `@capacitor/assets` porque su
dependencia `sharp` no descarga el binario detrás del proxy; en tu Mac funciona
sin problema. Como alternativa, podés arrastrar `assets/icon.png` al
**Assets.xcassets → AppIcon** en Xcode.)

### 3.3 Firmar y configurar en Xcode
1. En Xcode, seleccioná el target **App → Signing & Capabilities**.
2. Elegí tu **Team** (tu cuenta de Apple Developer). Xcode gestiona el
   *provisioning* automáticamente.
3. Verificá **Bundle Identifier:** `com.anotadordetruco.app` (coincide con
   `capacitor.config.json`).
4. Subí la versión/compilación: empezá en `1.0` / build `1`.

### 3.4 Subir a App Store Connect
1. En Xcode: **Product → Archive**.
2. En el Organizer: **Distribute App → App Store Connect → Upload**.
3. En https://appstoreconnect.apple.com creá la app (mismo bundle id) y completá
   la ficha con `store/listings.md`:
   - Name, Subtitle, Promotional text, Keywords, Description.
   - **Capturas 6.7":** las de `store/screenshots/` (1290×2796 ✅).
   - **App Privacy:** *Data Not Collected*.
   - **Support URL** y **Privacy Policy URL** (ver §5).
4. Seleccioná el build subido, respondé el cuestionario de export compliance
   (esta app **no usa cifrado propio** → normalmente "No").
5. **Submit for Review.** La revisión de Apple suele tardar **1–3 días**.

> 💡 **Rechazo más común para apps "envueltas":** Apple (guideline 4.2) rechaza
> apps que son "solo un sitio web". La tuya tiene varias funciones nativas reales
> (juego completo, offline, persistencia, sin login) así que está del lado bueno.
> Para reforzar: la descripción enfatiza el uso offline y de mesa.

---

## 4. Mejoras de la app ya incluidas en esta entrega

Estas mejoras se aplicaron porque eran **imprescindibles** para una app de
tienda (no solo una web):

1. **Guardado de la partida** — antes, al refrescar o cerrar la app se perdía
   todo. Ahora el estado se guarda en el dispositivo y la partida se retoma
   exactamente donde quedó. (Crítico: una app nativa se cierra/abre todo el tiempo.)
2. **Funciona sin internet** — service worker que cachea la app. Clave porque se
   usa en la mesa, sin datos.
3. **Instalable + ícono + splash** — manifest PWA e íconos en todas las densidades.

### Ideas para versiones futuras (opcionales)
- Estadísticas históricas entre partidas (victorias por jugador, rachas).
- Vibración háptica al anotar y al ganar (`@capacitor/haptics`).
- Modo "deshacer última mano".
- Tema claro / contraste alto para accesibilidad.
- Localización a portugués (mercado brasileño de truco).

---

## 5. Publicar la política de privacidad (ambas tiendas la exigen)

Necesitás una **URL pública**. La más rápida y gratis:

**Opción A — GitHub Pages:**
1. Subí `store/PRIVACY.md` al repo (ya está).
2. En GitHub: **Settings → Pages → Deploy from branch** → carpeta `/`.
3. La URL queda tipo `https://fetita.github.io/app-puntos-/store/PRIVACY`.

**Opción B — Gist público** o cualquier hosting estático (Netlify, Vercel).

Pegá esa URL en los campos *Privacy Policy* de Play Console y App Store Connect.

---

## 6. Checklist de lanzamiento

**Antes de subir**
- [ ] Cuentas creadas (Play Console + Apple Developer)
- [ ] Política de privacidad publicada con URL pública (§5)
- [ ] `npm install && npx cap sync` corre sin errores
- [ ] App probada en un dispositivo real (Android y/o iOS)
- [ ] Keystore de Android generado y **respaldado en lugar seguro**

**Google Play**
- [ ] Ficha completa con textos de `store/listings.md`
- [ ] Ícono 512, feature graphic y 5 capturas subidas
- [ ] Cuestionarios: clasificación, público, seguridad de datos
- [ ] `.aab` firmado subido y enviado a revisión

**App Store**
- [ ] Proyecto iOS generado en Mac (`npx cap add ios`)
- [ ] Íconos generados (`capacitor-assets` o Xcode)
- [ ] Signing/Team configurado en Xcode
- [ ] Archive subido a App Store Connect
- [ ] Ficha completa + capturas 6.7" + App Privacy = "Data Not Collected"
- [ ] Enviado a revisión

---

## 7. Flujo de trabajo para cambios futuros

Cada vez que edites la app (en `www/`):
```bash
# 1. editás www/index.html, etc.
# 2. subí el número de versión:
#    - Android: android/app/build.gradle  → versionCode +1, versionName
#    - iOS: en Xcode → Version / Build
npx cap sync          # copia la web a los proyectos nativos
# 3. recompilás y resubís el build a cada tienda
```
> Importante: **subí siempre la versión del service worker** (`CACHE` en
> `www/sw.js`, ej. `truco-v2`) cuando cambies archivos, para que los usuarios
> reciban la versión nueva offline.

---

## Resumen ejecutivo

La parte de **ingeniería para que sea una app de tienda ya está resuelta** en
este repo. Lo que queda es operativo y depende de vos:
1. Crear las dos cuentas de desarrollador (USD 25 + USD 99/año).
2. Publicar la política de privacidad (gratis, §5).
3. **Android:** compilar el `.aab` y completar la ficha → publicable hoy desde tu PC.
4. **iOS:** hacerlo desde una Mac con Xcode (o un CI macOS).

Los textos, capturas, ícono y feature graphic están listos para copiar y pegar.
