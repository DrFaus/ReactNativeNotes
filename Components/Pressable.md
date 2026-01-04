# `<Pressable>` en React Native

## Definición clara

### ¿Para qué sirve este componente?

`<Pressable>` es el componente base moderno de React Native para **capturar interacción táctil** (tap/press) y reaccionar a distintos **estados del gesto**, como:

* presionado (`pressed`)
* hover (web/desktop, si aplica)
* focus (teclado, accesibilidad)

En pocas palabras:

> `Pressable` es un contenedor “tocable” que te permite manejar eventos de presión y aplicar estilos dinámicos según el estado de interacción.

### ¿Qué problema resuelve?

Antes (y aún hoy) existían componentes como `TouchableOpacity`, `TouchableHighlight`, etc. Cada uno tenía “un efecto” específico y menos control.

`Pressable` resuelve:

* Unificar un modelo de interacción: un solo componente, varios eventos
* Permitir estilos **condicionales** según `pressed` sin hacks
* Dar control fino sobre el comportamiento de presión (tiempos, cancelación, hitslop)
* Mejor integración con accesibilidad y casos modernos

### ¿En qué casos se debe usar y en cuáles NO?

**Úsalo cuando:**

* Necesitas un “botón” o elemento clicable **custom**
* Quieres cambiar estilos al presionar (sin animaciones complejas)
* Necesitas eventos como:

  * `onPress`, `onLongPress`
  * `onPressIn`, `onPressOut`
* Haces tarjetas, filas, chips, icon buttons, items de lista, toggles

**No lo uses cuando:**

* Ya tienes un componente de UI listo (por ejemplo `Button` o un componente propio `PrimaryButton`)
* Necesitas gestos avanzados (pan, swipe, pinch) → mejor `react-native-gesture-handler`
* Tu “pressable” está dentro de una lista enorme y lo renders sin optimizar (no es “prohibido”, pero hay que cuidar rendimiento)

> Regla práctica:
> **UI custom interactiva → `Pressable`**
> **Gestos complejos → Gesture Handler**
> **Botón simple de sistema → `Button`**

---

## Diferencias clave con componentes similares

### `Pressable` vs `TouchableOpacity`

* **Pressable**

  * Permite `style` como función (`({ pressed }) => ...`)
  * Más eventos y control más moderno
  * Mejor base para componentes custom
* **TouchableOpacity**

  * Efecto típico: baja opacidad al presionar
  * Menos flexible (aunque aún útil)

### `Pressable` vs `TouchableHighlight`

* `TouchableHighlight` “envuelve” con highlight y requiere `underlayColor`
* `Pressable` te deja implementar tu propio efecto fácilmente

### `Pressable` vs `Button`

* `Button` es limitado en estilo y apariencia
* `Pressable` es 100% personalizable

### `Pressable` vs `react-native-gesture-handler`

* Gesture Handler es para gestos avanzados y control fino de interacción (swipe, pan, etc.)
* `Pressable` es ideal para “tap/press” estándar

---

## Ejemplo mínimo funcional

El ejemplo más simple posible:

```jsx
import React from "react";
import { Pressable, Text } from "react-native";

export default function App() {
  return (
    <Pressable onPress={() => console.log("Presionado!")}>
      <Text>Tap aquí</Text>
    </Pressable>
  );
}
```

### Explicación breve

* `Pressable`: área tocable
* `onPress`: callback cuando se completa el tap
* `Text`: contenido visible

---

## Ejemplo complejo y realista

Caso real: **botón reutilizable tipo “Guardar”** con:

* estado `loading`
* `disabled`
* feedback visual al presionar
* `onLongPress` opcional
* accesibilidad básica
* uso en formulario

