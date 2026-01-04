# `<ImageBackground>` en React Native

## Definición clara

### ¿Para qué sirve este componente?

`<ImageBackground>` es un componente que renderiza una **imagen como fondo** y te permite colocar **contenido encima** (children), como si fuera un contenedor tipo `View` pero con imagen de fondo.

En una frase:

> `ImageBackground` = `Image` + capacidad de actuar como “contenedor” para superponer contenido.

### ¿Qué problema resuelve?

En React Native, un `View` no puede tener `backgroundImage` (como en CSS web).
Si quieres un fondo con imagen (pantallas de login, headers, tarjetas, secciones), necesitas un patrón nativo.

`ImageBackground` resuelve:

* fondos con imagen
* superposición de texto/botones/inputs encima del fondo
* layouts comunes sin hacks (colocar una `Image` absoluta detrás)

### ¿En qué casos se debe usar y en cuáles NO?

#### ✅ Úsalo cuando:

* Necesitas un fondo con imagen y contenido superpuesto:

  * login / onboarding
  * pantalla de perfil
  * hero header
  * tarjetas con imagen y texto
  * secciones con banner
* Quieres un patrón simple y legible (más que `Image` absoluta)

#### ❌ NO lo uses cuando:

* Solo necesitas mostrar una imagen sin contenido encima → usa `Image`
* Estás poniendo muchos backgrounds distintos en una lista larga (posible impacto de memoria)
* Necesitas rendimiento extremo / efectos avanzados (parallax complejo) → a veces conviene `react-native-reanimated` + `Image` con estrategias específicas
* Necesitas backgrounds remotos pesados sin caching → considera `expo-image` (mejor manejo)

> Regla práctica:
> **Fondo de pantalla o sección con contenido encima → `ImageBackground`**
> **Imagen normal → `Image`**

---

## Diferencias clave con componentes similares

### `ImageBackground` vs `Image`

* `Image` muestra imagen, pero no está pensado como contenedor de children
* `ImageBackground` sí acepta children y te permite superponer UI fácilmente

### `ImageBackground` vs `View + Image absolute`

* `Image absolute` detrás puede ser más flexible a veces
* pero `ImageBackground` es más directo y reduce boilerplate
* ambos son válidos, `ImageBackground` es más “estándar” para casos comunes

### `ImageBackground` vs usar un color como fondo

* Para fondos simples o temáticos, un color es más ligero y suele rendir mejor
* `ImageBackground` se reserva para cuando la imagen aporta valor real

---

## Ejemplo mínimo funcional

El ejemplo más simple que funciona:

```jsx
import React from "react";
import { ImageBackground, Text } from "react-native";

export default function App() {
  return (
    <ImageBackground
      source={require("./assets/bg.jpg")}
      style={{ flex: 1, justifyContent: "center", alignItems: "center" }}
      resizeMode="cover"
    >
      <Text style={{ color: "white", fontSize: 22 }}>
        Hola con fondo
      </Text>
    </ImageBackground>
  );
}
```

### Explicación breve

* `source={require(...)}`

  * imagen local empaquetada (súper común y estable)
* `style={{ flex: 1 }}`

  * hace que el fondo cubra toda la pantalla
* `resizeMode="cover"`

  * llena el área recortando si es necesario (comportamiento típico de fondo)
* `children`

  * texto encima del fondo

---

## Ejemplo complejo y realista (cercano a producción)

Caso real: **pantalla de Login** con:

* fondo de imagen
* overlay para legibilidad
* `SafeAreaView`
* `ScrollView` (para teclado / pantallas pequeñas)
* estado de formulario
* botón con feedback (Pressable)
* props relevantes (`resizeMode`, `imageStyle`, `contentContainerStyle`)

