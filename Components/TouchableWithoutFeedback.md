# `<TouchableWithoutFeedback>` en React Native

## Definición clara

`<TouchableWithoutFeedback>` es un **componente envoltorio** (wrapper) que **detecta interacciones táctiles** (toques) **sin mostrar ningún feedback visual** por defecto.
No renderiza UI propia: **solo envuelve a un único hijo** y le añade capacidad de respuesta al toque.

---

## ¿Para qué sirve este componente?

Sirve para **capturar eventos táctiles** (`press`, `longPress`, etc.) **cuando no quieres efectos visuales automáticos** como:

* cambio de opacidad
* ripple
* highlight
* animación por defecto

Es ideal cuando:

* El feedback visual **ya lo manejas tú**
* El toque es **funcional**, no “botón”
* Necesitas **detectar toques invisibles** (fondos, overlays)

---

## ¿Qué problema resuelve?

Evita tener que:

* crear botones falsos
* usar `Pressable` u `Opacity` cuando **no quieres efectos**
* manejar eventos táctiles manualmente

Permite **separar interacción de apariencia**.

---

## ¿Cuándo se debe usar y cuándo NO?

### ✅ Úsalo cuando:

* Quieres detectar un toque **sin alterar el diseño**
* El feedback visual lo manejas tú (animaciones, estados)
* Necesitas un **tap invisible** (cerrar teclado, cerrar modal, fondo clickeable)
* El elemento ya tiene estilo propio y **no debe cambiar**

### ❌ NO lo uses cuando:

* Necesitas feedback visual estándar → usa `Pressable` o `TouchableOpacity`
* Estás creando un botón accesible
* El usuario **necesita saber que puede tocarlo**
* Quieres estados `pressed`, `hovered`, etc. (mejor `Pressable`)

---

## Diferencias clave con componentes similares

| Componente               | Feedback visual           | Recomendado hoy   |
| ------------------------ | ------------------------- | ----------------- |
| TouchableWithoutFeedback | ❌ Ninguno                 | ⚠️ Uso específico |
| TouchableOpacity         | ✅ Opacidad                | ⚠️ Legado         |
| TouchableHighlight       | ✅ Fondo                   | ❌ Poco usado      |
| Pressable                | ✅ Totalmente configurable | ✅ **Sí**          |

📌 **Conclusión práctica:**
`TouchableWithoutFeedback` sigue siendo útil, pero **`Pressable` es más moderno y flexible**. Aun así, hay casos donde este sigue siendo el más limpio.

---

# Ejemplo mínimo funcional

### El ejemplo más simple que funciona

```jsx
import { View, Text, TouchableWithoutFeedback } from "react-native";

export default function App() {
  return (
    <TouchableWithoutFeedback onPress={() => alert("Tocado")}>
      <View style={{ padding: 20, backgroundColor: "#ddd" }}>
        <Text>Tócame</Text>
      </View>
    </TouchableWithoutFeedback>
  );
}
```

---

### Explicación breve

* `TouchableWithoutFeedback`: envuelve el contenido
* `onPress`: se ejecuta al tocar
* `View`: **obligatorio**, porque el wrapper no renderiza nada
* No hay cambios visuales al tocar

---

# Ejemplo complejo y realista (producción)

## Caso real: cerrar teclado al tocar fuera de un formulario

Este es **EL caso clásico** donde `TouchableWithoutFeedback` brilla.

```jsx
import React, { useState } from "react";
import {
  View,
  Text,
  TextInput,
  TouchableWithoutFeedback,
  Keyboard,
  StyleSheet,
} from "react-native";

export default function FormScreen() {
  const [text, setText] = useState("");

  return (
    <TouchableWithoutFeedback
      onPress={Keyboard.dismiss}
      accessible={false}
    >
      <View style={styles.container}>
        <Text style={styles.label}>Nombre</Text>

        <TextInput
          value={text}
          onChangeText={setText}
          placeholder="Escribe algo"
          style={styles.input}
        />

        <Text style={styles.help}>
          Toca fuera del input para cerrar el teclado
        </Text>
      </View>
    </TouchableWithoutFeedback>
  );
}
```

### Qué incluye este ejemplo (importante)

* Uso real de `onPress`
* Integración con **estado**
* Uso de **Keyboard API**
* Estilos personalizados
* Caso real de UX móvil
* `accessible={false}` (clave para accesibilidad)

---

## Props más importantes

### `onPress`

* Se ejecuta al tocar
* Uso típico: cerrar teclado, cerrar modal, navegación

### `onLongPress`

* Acción secundaria (opciones, menú contextual)

### `onPressIn` / `onPressOut`

* Útiles si tú controlas animaciones manuales

### `disabled`

