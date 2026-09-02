# MYWOD — Training App

App de entrenamiento de gimnasio organizada por meses: 4 días fijos con roles distintos (piernas pesado, tren superior + correr, potencia + saltos, cuerpo completo + circuito largo) más un 5º día sorpresa opcional. Cada día combina fuerza pesada + un circuito, y el contenido rota cada semana.

## Cómo subirla a GitHub Pages (5 pasos)

### 1. Crear el repositorio en GitHub
- Entrá a [github.com](https://github.com) → **New repository**
- Nombre: `mywod` (o el que quieras)
- Visibilidad: **Public** (necesario para GitHub Pages gratis)
- No agregues README ni .gitignore
- Click en **Create repository**

### 2. Subir los archivos desde tu computadora

Abrí la terminal en la carpeta donde descomprimiste este zip y ejecutá:

```bash
git init
git add .
git commit -m "Initial commit - MYWOD app"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/mywod.git
git push -u origin main
```

> Reemplazá `TU_USUARIO` con tu nombre de usuario de GitHub.

### 3. Activar GitHub Pages
- En tu repositorio → **Settings** → **Pages** (menú izquierdo)
- Source: **Deploy from a branch**
- Branch: `main` / `/ (root)`
- Click **Save**
- Esperá 1-2 minutos

### 4. Abrir la URL en tu celular
- Tu app estará en: `https://TU_USUARIO.github.io/mywod/`
- Abrila en **Chrome (Android)** o **Safari (iPhone)**

### 5. Instalar en pantalla de inicio

**Android (Chrome):**
- Menú (⋮) → **Agregar a pantalla de inicio**
- Confirmá → aparece el ícono MYWOD

**iPhone (Safari):**
- Botón compartir (□↑) → **Agregar a pantalla de inicio**
- Confirmá → aparece el ícono MYWOD

---

## Características
- ✅ Funciona offline (Service Worker)
- ✅ Guarda tu posición (mes / semana / día) en el navegador (localStorage)
- ✅ Mes de 4 semanas: Base → Más pesado/menos reps → Más volumen → Descarga
- ✅ 4 días fijos (A/B/C/D) + Día E sorpresa opcional, con color por día
- ✅ Vista Referencia: cómo leer series×reps, formatos de circuito y glosario de ejercicios
- ✅ Estructura preparada para sumar más meses
- ✅ Ícono personalizado en pantalla de inicio

## Estructura de archivos
```
mywod/
├── index.html        ← App principal
├── manifest.json     ← Configuración PWA
├── sw.js             ← Service Worker (offline)
├── _config.yml       ← Config GitHub Pages
├── README.md         ← Este archivo
└── icons/
    ├── icon-32.png
    ├── icon-180.png   ← Apple touch icon
    ├── icon-192.png   ← Android icon
    └── icon-512.png   ← Android splash
```
