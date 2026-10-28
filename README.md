# DV WIMC · Where Is My Car (Android nativo)

Proyecto **independiente** de DV: app Android nativa (Kotlin + Jetpack Compose + Google Maps SDK) que guarda
dónde has aparcado con un botón y te lleva de vuelta. Requisitos originales en
[`docs/ESPECIFICACION-ORIGINAL.md`](docs/ESPECIFICACION-ORIGINAL.md).

**APK:** pestaña *Releases* de este repositorio → `DV-WIMC.apk`.

## Qué hace
- 4 pestañas: Inicio · Mapa · Historial · Ajustes.
- **APARCAR AHORA** siempre responde: GPS alta precisión con límite de 1,5 s → centro del mapa (solo si lo has movido tú)
  → diálogo (reintentar / centro del mapa / coordenadas manuales / Madrid Sol). Nunca cae en Madrid Sol en silencio.
- Segundo toque mientras guarda = guarda al instante con lo que haya.
- Mapa Google nativo: tocar para aparcar (con confirmación), «Mi posición», «Coche», «Aparcar centro», «< Volver a Inicio».
- Historial (50), USAR, DESAPARCAR, CÓMO LLEGAR (a pie, en Google Maps), COMPARTIR.
- Detección automática de coche (Activity Recognition): «Parece que has parado. ¿Aparcar aquí?» + botón «Simular parada».

## Cómo se compila (todo automático)
Cada `push` a `main` ejecuta `.github/workflows/build-apk.yml`: compila, firma con la clave fija y publica el APK
en Releases. El secreto `MAPS_API_KEY` (Settings → Secrets → Actions) da el mapa; ver
[`docs/CLAVE-GOOGLE-MAPS.md`](docs/CLAVE-GOOGLE-MAPS.md). Sin él, el mapa sale gris pero todo lo demás funciona.

Local: `./gradlew assembleRelease` (JDK 17 + Android SDK 35). Pon `MAPS_API_KEY=...` en `local.properties`.

## Instalar en el móvil
1. Descarga `DV-WIMC.apk` desde Releases en el móvil (o pásalo por cable/Drive).
2. Ábrelo y permite «Instalar apps de origen desconocido» para esa app.
3. Primera vez: pulsa APARCAR y concede **Ubicación precisa**.
4. Auto-aparcar sin preguntar necesita ubicación «Permitir siempre»; sin ella recibes notificación.

## Estructura
| Ruta | Contenido |
|---|---|
| `app/src/main/java/com/dv/wimc/` | `MainActivity`, `UI` (Compose), `WimcViewModel` (lógica de aparcar), `Models` (datos), `DrivingDetection` (coche) |
| `app/src/main/res/` | icono adaptativo (vector), tema, recursos |
| `app/debug.keystore` | firma fija (ver nota) |
| `icon/` | iconos 1024/512, adaptive foreground y SVG fuente |
| `docs/` | especificación original y guía de la clave de Maps |

## Decisiones respecto a la especificación
- Datos en SharedPreferences (`wimc_parked`, `wimc_history`) para que también los lea el receiver en segundo plano.
- Detección con transiciones de Activity Recognition (IN_VEHICLE entra/sale) en vez de umbrales de velocidad: funciona
  en segundo plano sin servicio permanente y gasta menos batería.
- Cómo llegar usa `travelmode=walking`.

## Nota de seguridad sobre la firma
`app/debug.keystore` es una clave de uso personal que se publica en el repo para que los APK mantengan siempre la
misma firma (y se puedan actualizar sin desinstalar). Es adecuada para sideload personal; si algún día se publica en
Google Play, hay que crear una clave privada nueva.
