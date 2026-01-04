# `<ScrollView>` en React Native

## Definición clara

### ¿Para qué sirve este componente?

`<ScrollView>` es un contenedor que permite **desplazar (scroll)** su contenido cuando **no cabe en pantalla**. A diferencia de un `View`, no “recorta” el contenido fuera del área visible: lo vuelve accesible mediante desplazamiento vertical u horizontal.

### ¿Qué problema resuelve?

Resuelve el problema de:

* **contenido más grande que el viewport**
* pantallas con secciones largas (texto, formularios, detalles)
* layouts donde necesitas scroll natural sin paginar

### ¿En qué casos se debe usar y en cuáles NO?

**Úsalo cuando:**

* La cantidad de contenido es **moderada** y **no muy grande**
* Estás haciendo pantallas tipo:

  * “Detalles” (perfil, producto, ficha)
  * “Settings” (muchas opciones)
  * “Formulario” (inputs + secciones)
  * “Artículo” (texto largo)
* Necesitas scroll con contenido **dinámico**, pero no masivo

**NO lo uses cuando:**

* Tienes listas grandes o potencialmente grandes (100+ items, o desconocido)

  * Para eso: **`FlatList` / `SectionList`**
* Necesitas rendimiento con virtualización (render solo lo visible)
* Tu pantalla tiene scroll y además metes otro scroll interno sin control (nested scroll) → mala UX y bugs

> Regla rápida:
> **Contenido “tipo página” (poco/medio) → `ScrollView`**
> **Contenido “tipo lista” (mucho) → `FlatList` / `SectionList`**

---

## Diferencias clave con componentes similares

### `ScrollView` vs `FlatList`

* **ScrollView**

  * Renderiza **todo** su contenido de golpe
  * Fácil de usar, ideal para contenido corto/mediano
  * Peor rendimiento si hay muchos elementos
* **FlatList**

  * Renderiza **solo lo visible** (virtualiza)
  * Mejor rendimiento con listas grandes
  * Requiere `data`, `renderItem`, `keyExtractor`

### `ScrollView` vs `View`

* `View` no permite scroll; si el contenido excede, se “pierde”
* `ScrollView` lo hace accesible con desplazamiento

### `ScrollView` vs `SectionList`

* `SectionList` es como `FlatList` pero por secciones (headers)

---

## Ejemplo mínimo funcional

El ejemplo más simple posible: contenido largo que obliga a scrollear.

```jsx
import React from "react";
import { ScrollView, Text } from "react-native";

export default function App() {
  return (
    <ScrollView>
      <Text style={{ fontSize: 18, margin: 16 }}>
        Esta es una pantalla con contenido largo:
      </Text>

      {Array.from({ length: 30 }).map((_, i) => (
        <Text key={i} style={{ marginHorizontal: 16, marginBottom: 8 }}>
          Línea #{i + 1}
        </Text>
      ))}
    </ScrollView>
  );
}
```

### Explicación breve del código

* `<ScrollView>`: contenedor con scroll vertical por defecto
* `Array.from({ length: 30 })`: genera contenido repetido
* `key={i}`: clave necesaria para map
* `Text`: elementos que exceden altura de pantalla → aparece scroll

---

## Ejemplo complejo y realista

Caso real: **pantalla de “Editar perfil”** con:

* formulario largo
* control de teclado
* scroll suave
* eventos (scroll)
* estado (inputs)
* estilos y layout más cercanos a producción

