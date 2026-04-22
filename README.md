# MYWOD — Training App

App de entrenamiento personalizada PPL (Push / Pull / Legs) con seguimiento de pesos, progresión y biblioteca de ejercicios linkada a Muscle & Strength.

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
- ✅ Guarda pesos y progresión en el navegador (localStorage)
- ✅ 5 días de rutina PPL + Día 6 de test RM (semanas 6, 12, 18)
- ✅ 3 bloques de 6 semanas (Acumulación → Intensificación → Definición)
- ✅ Links a fichas de ejercicios en Muscle & Strength
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
