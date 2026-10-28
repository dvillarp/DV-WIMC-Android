# APP NATIVA ANDROID: WIMC - Where Is My Car - DV

## 1. NOMBRE Y CONCEPTO
App: WIMC (Where Is My Car) DV
Objetivo: Guardar donde has aparcado el coche con un solo botón, verlo en Google Maps, y que te lleve de vuelta. Sin scroll, textos pequeños, todo en una pantalla, ultra simple y fiable.

## 2. ICONO - ESTILO OBLIGATORIO
Estilo referencia: Icono tipo documento PDF de DV PDF Reader (te paso imagen referencia photo4181305136358476998.jpeg)
Especificaciones exactas:
- Fondo exterior: cuadrado redondeado azul marino oscuro #1E3A6B, radio 22% (como iOS/Android adaptive icon)
- Interior: hoja blanca con esquina superior derecha doblada (dog-ear) color #CBD5E1 claro
- En la hoja, letras azules brillantes #1C5DDB: "WIMC" en negrita, mayúsculas, centrado en parte superior de la hoja, fuente sans-serif bold, proporción cuadrada
- Banda horizontal roja #E30613 que cruza la hoja por la mitad inferior, sobresaliendo un poco por los lados de la hoja (efecto sticker)
- Dentro de banda roja, texto blanco "DV" en negrita, centrado
- Debajo de banda, línea gris clara horizontal fina
- Sin sombras interiores, flat design, vector limpio, 1024x1024px

Prompt para generar icono con IA de imágenes:
"App icon in same style as DV PDF Reader reference: outer dark blue rounded square background #1E3A6B, inner white document icon with folded top-right corner light gray #CBD5E1, bold blue uppercase letters 'WIMC' #1C5DDB centered in upper part of document, solid red horizontal band #E30613 with white uppercase letters 'DV' centered inside, thin light gray line below band, clean flat professional document app icon, 1024x1024, no shadows, crisp vector"

## 3. COLORES Y ESTILO UI
- Primario azul: #1446FF (botón APARCAR)
- Rojo DV: #E30613 (banda y des-aparcar)
- Fondo azul oscuro header: #1E3A6B
- Fondo app: #f5f7fb gris muy claro
- Cards blancas: #FFFFFF, radio 12px, sombra 0 1px 3px rgba(0,0,0,0.1)
- Textos: 11-13px máximo, sin scroll vertical en pantalla principal
- Altura header: 52px, bottom nav: 48px, contenido: calc(100dvh - 100px)
- Sin scroll en Inicio y Mapa, solo Historial con scroll interno

## 4. ESTRUCTURA DE PANTALLAS (Bottom Navigation 4 tabs)