```jsx
import React, { useMemo, useState } from "react";
import {
  ScrollView,
  View,
  Text,
  TextInput,
  Pressable,
  KeyboardAvoidingView,
  Platform,
} from "react-native";

export default function EditProfileScreen({ initialData }) {
  const [name, setName] = useState(initialData?.name ?? "");
  const [email, setEmail] = useState(initialData?.email ?? "");
  const [bio, setBio] = useState(initialData?.bio ?? "");
  const [saving, setSaving] = useState(false);

  const canSave = useMemo(() => {
    return name.trim().length >= 2 && email.includes("@") && !saving;
  }, [name, email, saving]);

  const handleSave = async () => {
    if (!canSave) return;
    setSaving(true);
    try {
      // Simula request
      await new Promise((r) => setTimeout(r, 600));
      // aquí iría tu llamada real a API / DB
    } finally {
      setSaving(false);
    }
  };

  return (
    <KeyboardAvoidingView
      style={{ flex: 1 }}
      behavior={Platform.select({ ios: "padding", android: undefined })}
    >
      <ScrollView
        style={{ flex: 1 }}
        contentContainerStyle={{ padding: 16, gap: 12 }}
        keyboardShouldPersistTaps="handled"
        showsVerticalScrollIndicator={false}
        onScroll={(e) => {
          // Evento real: podrías ocultar toolbars, animar header, etc.
          // console.log(e.nativeEvent.contentOffset.y);
        }}
        scrollEventThrottle={16}
      >
        <Text style={{ fontSize: 20, fontWeight: "700" }}>
          Editar perfil
        </Text>

        <Seccion titulo="Información básica">
          <Campo label="Nombre">
            <TextInput
              value={name}
              onChangeText={setName}
              placeholder="Tu nombre"
              style={styles.input}
              returnKeyType="next"
            />
          </Campo>

          <Campo label="Email">
            <TextInput
              value={email}
              onChangeText={setEmail}
              placeholder="correo@ejemplo.com"
              style={styles.input}
              keyboardType="email-address"
              autoCapitalize="none"
            />
          </Campo>
        </Seccion>

        <Seccion titulo="Biografía">
          <Campo label="Bio">
            <TextInput
              value={bio}
              onChangeText={setBio}
              placeholder="Cuéntanos un poco..."
              style={[styles.input, { height: 120, textAlignVertical: "top" }]}
              multiline
            />
          </Campo>
        </Seccion>

        <Pressable
          onPress={handleSave}
          disabled={!canSave}
          style={[
            styles.button,
            { opacity: canSave ? 1 : 0.5 },
          ]}
        >
          <Text style={{ fontWeight: "700" }}>
            {saving ? "Guardando..." : "Guardar cambios"}
          </Text>
        </Pressable>

        {/* Espacio final para que el último input no quede pegado */}
        <View style={{ height: 24 }} />
      </ScrollView>
    </KeyboardAvoidingView>
  );
}

function Seccion({ titulo, children }) {
  return (
    <View style={styles.section}>
      <Text style={styles.sectionTitle}>{titulo}</Text>
      <View style={{ gap: 10 }}>{children}</View>
    </View>
  );
}

function Campo({ label, children }) {
  return (
    <View style={{ gap: 6 }}>
      <Text style={{ fontWeight: "600" }}>{label}</Text>
      {children}
    </View>
  );
}

const styles = {
  section: {
    borderWidth: 1,
    borderRadius: 12,
    padding: 12,
  },
  sectionTitle: {
    fontSize: 16,
    fontWeight: "700",
    marginBottom: 10,
  },
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
};
```

### Qué demuestra este ejemplo

* Props relevantes (`contentContainerStyle`, `keyboardShouldPersistTaps`, `onScroll`, `scrollEventThrottle`, `showsVerticalScrollIndicator`)
* Estado real (inputs + saving)
* Caso real (formulario de edición)
* Manejo de teclado (muy importante en RN)
* Estilos organizados

---

## Props más importantes

### Props clave de `ScrollView` (las más usadas)

* **`contentContainerStyle`**

  * Estilos aplicados al contenedor interno (el “contenido”)
  * Útil para `padding`, `gap`, `alignItems`, etc.
* **`horizontal`**

  * Cambia scroll a horizontal
* **`showsVerticalScrollIndicator` / `showsHorizontalScrollIndicator`**

  * Muestra/oculta la barra
* **`onScroll`**

  * Captura evento de scroll (posiciones)
* **`scrollEventThrottle`**

  * Frecuencia con la que se llama `onScroll` (16 ≈ 60fps)
* **`keyboardShouldPersistTaps`**

  * Controla taps cuando el teclado está abierto
  * Valores comunes:

    * `"never"` (default en muchos casos)
    * `"handled"`
    * `"always"`
* **`keyboardDismissMode`** (iOS especialmente)

  * Controla cómo se cierra el teclado al hacer scroll (`"on-drag"`)
* **`refreshControl`**

  * Pull-to-refresh (generalmente con `<RefreshControl />`)
* **`nestedScrollEnabled`** (Android)

  * Permite scroll anidado en ciertos casos

### Cuándo conviene usar cada una

* `contentContainerStyle`: casi siempre en pantallas “tipo página”
* `keyboardShouldPersistTaps="handled"`: formularios con botones/listas dentro
* `onScroll` + `scrollEventThrottle`: headers animados, ocultar/mostrar barras
* `refreshControl`: pantallas tipo feed pequeño o “detalle con refresh”
* `horizontal`: carruseles simples (aunque a veces conviene `FlatList horizontal`)

---

## Prop `style` (y estilos relacionados)

### Rol de `style` en `ScrollView`

* `style`: afecta el contenedor del `ScrollView` (su “caja externa”)
* `contentContainerStyle`: afecta el **contenido interno** que se desplaza

Esta distinción es CLAVE.

Ejemplo típico:

* Quieres padding en el contenido → `contentContainerStyle`
* Quieres que ocupe toda la pantalla → `style={{ flex: 1 }}`

### Propiedades de estilo más comunes

En `style`:

* `flex`, `height`, `width`
* `backgroundColor`
* `padding` (si quieres padding externo)
* `margin`
* `borderWidth`, `borderRadius`

En `contentContainerStyle`:

* `padding`, `paddingBottom` (muy común)
* `gap` (si tu RN lo soporta según versión; si no, usa `View` con márgenes)
* `alignItems`
* `justifyContent` (con cuidado)