```jsx
import React, { useMemo, useState } from "react";
import {
  ImageBackground,
  SafeAreaView,
  ScrollView,
  View,
  Text,
  TextInput,
  Pressable,
  StyleSheet,
} from "react-native";

export default function LoginScreen({ onLogin }) {
  const [email, setEmail] = useState("");
  const [pass, setPass] = useState("");
  const [loading, setLoading] = useState(false);

  const canSubmit = useMemo(() => {
    return email.includes("@") && pass.length >= 6 && !loading;
  }, [email, pass, loading]);

  const submit = async () => {
    if (!canSubmit) return;
    setLoading(true);
    try {
      await onLogin?.({ email, pass });
    } finally {
      setLoading(false);
    }
  };

  return (
    <ImageBackground
      source={require("./assets/bg.jpg")}
      style={styles.bg}
      resizeMode="cover"
      imageStyle={styles.bgImage} // estilo SOLO para la imagen
    >
      {/* Overlay oscuro para legibilidad */}
      <View style={styles.overlay} />

      <SafeAreaView style={{ flex: 1 }}>
        <ScrollView
          contentContainerStyle={styles.content}
          keyboardShouldPersistTaps="handled"
          showsVerticalScrollIndicator={false}
        >
          <Text style={styles.title}>Bienvenido</Text>
          <Text style={styles.subtitle}>Inicia sesión para continuar</Text>

          <View style={styles.card}>
            <Text style={styles.label}>Email</Text>
            <TextInput
              value={email}
              onChangeText={setEmail}
              placeholder="correo@ejemplo.com"
              autoCapitalize="none"
              keyboardType="email-address"
              style={styles.input}
            />

            <Text style={styles.label}>Contraseña</Text>
            <TextInput
              value={pass}
              onChangeText={setPass}
              placeholder="••••••••"
              secureTextEntry
              style={styles.input}
            />

            <Pressable
              onPress={submit}
              disabled={!canSubmit}
              style={({ pressed }) => [
                styles.button,
                !canSubmit && styles.buttonDisabled,
                pressed && canSubmit && styles.buttonPressed,
              ]}
            >
              <Text style={styles.buttonText}>
                {loading ? "Entrando..." : "Entrar"}
              </Text>
            </Pressable>

            <Pressable onPress={() => console.log("Olvidé mi contraseña")}>
              <Text style={styles.link}>¿Olvidaste tu contraseña?</Text>
            </Pressable>
          </View>
        </ScrollView>
      </SafeAreaView>
    </ImageBackground>
  );
}

const styles = StyleSheet.create({
  bg: { flex: 1 },
  bgImage: {
    // Ejemplo: suaviza esquinas (si fuera tarjeta) o ajusta opacidad/blur (limitado)
  },
  overlay: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: "rgba(0,0,0,0.45)",
  },
  content: {
    flexGrow: 1,
    padding: 20,
    justifyContent: "center",
    gap: 10,
  },
  title: { color: "white", fontSize: 26, fontWeight: "800" },
  subtitle: { color: "white", opacity: 0.9, marginBottom: 10 },
  card: {
    borderWidth: 1,
    borderRadius: 14,
    padding: 14,
    backgroundColor: "rgba(255,255,255,0.92)",
    gap: 10,
  },
  label: { fontWeight: "700" },
  input: {
    borderWidth: 1,
    borderRadius: 10,
    padding: 10,
  },
  button: {
    borderWidth: 1,
    borderRadius: 12,
    padding: 12,
    alignItems: "center",
    marginTop: 6,
  },
  buttonPressed: { opacity: 0.7 },
  buttonDisabled: { opacity: 0.4 },
  buttonText: { fontWeight: "800" },
  link: { marginTop: 8, fontWeight: "700" },
});
```

### Qué demuestra este ejemplo

* Uso real con overlay (para legibilidad)
* Integración con estado (`useState`, `useMemo`)
* Contenido scrollable (pantallas pequeñas + teclado)
* Props relevantes de fondo (`resizeMode`, `imageStyle`)
* Estilos organizados

---

## Props más importantes

### Props clave de `ImageBackground`

* **`source`**

  * La imagen (local con `require` o remota `{ uri }`)
  * Úsalo siempre con imágenes optimizadas

* **`resizeMode`**

  * Cómo se ajusta la imagen al contenedor
  * Valores típicos: `"cover"`, `"contain"`, `"stretch"`, `"repeat"`, `"center"`
  * Para fondos casi siempre: `"cover"`

* **`style`**

  * Estilo del contenedor (tamaño, layout)
  * `flex: 1` si es pantalla completa

* **`imageStyle`**

  * Estilo aplicado **solo a la imagen** (no al contenedor)
  * Útil para `borderRadius`, `opacity`, etc.
  * Muy útil si el `ImageBackground` es una tarjeta

* **`children`**

  * Todo lo que va encima del fondo

* **`onLoad` / `onError`**

  * Para imágenes remotas o diagnósticos
  * Útil si quieres mostrar fallback o log

### Cuándo conviene usar cada una

* `resizeMode="cover"`: fondos de pantalla o banners
* `imageStyle={{ borderRadius: 12 }}`: tarjetas con imagen de fondo
* `onError`: cuando la imagen es remota y puede fallar

---

## Prop `style`

### Rol de `style` en este componente

`style` define el **contenedor**, no la imagen.
Esto es importante porque mucha gente espera que `style` afecte directamente a la imagen.

* `style`: layout, tamaño, posición del contenedor
* `imageStyle`: estilos visuales de la imagen

### Propiedades de estilo más comunes

En `style`:

* `flex`, `width`, `height`
* `justifyContent`, `alignItems`
* `padding`, `margin`

En `imageStyle`:

* `borderRadius` (muy común)
* `opacity` (con cuidado)
* `transform` (si quieres mover/escala)
* `resizeMode` también existe como prop, pero mejor usar `resizeMode` directo