### TAB 1: INICIO
Header con logo 36px + "WIMC" + "Where Is My Car · DV" + badge "NO APARCADO" (rojo) / "APARCADO" (verde #16a34a)

Card 1 Estado:
- Texto: "Pulsa APARCAR para guardar" / "Aparcado hace Xmin"
- Precisión GPS: "Precisión: Xm - Excelente/Media"
- Info coche si aparcado: lat,lng, fecha, precisión, en fondo azul claro #f0f7ff

Card 2 Acciones principales (centrado):
- Botón grande: "📍 APARCAR AHORA" azul #1446FF, altura 56px, radio 10px, bold 14px
- Botón "🏁 DESAPARCAR" rojo, mismo tamaño, solo visible si hay coche aparcado
- Fila pequeña: "📡 Ubicación precisa" + "👆 Tocar mapa para aparcar" (outline)
- Texto 10px gris: "Si el GPS falla, toca el mapa para aparcar manual"

Card 3 Acciones secundarias:
- "🗺️ Ver en mapa" | "🧭 Cómo llegar" | "📤 Compartir" (outline, 11px)

### TAB 2: MAPA
- Google Maps nativo (NO OSM/Leaflet). Usar Google Maps SDK Android / Google Maps.
- Mapa ocupa flex 1, min-height 250px
- Barra superior con botón "< Volver a Inicio" (esto faltaba y es obligatorio)
- Instrucción: "👆 Toca en cualquier punto del mapa para aparcar allí (funciona sin GPS)"
- Botones: "📍 Mi posición" (centra en usuario), "🚗 Coche" (centra en coche), "📌 Aparcar centro" (aparca en centro del mapa)
- Click en mapa: Confirm dialog "¿Aparcar aquí? lat,lng" -> guarda
- Si coche aparcado: Marker coche con icono coche
- Si posición usuario: Marker azul "Tú"

### TAB 3: HISTORIAL
- Lista de aparcamientos guardados en local (Room/SharedPreferences/DataStore)
- Cada item: fecha/hora + lat,lng (5 decimales) + botón "USAR" que carga esa posición y va a mapa
- Botón "🗑️ Borrar historial"
- Vacío: "Sin historial"

### TAB 4: AJUSTES
- Texto explicativo GPS:
"Si ubicación no funciona: Activa GPS y WiFi, sal al exterior, da permiso Ubicación Precisa en Android. Si sigue sin ir, el mapa SIEMPRE permite aparcar tocando."
- Versión: "WIMC 4.2 Final DV"
- Botón test: "🚘 Simular parada de coche"
- Info almacenamiento

## 5. LÓGICA CRÍTICA - APARCAR (LO MÁS IMPORTANTE - APRENDIDO TRAS FALLOS)

El botón APARCAR DEBE FUNCIONAR SIEMPRE, aunque falle GPS y falle mapa. Nunca quedarse haciendo nada.

Implementación obligatoria:

fun doPark() {
  isParking = true
  showBanner("Guardando...")
  buttonText = "⏳ Guardando..."

  // Intenta GPS alta precisión con timeout 1200ms
  val fallbackTimer = Timer 1500ms -> si GPS no responde:
    savePark(mapCenter ?: MadridFallback) + showBanner("GPS lento - usando centro del mapa")
    isParking = false

  FusedLocationProviderClient.getCurrentLocation(PRIORITY_HIGH_ACCURACY, CancellationToken)
    .addOnSuccess { location ->
      if (location != null && location.accuracy < 100) {
        savePark(location)
        showBanner("Ubicación precisa: ${accuracy}m")
      } else {
        // GPS con mala precisión -> usar centro mapa
        savePark(mapCenter)
      }
      cancel fallbackTimer
      isParking = false
    }
    .addOnFailure {
      // GPS falló -> NO guardar automáticamente Sol
      showDialog(
        "GPS falló",
        "No se pudo obtener ubicación precisa",
        Opciones: [
          "Reintentar GPS",
          "Aparcar en centro del mapa",
          "Introducir coordenadas manual",
          "Aparcar en Madrid Sol (último recurso)"
        ]
      )
      isParking = false
    }

  // Segundo click mientras isParking = true -> fuerza guardado instantáneo con lo que haya
}

fun savePark(lat,lng,accuracy) {
  parked = {lat,lng,ts=now,acc}
  save to DataStore/SharedPreferences "wimc_parked"
  history.unshift(parked) -> max 50 -> save "wimc_history"
  update UI, marker, showBanner "✅ Coche guardado"
  vibrate 100ms
}

GPS debe pedir:
- ACCESS_FINE_LOCATION + ACCESS_COARSE_LOCATION
- enableHighAccuracy true
- En Android 12+: pedir "Ubicación precisa" (no aproximada)
- Si usuario da aproximada, mostrar aviso "Activa ubicación precisa en Ajustes > Ubicación > WIMC > Precisa"

NO HACER fallback silencioso a Sol (40.4165,-3.70256). Solo guardar Sol si usuario lo elige explícitamente en dialog. Antes lo hacíamos y confundía.

## 6. FUNCIONES ADICIONALES

- DESAPARCAR: Borra parked, borra marker, banner "Des-aparcado"
- VER EN MAPA: Va a tab mapa y centra en coche
- CÓMO LLEGAR: Intent con "https://www.google.com/maps/dir/?api=1&destination=lat,lng" o Google Maps intent
- COMPARTIR: Intent share con "Mi coche: https://www.google.com/maps?q=lat,lng"
- DETECCIÓN AUTOMÁTICA COCHE (opcional pero pedida):
    - Usar ActivityRecognition + FusedLocation speed
    - Si speed >15 km/h -> driving=true
    - Si driving=true y speed <3 km/h por 60s -> mostrar notificación/modal "Parece que has parado. ¿Aparcar aquí?" con opciones "Aparcar ahora / Auto-aparcar siempre / Ignorar"
    - Toggle en ajustes "Detección automática" y "Auto-aparcar"

## 7. TECNOLOGÍA RECOMENDADA NATIVA

Opción A (recomendada):
- Kotlin + Jetpack Compose
- Google Maps: implementation 'com.google.maps.android:maps-compose:4.3.3'
- Play Services Location: 'com.google.android.gms:play-services-location:21.0.1'
- DataStore Preferences
- Activity Recognition

Opción B:
- React Native + @react-native-maps/google-maps + @react-native-google-maps + react-native-geolocation-service

## 8. PERMISOS ANDROID MANIFEST

<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACTIVITY_RECOGNITION" />
<uses-permission android:name="android.permission.VIBRATE" />
<uses-permission android:name="android.permission.INTERNET" />

## 9. ERRORES QUE NO DEBE COMETER (aprendidos)

- No usar Leaflet/OSM, usar Google Maps nativo
- No hacer fallback silencioso a Madrid Sol - siempre preguntar
- No dejar botón APARCAR sin feedback - siempre cambiar texto a "⏳ Guardando..." y banner
- No usar inline onclick con comillas anidadas que rompen JS (usar addEventListener)
- No depender de mapa para aparcar - aparcado debe funcionar aunque mapa falle
- No bloquear GPS en WebView - en nativo usar FusedLocationProviderClient, no WebView
- Textos pequeños, sin scroll, todo visible sin hacer scroll
- Añadir botón atrás en mapa ( "< Volver a Inicio")

## 10. ENTREGABLES

- APK debug
- Icono 1024x1024, 512x512, adaptive icon (foreground + background #1E3A6B)
- Código fuente Kotlin
- README con instrucciones instalación

---

FIN DEL PROMPT