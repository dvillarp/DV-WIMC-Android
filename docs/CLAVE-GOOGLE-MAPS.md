# Clave de Google Maps (paso a paso, ~10 minutos)

La app funciona **sin** clave (APARCAR, historial, cómo llegar, compartir…), pero el mapa se ve gris.
Para ver el mapa necesitas una clave de API de Google. Es gratuita para este uso personal, pero Google
pide tener una cuenta de facturación activada (no se cobra con el uso normal de un móvil).

1. Entra en https://console.cloud.google.com con tu cuenta de Google.
2. Arriba, selector de proyecto → **Proyecto nuevo** → nombre `DV WIMC` → Crear.
3. Menú ☰ → **Facturación** → vincula/crea una cuenta de facturación (tarjeta). Necesario para activar Maps.
4. Menú ☰ → **APIs y servicios → Biblioteca** → busca **Maps SDK for Android** → **Habilitar**.
5. **APIs y servicios → Credenciales → Crear credenciales → Clave de API**. Copia la clave (empieza por `AIza`).
6. (Recomendado) Pulsa la clave → **Restricciones de la aplicación → Aplicaciones Android → Añadir**:
   - Nombre del paquete: `com.dv.wimc`
   - Huella SHA-1: `B3:28:48:24:E8:56:33:D3:88:44:03:85:FF:F4:A2:7C:26:3D:6C:AA`
   - En «Restricciones de API» elige solo *Maps SDK for Android*. Guardar.
7. Ejecuta de nuevo `DV-WIMC-Android-INSTALAR.bat` y pega la clave cuando la pida. Se guarda como secreto
   `MAPS_API_KEY` del repositorio (nunca en el código) y se recompila el APK.

La clave queda dentro del APK (es normal en Android); por eso conviene la restricción del punto 6.
