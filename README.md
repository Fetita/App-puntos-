# Anotador de Truco

El anotador de truco más lindo: fósforos de tiza, malas y buenas, envido y truco
a un toque. De 2 a 7 jugadores (incluye modos rotativos). Funciona sin internet.

La app es una web app (HTML/CSS/JS, sin build) envuelta con **Capacitor** para
publicarse en **App Store** y **Google Play**.

## Estructura

```
www/                 ← la app (esto es lo que se edita)
  index.html         ← toda la app: UI + lógica + persistencia
  manifest.webmanifest
  sw.js              ← service worker (offline)
  icons/             ← íconos PWA
assets/              ← arte fuente (icon.svg, feature-graphic.svg) + master PNGs
android/             ← proyecto nativo Android (generado por Capacitor)
store/               ← textos, capturas y política de privacidad para las tiendas
capacitor.config.json
LAUNCH_PLAN.md       ← 👉 guía completa para publicar en ambas tiendas
```

## Desarrollo

```bash
npm install            # Capacitor y herramientas
npm run serve          # servir www/ en http://localhost:5000
```

Editás `www/index.html` y recargás. No hay paso de build.

## Compilar para las tiendas

```bash
npx cap sync           # copia www/ a los proyectos nativos
npm run open:android   # abrir en Android Studio
# iOS (en una Mac): npx cap add ios && npm run open:ios
```

Ver **[LAUNCH_PLAN.md](LAUNCH_PLAN.md)** para el paso a paso completo de
publicación (cuentas, firma, fichas de tienda, revisión).

## Privacidad

La app no recopila ningún dato; todo se guarda solo en el dispositivo.
Ver [store/PRIVACY.md](store/PRIVACY.md).