```jsx
import React, { useMemo, useState } from "react";
import { View, Text, TextInput, Pressable } from "react-native";

function PrimaryButton({
  title,
  onPress,
  onLongPress,
  disabled,
  loading,
}) {
  const isDisabled = disabled || loading;

  return (
    <Pressable
      onPress={onPress}
      onLongPress={onLongPress}
      disabled={isDisabled}
      hitSlop={10}
      accessibilityRole="button"
      accessibilityState={{ disabled: isDisabled, busy: loading }}
      style={({ pressed }) => [
        styles.button,
        isDisabled && styles.buttonDisabled,
        pressed && !isDisabled && styles.buttonPressed,
      ]}
    >
      <Text style={styles.buttonText}>
        {loading ? "Guardando..." : title}
      </Text>
    </Pressable>
  );
}

export default function FormScreen() {
  const [name, setName] = useState("");
  const [saving, setSaving] = useState(false);

  const canSave = useMemo(() => name.trim().length >= 2 && !saving, [name, saving]);

  const save = async () => {
    if (!canSave) return;
    setSaving(true);
    try {
      // simula request
      await new Promise((r) => setTimeout(r, 800));
      console.log("Guardado:", name);
    } finally {
      setSaving(false);
    }
  };

  return (
    <View style={{ padding: 20, gap: 12 }}>
      <Text style={{ fontSize: 18, fontWeight: "700" }}>Perfil</Text>

      <TextInput
        value={name}
        onChangeText={setName}
        placeholder="Nombre"
        style={styles.input}
      />

      <PrimaryButton
        title="Guardar"
        onPress={save}
        onLongPress={() => console.log("Long press: abrir opciones")}
        disabled={!canSave}
        loading={saving}
      />
    </View>
  );
}

const styles = {
  input: {
    borderWidth: 1,
    borderRadius: 10,
    padding: 10,
  },
  button: {
    borderWidth: 1,
    borderRadius: 12,
    padding: 14,
    alignItems: "center",
  },
  buttonPressed: {
    opacity: 0.7,
  },
  buttonDisabled: {
    opacity: 0.4,
  },
  buttonText: {
    fontWeight: "700",
  },
};
```

### Qué demuestra este ejemplo

* Props relevantes: `disabled`, `hitSlop`, `onLongPress`
* Integración con estado (`saving`, `name`)
* Estilos personalizados + estilo dinámico con `pressed`
* Caso real: formulario y botón de guardado
* Accesibilidad: `accessibilityRole`, `accessibilityState`

---

## Props más importantes

Aquí van las más útiles en proyectos reales:

### Interacción básica

* **`onPress`**

  * Se dispara cuando el usuario hace tap y suelta dentro del área
  * Úsalo para acciones principales

* **`onLongPress`**

  * Se dispara si el usuario mantiene presionado
  * Útil para menús contextuales, acciones secundarias

* **`onPressIn` / `onPressOut`**

  * `onPressIn`: cuando inicia el press
  * `onPressOut`: cuando termina
  * Útiles para feedback instantáneo (cambiar estado, iniciar animación)

### Control de estado y alcance

* **`disabled`**

  * Desactiva interacción
  * Úsalo para evitar taps cuando no hay permiso/validación

* **`hitSlop`**

  * Aumenta el área tocable sin cambiar el layout visible
  * Ideal para iconos pequeños (mínimo ~44x44 recomendado)

* **`pressRetentionOffset`** (menos usada, pero útil)

  * Permite que el dedo se “salga un poco” sin cancelar el press

### Estilos dinámicos

* **`style`**

  * Puede ser objeto/array o función:

  ```js
  style={({ pressed }) => [base, pressed && pressedStyle]}
  ```

  * Esta es una de las razones principales para usar `Pressable`

### Eventos de hover/focus (según plataforma)

* **`onHoverIn` / `onHoverOut`** (web/desktop)
* **`onFocus` / `onBlur`** (teclado/accesibilidad)

---

## Prop `style`

### Rol de `style` en `Pressable`

`style` define cómo se ve tu zona tocable.
La ventaja de `Pressable` es que **puede reaccionar al estado** sin crear estados manuales:

```jsx
<Pressable style={({ pressed }) => [{ padding: 12 }, pressed && { opacity: 0.7 }]} />
```

### Propiedades de estilo más comunes

* Layout: `padding`, `margin`, `flex`, `alignItems`, `justifyContent`
* Bordes: `borderWidth`, `borderRadius`
* Visual: `backgroundColor`, `opacity`
* Sombra (según plataforma): `shadow*` (iOS) / `elevation` (Android)

### Estilos que sí funcionan y cuáles no

**Sí:**

* Casi todo lo de `ViewStyle` (Pressable es un contenedor tipo View)

**Confusiones típicas:**

* `color` no aplica al contenedor: aplica al `Text`
* `fontSize` no aplica al contenedor: aplica al `Text`