### Estilos que sí funcionan y cuáles no

**Sí funcionan (como contenedor):**

* Casi todo lo de `ViewStyle`

**Cosas que confunden:**

* `borderRadius` en `style` a veces no “recorta” la imagen como esperas
  → suele ser mejor poner `borderRadius` en **`imageStyle`**.

Ejemplo tarjeta:

```jsx
<ImageBackground
  source={...}
  style={{ height: 180, borderRadius: 16, overflow: "hidden" }}
  imageStyle={{ borderRadius: 16 }}
>
  ...
</ImageBackground>
```

> Nota: a veces necesitas `overflow: "hidden"` para que el contenido y/o la imagen se recorten bien.

---

## Diferencias relevantes entre iOS y Android

* **Sombras y recortes**

  * Android a veces se comporta distinto con `borderRadius` + `overflow: hidden`
* **`resizeMode="repeat"`**

  * puede comportarse distinto según plataforma y versión
* **Performance**

  * imágenes grandes pesan en ambas plataformas; Android puede ser más sensible a memoria

En general, el comportamiento es bastante consistente, pero los detalles de recorte/sombras son donde más se nota.

---

## Errores típicos (mínimo 8)

1. **Olvidar `flex: 1` cuando es pantalla completa**

* Síntoma: el fondo no cubre toda la pantalla
* Solución: `style={{ flex: 1 }}`

2. **Usar imagen enorme sin optimizar**

* Síntoma: app lenta, consumo de RAM, crasheos en Android
* Solución: usa imágenes del tamaño razonable y comprímelas

3. **Texto ilegible por falta de overlay**

* Síntoma: no se lee nada
* Solución: overlay con `rgba(...)` o blur (si usas librería)

4. **Confundir `style` vs `imageStyle`**

* Síntoma: `borderRadius` no funciona como esperas
* Solución: `imageStyle` + a veces `overflow: "hidden"`

5. **Poner el fondo dentro del `ScrollView` sin querer**

* Síntoma: el fondo se mueve con el contenido
* Solución: `ImageBackground` debe envolver al `ScrollView` si quieres fondo fijo

6. **Usar `{ uri }` sin manejar error**

* Síntoma: fondo en blanco si falla la carga
* Solución: `onError`, fallback, o caching

7. **No considerar safe area**

* Síntoma: contenido debajo del notch
* Solución: `SafeAreaView` dentro (o safe-area-context)

8. **Demasiadas pantallas con fondos pesados**

* Síntoma: navegación pesada, “tirones”
* Solución: reutilizar assets, usar imágenes ligeras, precarga selectiva

---

## Componentes y herramientas auxiliares

### Componentes que se usan junto con `<ImageBackground>`

* `SafeAreaView` (para notch)
* `ScrollView` (formularios o pantallas largas)
* `View` overlay absoluto (legibilidad)
* `Pressable` (botones encima del fondo)
* `TextInput` (login, formularios)

### Hooks, APIs o utilidades relacionadas

* `useWindowDimensions` (adaptar layout a tamaños de pantalla)
* `Platform` (ajustes de comportamiento por OS)
* `StatusBar` (cambiar estilo en pantallas oscuras/claras)

### Librerías comunes que lo complementan

* **`expo-image`** (recomendado si usas imágenes remotas/caching avanzado)
* `react-native-linear-gradient` o `expo-linear-gradient`

  * overlays más elegantes que un `View` con rgba
* `react-native-reanimated`

  * parallax o animaciones con scroll

Por qué se usan juntos:

* `SafeAreaView` protege bordes
* `ScrollView` maneja contenido largo/teclado
* overlay/gradient mejora legibilidad
* `expo-image` mejora rendimiento y caching

---

## Buenas prácticas

### Recomendaciones en proyectos reales

* Usa `ImageBackground` cuando la imagen aporta valor real (no por “decoración gratis”)
* Siempre considera legibilidad: overlay o gradient
* Mantén `ImageBackground` como contenedor de layout, y el contenido en componentes separados
* Si el fondo es fijo y hay scroll: `ImageBackground` **envuelve** al `ScrollView`

### Consideraciones de rendimiento

* Optimiza imágenes (tamaño y compresión)
* Evita fondos pesados en pantallas que cambian mucho
* Para imágenes remotas, considera `expo-image`

### Organización de estilos y código

* `styles` separados (`StyleSheet.create`)
* overlay como componente/estilo reutilizable
* separar pantallas (layout) de componentes (formulario/botones)

---

## Modelo mental final

> `ImageBackground` es la solución nativa a “background-image” del mundo web.
> Úsalo cuando necesitas una imagen de fondo con contenido encima, y cuida legibilidad + peso de assets.

Si quieres, te paso también una plantilla lista para **tarjetas tipo “card con imagen”** (con `borderRadius`, overlay y botón) que queda muy bien para tu app.
