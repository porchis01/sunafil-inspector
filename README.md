# SUNAFIL Inspector SST — Guía de Instalación y Despliegue 

## Archivos del proyecto

```
sunafil-pwa/
├── index.html          ← Aplicación principal
├── manifest.json       ← Configuración PWA
├── service-worker.js   ← Funcionalidad offline
├── icons/
│   ├── icon-192.png    ← Ícono de app (192x192)
│   └── icon-512.png    ← Ícono de app (512x512)
└── README.md           ← Esta guía
```

## Cómo subir a GitHub Pages (paso a paso para principiantes)

### PASO 1 — Crear cuenta en GitHub
1. Ir a https://github.com
2. Clic en "Sign up"
3. Completar con correo, usuario y contraseña
4. Verificar el correo

### PASO 2 — Crear repositorio
1. Iniciar sesión en GitHub
2. Clic en el botón verde "New" (arriba a la izquierda)
3. En "Repository name" escribir: `sunafil-inspector`
4. Seleccionar "Public"
5. Clic en "Create repository"

### PASO 3 — Subir los archivos
1. En la página del repositorio recién creado, clic en "uploading an existing file"
2. Arrastrar TODOS los archivos de la carpeta `sunafil-pwa/` a la zona de carga
   - index.html
   - manifest.json
   - service-worker.js
   - La carpeta icons/ con sus dos imágenes
3. En "Commit changes" escribir: "Primera versión"
4. Clic en "Commit changes"

### PASO 4 — Activar GitHub Pages
1. En el repositorio, clic en "Settings" (engranaje)
2. En el menú izquierdo, clic en "Pages"
3. En "Source" seleccionar "Deploy from a branch"
4. En "Branch" seleccionar "main" y carpeta "/ (root)"
5. Clic en "Save"
6. Esperar 2-3 minutos

### PASO 5 — Obtener la URL
Tu app estará disponible en:
`https://TU_USUARIO.github.io/sunafil-inspector/`

---

## Cómo instalar en Android

1. Abrir Chrome en el celular Android
2. Ir a la URL de la app
3. Aparecerá un banner "Instalar SUNAFIL Inspector" — tocar "Instalar"
4. Si no aparece el banner: tocar los 3 puntos (⋮) → "Añadir a pantalla de inicio"
5. La app aparecerá como ícono en tu pantalla de inicio

## Cómo instalar en iPhone/iPad (iOS)

1. Abrir Safari (OBLIGATORIO usar Safari, no Chrome)
2. Ir a la URL de la app
3. Tocar el botón compartir (cuadrado con flecha hacia arriba ↑)
4. Desplazarse hacia abajo y tocar "Añadir a pantalla de inicio"
5. Poner nombre "SUNAFIL SST" y tocar "Añadir"
6. El ícono aparecerá en tu pantalla de inicio

## Cómo usar en escritorio/laptop

1. Abrir Google Chrome
2. Ir a la URL de la app
3. En la barra de direcciones aparecerá un ícono de instalar (📥)
4. Clic en ese ícono → "Instalar"
5. La app se abrirá como ventana independiente (sin barra del navegador)

---

## Nota sobre la API Key de Anthropic

Esta versión de la PWA **NO requiere API Key** porque:
- Toda la lógica de identificación de normas está precargada en el código
- Las normas (NTE G.050 y RS N° 021-83-TR) están embebidas en el archivo index.html
- Funciona 100% offline una vez instalada
- No envía datos a ningún servidor externo

Si en el futuro se requiere integrar IA (Claude), ver la sección "Backend seguro" abajo.

---

## Arquitectura de backend seguro (para futura integración con Claude API)

Para proteger la API Key, la arquitectura recomendada es:

```
[App en el teléfono] → HTTPS → [Servidor proxy] → [API Anthropic]
                                     ↑
                              API Key guardada
                              solo aquí (nunca
                              en el frontend)
```

### Opción recomendada: Cloudflare Workers (gratis)

1. Crear cuenta en https://cloudflare.com
2. Ir a "Workers & Pages" → "Create Worker"
3. Pegar el siguiente código:

```javascript
export default {
  async fetch(request, env) {
    if (request.method !== 'POST') return new Response('Method not allowed', { status: 405 });
    
    const body = await request.json();
    
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': env.ANTHROPIC_API_KEY,
        'anthropic-version': '2023-06-01'
      },
      body: JSON.stringify(body)
    });
    
    const data = await response.json();
    
    return new Response(JSON.stringify(data), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': 'https://TU_USUARIO.github.io'
      }
    });
  }
};
```

4. En "Settings" → "Variables" → agregar variable secreta:
   - Nombre: `ANTHROPIC_API_KEY`
   - Valor: tu API Key de Anthropic
5. La API Key nunca aparece en el frontend

---

## Soporte y versiones

- Versión: 1.0.0
- Compatible con: Chrome 80+, Safari 14+, Firefox 79+, Edge 80+
- Sistemas: Android 8+, iOS 14+, Windows 10+, macOS 10.15+