* Desactiva interacción
* Ojo: **no cambia apariencia**

### `accessible`

* Por defecto `true`
* Muchas veces conviene ponerlo en `false`

### `accessibilityRole`

* Útil si simula un botón (`"button"`)

---

## Prop `style` (muy importante)

### ❗ Punto clave

`TouchableWithoutFeedback` **NO acepta `style`**.

```jsx
<TouchableWithoutFeedback style={{ padding: 10 }}>
// ❌ NO FUNCIONA
```

### ✔️ La forma correcta

El estilo **va en el hijo**:

```jsx
<TouchableWithoutFeedback>
  <View style={{ padding: 10 }} />
</TouchableWithoutFeedback>
```

---

## Estilos que sí funcionan

Todos los estilos aplicados **al hijo**:

* `padding`, `margin`
* `backgroundColor`
* `borderRadius`
* `flex`, `alignItems`
* estilos de texto

## Estilos que NO funcionan

* `style` en el wrapper
* estados `:hover`, `:active`
* feedback visual automático

---

## Diferencias iOS vs Android

* Android **no muestra ripple**
* iOS **no muestra highlight**
* Comportamiento táctil consistente en ambos
* Accesibilidad más delicada (Android TalkBack puede leerlo si no desactivas `accessible`)

---

# Errores típicos (al menos 8)

### 1. No envolver en un View

**Síntoma:** error o comportamiento raro
✔️ Solución: siempre un único hijo renderizable

---

### 2. Esperar feedback visual

**Síntoma:** “no parece botón”
✔️ Solución: usar `Pressable` o animar manualmente

---

### 3. Usarlo como botón principal

**Síntoma:** mala UX
✔️ Solución: usar `Pressable` o `Button`

---

### 4. Olvidar `accessible={false}`

**Síntoma:** lectores de pantalla leen cosas invisibles
✔️ Solución: desactivar accesibilidad cuando sea fondo

---

### 5. Intentar usar `style`

**Síntoma:** estilos no aplican
✔️ Solución: aplicar estilos al hijo

---

### 6. Usarlo dentro de listas largas

**Síntoma:** rendimiento pobre
✔️ Solución: usar `Pressable` + memoización

---

### 7. Interferir con scroll

**Síntoma:** ScrollView no responde bien
✔️ Solución: usar solo como wrapper externo

---

### 8. Capturar taps que no deberían

**Síntoma:** toques “fantasma”
✔️ Solución: estructura clara de overlays

---

# Componentes y herramientas auxiliares

### Componentes comunes junto a TouchableWithoutFeedback

* `View` (obligatorio)
* `TextInput` (cerrar teclado)
* `Modal` (cerrar al tocar fondo)
* `ScrollView` (wrapper externo)
* `KeyboardAvoidingView`

---

## Hooks y APIs relacionadas

* `Keyboard.dismiss()`
* `useRef` (focus/blur)
* `useState` (control de estado)
* `InteractionManager` (si hay animaciones)

---

## Librerías que lo complementan

* **react-native-reanimated** → feedback manual
* **gesture-handler** → gestos avanzados
* **react-native-paper** → alternativas con feedback y accesibilidad

---

## Buenas prácticas

### ✔️ Recomendado

* Usarlo para **fondos táctiles**
* Usarlo para **cerrar teclado o modal**
* Combinarlo con animaciones propias
* Desactivar accesibilidad cuando no es un botón real

### ❌ Evitar

* Usarlo como botón principal
* Usarlo sin feedback en acciones críticas
* Anidar muchos TouchableWithoutFeedback

---

## Consideraciones de rendimiento

* Es ligero, pero:

  * no tiene control de estados
  * no optimiza renders
* En listas o UI compleja → **`Pressable` es mejor**

---

## Organización de código

* Encapsúlalo en componentes (`DismissKeyboardView`)
* No repitas lógica de `Keyboard.dismiss`
* Centraliza overlays y fondos interactivos

```jsx
const DismissKeyboard = ({ children }) => (
  <TouchableWithoutFeedback onPress={Keyboard.dismiss} accessible={false}>
    <View style={{ flex: 1 }}>{children}</View>
  </TouchableWithoutFeedback>
);
```

---

## Conclusión honesta

`TouchableWithoutFeedback` **no está obsoleto**, pero **es específico**.

👉 Úsalo cuando:

* **quieres interacción sin apariencia**
* el feedback lo controlas tú
* el toque es “invisible”

👉 Para todo lo demás:

* **Pressable** es la opción moderna y recomendada.

Si quieres, en el siguiente mensaje puedo:

* compararlo **línea por línea con Pressable**
* mostrar cómo **reemplazarlo gradualmente**
* o adaptar ejemplos a tu proyecto con **Expo Router**