### Estilos que sí funcionan y cuáles no

**Sí funcionan:**

* layout (flex/size)
* backgroundColor
* padding/margin
* border

**Cosas que suelen confundir:**

* `justifyContent: "center"` en `contentContainerStyle`:

  * Solo “centra” si el contenido no excede el tamaño
* `gap`:

  * depende de versión/engine; si falla, usa `View` wrappers con `marginBottom`

### Diferencias iOS vs Android

* `keyboardDismissMode="on-drag"` funciona mejor/está más presente en iOS
* `nestedScrollEnabled` es clave en Android si anidas scrolls
* Sensación de “rebote”:

  * iOS suele tener rebote (bounce) más natural
  * Android se siente distinto (depende de versión y config)

---

## Errores típicos (mínimo 8)

1. **Usar `ScrollView` para listas grandes**

* Síntoma: lag, consumo alto, scroll “pesado”
* Solución: usar `FlatList` / `SectionList`

2. **Olvidar `contentContainerStyle` y meter padding en `style`**

* Síntoma: padding raro o no aplica como esperas
* Solución: padding del contenido → `contentContainerStyle`

3. **Scroll anidado sin control**

* Síntoma: gesto se “pelea”, scroll errático
* Solución: evitar nested scroll; si es necesario, `nestedScrollEnabled` (Android) y diseño cuidadoso

4. **No considerar el teclado en formularios**

* Síntoma: inputs tapados por teclado
* Solución: `KeyboardAvoidingView`, o librería `react-native-keyboard-aware-scroll-view`

5. **`onScroll` sin `scrollEventThrottle`**

* Síntoma: eventos inconsistentes o baja frecuencia
* Solución: `scrollEventThrottle={16}` si necesitas respuesta fluida

6. **Poner componentes pesados dentro sin optimización**

* Síntoma: render lento
* Solución: dividir componentes, memoizar, o cambiar a listas virtualizadas

7. **No dar espacio al final en pantallas largas**

* Síntoma: último elemento queda pegado o tapado (tab bar)
* Solución: `contentContainerStyle={{ paddingBottom: X }}`

8. **Creer que `ScrollView` “paginado” es lo ideal para carousels grandes**

* Síntoma: mala memoria / rendimiento
* Solución: `FlatList horizontal` + `pagingEnabled` si aplica

---

## Componentes y herramientas auxiliares

### Componentes que se usan junto con `<ScrollView>`

* **`KeyboardAvoidingView`**

  * Para formularios y teclado
* **`RefreshControl`**

  * Pull-to-refresh
* **`SafeAreaView`**

  * Evitar notch y áreas inseguras
* **`TextInput`**

  * Formularios
* **`Pressable`**

  * Botones dentro del scroll
* **`FlatList`**

  * Si parte del contenido es lista grande, a veces se reestructura

### Hooks/APIs/utilidades relacionadas

* `useState` / `useReducer`: formularios y pantallas con lógica
* `useRef`: `scrollViewRef.current?.scrollTo(...)`
* API de `ScrollView`:

  * `scrollTo({ y, animated })`
  * `scrollToEnd()`

Ejemplo de scroll programático:

```jsx
const ref = useRef(null);

// ...
ref.current?.scrollTo({ y: 0, animated: true });
```

### Librerías comunes que lo complementan

* `react-native-keyboard-aware-scroll-view`

  * Maneja teclado + scroll automático a input enfocado
* `react-native-gesture-handler`

  * Mejores gestos en algunos escenarios
* `react-native-reanimated`

  * Headers animados con scroll (ya más pro)

---

## Buenas prácticas

### Recomendaciones de uso en proyectos reales

* Úsalo para pantallas tipo “página”: settings, detalles, formularios medianos
* Usa `contentContainerStyle` para padding y layout del contenido
* Añade `paddingBottom` extra si hay tab bar o botones fijos
* Si hay inputs, usa manejo de teclado desde el inicio

### Consideraciones de rendimiento

* Recuerda: **renderiza todo**
* Evita meter 200 componentes repetidos
* Divide secciones en componentes y memoiza si hace falta
* Si crece como “lista”, migra a `FlatList`

### Organización de estilos y código

* Estilos base en un objeto `styles` (como hicimos)
* Secciones reutilizables (`Seccion`, `Campo`)
* Mantén el `ScrollView` como “layout”, no como lugar de lógica

---

### Mini resumen mental

* `ScrollView`: excelente para pantallas largas **no masivas**
* `FlatList`: para listas grandes
* `style` ≠ `contentContainerStyle` (diferencia crucial)
* Formularios: teclado + scroll siempre planificados

Si quieres, el siguiente paso natural es que te haga el mismo formato para **`FlatList`** (que es el “siguiente nivel” de scroll en RN y donde salen la mayoría de bugs de rendimiento).
