# Deploy en cPanel (Max Dominios) - Checklist Rapido

## 1) Estructura recomendada en hosting

Usa esta estructura en tu cuenta:

- `/home/blogscreecom/app` -> codigo backend (carpeta `app` local)
- `/home/blogscreecom/config` -> archivos de config privados (opcional)
- `/home/blogscreecom/sql` -> scripts SQL privados (opcional)
- `/home/blogscreecom/public_html` -> contenido publico (carpeta `public` local)

Regla clave:
- Solo deja archivos publicos dentro de `public_html`.
- No subas credenciales ni SQL a rutas publicas.

## 2) Que subir exactamente

### Fuera de `public_html`
Sube:
- carpeta `app/` completa
- carpeta `config/` (si la usas)
- `sql/init.sql` (opcional, puede quedar local)

### Dentro de `public_html`
Copia el contenido de `public/`:
- `public/index.php` -> `/home/blogscreecom/public_html/index.php`
- `public/.htaccess` -> `/home/blogscreecom/public_html/.htaccess`
- `public/img/` -> `/home/blogscreecom/public_html/img/`
- `public/uploads/` -> `/home/blogscreecom/public_html/uploads/`

## 3) Base de datos en cPanel

1. Crea DB MySQL y usuario en cPanel.
2. Asigna todos los privilegios del usuario a la DB.
3. Importa `sql/init.sql` desde phpMyAdmin.
4. Si la DB tiene prefijo de cPanel, actualiza el nombre final (ejemplo: `cpuser_RedSocialBlogs`).

## 4) Variables de entorno por `.htaccess` (recomendado)

En `/home/blogscreecom/public_html/.htaccess`, agrega al inicio (antes de rewrite):

```apache
# Variables de entorno para Database.php
SetEnv DB_HOST localhost
SetEnv DB_PORT 3306
SetEnv DB_NAME TU_DB_C_PANEL
SetEnv DB_USER TU_USUARIO_DB
SetEnv DB_PASS TU_PASSWORD_DB

<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^ index.php [L]
</IfModule>
```

Notas:
- `DB_HOST` en cPanel normalmente es `localhost`.
- Si `SetEnv` no funciona en tu plan, usa la alternativa del paso 5.

## 5) Alternativa si `.htaccess` no permite `SetEnv`

Edita `app/database/Database.php` y reemplaza defaults por valores de produccion:

- host por defecto: `localhost`
- dbname por defecto: tu DB de cPanel
- user por defecto: tu usuario DB de cPanel
- pass por defecto: tu password DB

## 6) Permisos de carpetas

Asegura permisos:
- `/home/blogscreecom/public_html/uploads` con `755`.
- Si falla la subida, prueba `775`.

No uses `777` salvo prueba temporal.

## 7) Validaciones despues del deploy

1. Abre dominio principal y verifica Home.
2. Prueba rutas limpias: `/login`, `/admin`, `/noticias`.
3. Inicia sesion admin.
4. Crea una publicacion con imagen y confirma que se guarda en `/uploads`.
5. Revisa fecha/hora y acentos en publicaciones.

## 8) Checklist express (copiar y marcar)

- [ ] Subi `app/` fuera de `public_html`
- [ ] Copie contenido de `public/` dentro de `public_html`
- [ ] Importe `sql/init.sql` en phpMyAdmin
- [ ] Configure DB por `SetEnv` en `.htaccess`
- [ ] Verifique permisos de `uploads/`
- [ ] Prove `/login`, `/admin`, crear noticia con imagen

## 9) Troubleshooting rapido

### Error de conexion DB
- Verifica `DB_HOST=localhost`.
- Verifica prefijo real de DB y usuario en cPanel.
- Verifica privilegios del usuario sobre la DB.

### 404 en rutas como `/login`
- Confirma que `.htaccess` esta en `public_html`.
- Confirma que `mod_rewrite` esta habilitado en hosting.

### No sube imagen
- Revisa permisos de `public_html/uploads`.
- Revisa limite `upload_max_filesize` y `post_max_size` en PHP.

## 10) Google Drive real (subida automatica)

Para que las imagenes queden en Google Drive (y no en `/uploads`), ahora el proyecto usa API oficial de Drive con cuenta de servicio.

1. En Google Cloud, crea un proyecto y habilita la API de Google Drive.
2. Crea una **Service Account** y descarga el JSON de credenciales.
3. Sube el JSON a ruta privada de cPanel, por ejemplo:
    - `/home/blogscreecom/config/google-drive-credentials.json`
4. En tu carpeta de Drive del correo `centroregionalcree@gmail.com`:
    - Crea/elige carpeta destino.
    - Compartela con el `client_email` de la Service Account con permiso **Editor**.
    - Copia el `folderId` (lo que va despues de `folders/` en la URL).
5. En `public_html/.htaccess`, define:
    - `SetEnv GOOGLE_DRIVE_CREDENTIALS_PATH /home/blogscreecom/config/google-drive-credentials.json`
    - `SetEnv GOOGLE_DRIVE_FOLDER_ID TU_FOLDER_ID_DE_DRIVE`
6. En el panel admin, usa el boton **Migrar Imagenes a Drive** para pasar publicaciones viejas que estaban en `/uploads`.

Resultado esperado:
- Nuevas imagenes se suben directo a Drive.
- Imagenes antiguas locales se convierten a URL `https://drive.google.com/uc?export=view&id=...`.