### Diferencias iOS vs Android

* Sombras:

  * iOS: `shadowColor`, `shadowOffset`, `shadowOpacity`, `shadowRadius`
  * Android: `elevation`
* Efectos tipo “ripple”:

  * Android tiene ripple nativo (se puede configurar con `android_ripple`)
  * iOS no usa ripple nativo igual

Ejemplo ripple Android:

```jsx
<Pressable
  android_ripple={{ borderless: false }}
  style={{ padding: 12, borderRadius: 10 }}
>
  <Text>Android ripple</Text>
</Pressable>
```

---

## Errores típicos (mínimo 8)

1. **No dar área tocable suficiente**

* Síntoma: “no me agarra el tap”
* Corrección: `hitSlop`, padding mínimo, tamaños adecuados

2. **Usar `Pressable` sin feedback visual**

* Síntoma: el usuario no sabe si presionó
* Corrección: `style` con `pressed`, o ripple en Android

3. **No manejar `disabled` correctamente**

* Síntoma: doble envío, múltiples requests
* Corrección: `disabled={loading || !canSave}`

4. **Poner lógica pesada directamente en `onPress`**

* Síntoma: UI se congela al presionar
* Corrección: async, defer, dividir trabajo, usar loading

5. **Crear `onPress={() => ...}` dentro de listas sin optimizar**

* Síntoma: renders extra, listas lentas
* Corrección: `useCallback` + `React.memo` en items, o `renderItem` optimizado

6. **Creer que `pressed` es un estado global**

* Síntoma: intentas leer `pressed` fuera del `style`/children function
* Corrección: usar `style` función o manejar estado manual si necesitas lógica extra

7. **No cuidar accesibilidad**

* Síntoma: lector de pantalla no entiende el botón
* Corrección: `accessibilityRole="button"` y `accessibilityLabel`

8. **Confundir `Pressable` con gestos avanzados**

* Síntoma: intentas hacer swipe/drag y se siente “mal”
* Corrección: usar `react-native-gesture-handler` para gestos complejos

---

## Componentes y herramientas auxiliares

### Componentes que se usan junto con `<Pressable>`

* `Text` (label)
* `View` (estructura interna)
* `ActivityIndicator` (loading)
* `Ionicons` / iconos (botones de icono)
* `FlatList` (items presionables)
* `TextInput` (formularios con botones)

### Hooks, APIs o utilidades relacionadas

* `useState` (disabled/loading)
* `useCallback` (handlers estables en listas)
* `useMemo` (props derivadas como `canSave`)
* `useRef` (evitar doble tap, throttling simple)
* `Haptics` (Expo) para feedback táctil (si lo usas)

### Librerías comunes que lo complementan

* `react-native-gesture-handler` (cuando el “press” convive con gestos)
* `expo-haptics` (feedback)
* `react-native-reanimated` (animaciones al presionar)
* UI kits (Paper, Tamagui, NativeBase) — suelen envolver Pressable con componentes listos

---

## Buenas prácticas

### Recomendaciones para proyectos reales

* Crea componentes reutilizables (`PrimaryButton`, `IconButton`, `ListItem`)
* Siempre añade feedback visual (pressed/ripple/opacity)
* Usa `disabled` para evitar taps múltiples
* Dale área tocable decente (especialmente iconos)

### Consideraciones de rendimiento

* En listas grandes:

  * memoiza items (`React.memo`)
  * memoiza callbacks (`useCallback`)
  * evita crear funciones inline innecesarias si duele el rendimiento
* `Pressable` en sí no es “lento”, pero el patrón alrededor puede serlo

### Organización de estilos y código

* `styles` separados (objeto o `StyleSheet.create`)
* `Pressable` como “capa interactiva”, contenido en subcomponentes
* Mantén `onPress` pequeño y delega la lógica compleja a funciones/servicios

---

### Modelo mental final

`Pressable` es el “bloque Lego” de interacción:

* tú defines el look
* tú decides el comportamiento
* React te da el estado `pressed` para feedback sin complicarte

Si quieres, el siguiente paso natural es que hagamos **`FlatList`** (porque ahí `Pressable` aparece en casi todos los ítems y es donde se notan los problemas de rendimiento y callbacks).
