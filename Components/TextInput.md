# `<TextInput>` en React Native

## Definición clara

`<TextInput>` es el componente nativo de React Native para **capturar texto del usuario** (una línea o múltiples líneas) usando el teclado del sistema (iOS/Android). Es el equivalente funcional de un `<input>` o `<textarea>` en la web, pero con un modelo y limitaciones propias de móvil.

---

## ¿Para qué sirve?

* Permitir que el usuario escriba **texto libre** (nombre, correo, comentarios).
* Capturar **datos estructurados** (teléfono, código postal, OTP, números).
* Implementar **búsqueda** (search bar).
* Crear **formularios** (login, registro, dirección, perfil).
* Entradas más avanzadas: contraseñas, máscara básica, autocompletado, validación en vivo.

---

## ¿Qué problema resuelve?

En móvil no existe “un input web”; necesitas un componente que:

* Abra y controle el **teclado** del sistema.
* Te permita configurar el **tipo de teclado** (numérico, email, phone…).
* Gestione eventos de texto y foco (focus/blur).
* Controle seguridad (password) y autocompletado.
* Se integre con accesibilidad (labels, hints).

`TextInput` es la interfaz estándar para todo eso.

---

## ¿Cuándo se debe usar y cuándo NO?

### ✅ Úsalo cuando…

* Necesitas entrada de texto (obvio, pero práctico).
* Estás haciendo un formulario (login, settings, perfil).
* Necesitas input con teclado del sistema.
* Necesitas controlar/validar el texto en tiempo real.

### ❌ Evítalo cuando…

* Solo quieres texto “clickeable” o un botón: usa `Pressable`/`TouchableOpacity`.
* Lo que necesitas es una selección controlada: usa `Switch`, `Picker` (o `@react-native-picker/picker`), `Radio`, `Slider`, etc.
* Tienes listas enormes con inputs por celda sin optimización: un `TextInput` por item en una `FlatList` larga puede volverse pesado si no cuidas renderizados y foco.

---

## Diferencias clave con componentes similares

* **Text**: solo muestra texto, no permite editarlo.
* **Pressable/TouchableOpacity**: interacción táctil, no captura texto.
* **SearchBar** (de libs como React Native Paper / Elements): normalmente es un wrapper “bonito” alrededor de `TextInput`.
* **Textarea web**: en RN el multi-línea se logra con `multiline`, no es otro componente.

---

# Ejemplo mínimo funcional (lo más simple que funciona)

```jsx
import React, { useState } from "react";
import { View, Text, TextInput } from "react-native";

export default function App() {
  const [nombre, setNombre] = useState("");

  return (
    <View style={{ padding: 20 }}>
      <Text>Tu nombre:</Text>

      <TextInput
        value={nombre}
        onChangeText={setNombre}
        placeholder="Escribe aquí"
        style={{
          borderWidth: 1,
          borderColor: "#999",
          padding: 10,
          marginTop: 8,
          borderRadius: 8,
        }}
      />

      <Text style={{ marginTop: 12 }}>
        Hola, {nombre || "…"}
      </Text>
    </View>
  );
}
```

### Explicación breve por partes

* `useState("")`: guarda el texto actual.
* `value={nombre}`: convierte el input en **controlado** (la fuente de verdad es el estado).
* `onChangeText={setNombre}`: actualiza el estado cada vez que cambia el texto.
* `placeholder`: texto gris cuando está vacío.
* `style`: define borde, padding, etc.

---

# Ejemplo complejo y realista (cerca de producción)

Caso real: **formulario de login** con:

* email + password
* validación básica
* botón deshabilitado
* manejo de foco entre inputs
* submit con `onSubmitEditing`
* teclado adecuado
* estilo y mensajes de error

```jsx
import React, { useMemo, useRef, useState } from "react";
import {
  View,
  Text,
  TextInput,
  Pressable,
  KeyboardAvoidingView,
  Platform,
  ActivityIndicator,
} from "react-native";

export default function LoginScreen() {
  const [email, setEmail] = useState("");
  const [pass, setPass] = useState("");
  const [touched, setTouched] = useState({ email: false, pass: false });
  const [loading, setLoading] = useState(false);

  const passRef = useRef(null);

  const emailError = useMemo(() => {
    if (!touched.email) return "";
    if (!email.trim()) return "El email es obligatorio.";
    if (!email.includes("@")) return "Parece que falta un '@'.";
    return "";
  }, [email, touched.email]);

  const passError = useMemo(() => {
    if (!touched.pass) return "";
    if (pass.length < 6) return "La contraseña debe tener al menos 6 caracteres.";
    return "";
  }, [pass, touched.pass]);

  const canSubmit = !emailError && !passError && email && pass;

  async function onLogin() {
    setTouched({ email: true, pass: true });
    if (!canSubmit) return;

    try {
      setLoading(true);
      // Simula request
      await new Promise((r) => setTimeout(r, 700));
      // Aquí iría navegación / auth real
      alert("Login OK (simulado)");
    } finally {
      setLoading(false);
    }
  }

  return (
    <KeyboardAvoidingView
      style={{ flex: 1, padding: 20, justifyContent: "center" }}
      behavior={Platform.OS === "ios" ? "padding" : undefined}
    >
      <Text style={{ fontSize: 24, fontWeight: "700", marginBottom: 16 }}>
        Iniciar sesión
      </Text>

      {/* Email */}
      <Text style={{ marginBottom: 6 }}>Email</Text>
      <TextInput
        value={email}
        onChangeText={setEmail}
        onBlur={() => setTouched((t) => ({ ...t, email: true }))}
        placeholder="correo@dominio.com"
        autoCapitalize="none"
        keyboardType="email-address"
        textContentType="emailAddress"
        autoComplete="email"
        returnKeyType="next"
        blurOnSubmit={false}
        onSubmitEditing={() => passRef.current?.focus()}
        style={[
          styles.input,
          emailError ? styles.inputError : null,
        ]}
      />
      {!!emailError && <Text style={styles.errorText}>{emailError}</Text>}

      {/* Password */}
      <Text style={{ marginTop: 14, marginBottom: 6 }}>Contraseña</Text>
      <TextInput
        ref={passRef}
        value={pass}
        onChangeText={setPass}
        onBlur={() => setTouched((t) => ({ ...t, pass: true }))}
        placeholder="••••••••"
        secureTextEntry
        textContentType="password"
        autoComplete="password"
        returnKeyType="done"
        onSubmitEditing={onLogin}
        style={[
          styles.input,
          passError ? styles.inputError : null,
        ]}
      />
      {!!passError && <Text style={styles.errorText}>{passError}</Text>}

      {/* Botón */}
      <Pressable
        onPress={onLogin}
        disabled={!canSubmit || loading}
        style={({ pressed }) => [
          styles.button,
          (!canSubmit || loading) ? styles.buttonDisabled : null,
          pressed && (!loading) ? styles.buttonPressed : null,
        ]}
      >
        {loading ? (
          <ActivityIndicator />
        ) : (
          <Text style={styles.buttonText}>Entrar</Text>
        )}
      </Pressable>
    </KeyboardAvoidingView>
  );
}

const styles = {
  input: {
    borderWidth: 1,
    borderColor: "#999",
    borderRadius: 10,
    paddingHorizontal: 12,
    paddingVertical: 10,
    backgroundColor: "white",
  },
  inputError: {
    borderColor: "#d33",
  },
  errorText: {
    color: "#d33",
    marginTop: 6,
  },
  button: {
    marginTop: 18,
    paddingVertical: 12,
    borderRadius: 10,
    alignItems: "center",
    backgroundColor: "#111",
  },
  buttonDisabled: {
    opacity: 0.45,
  },
  buttonPressed: {
    opacity: 0.75,
  },
  buttonText: {
    color: "white",
    fontWeight: "700",
  },
};
```

### Qué hace “cercano a producción”

* Validación basada en `touched` (no molestas al usuario desde el primer render).
* `keyboardType`, `autoComplete`, `textContentType` para UX real.
* `returnKeyType` y `onSubmitEditing` para moverte por el formulario.
* `ref` para focus controlado.
* `KeyboardAvoidingView` para evitar que el teclado tape inputs (sobre todo iOS).
* Botón deshabilitado y loader.

---

# Props más importantes de TextInput

### Control del valor

* **`value`**: texto actual (modo controlado). Úsalo casi siempre.
* **`defaultValue`**: valor inicial (modo no controlado). Útil en prototipos, pero menos recomendable.
* **`onChangeText`**: callback directo con el texto nuevo. Lo más común.

### Experiencia de teclado

* **`keyboardType`**: tipo de teclado (`default`, `numeric`, `phone-pad`, `email-address`, etc.). Úsalo para reducir errores.
* **`returnKeyType`**: etiqueta del botón “Enter” (`next`, `done`, `search`).
* **`onSubmitEditing`**: cuando se presiona “Enter”. Útil para submit o focus al siguiente.
* **`blurOnSubmit`**: si debe perder foco al enviar. En formularios suele ser `false` si vas al siguiente input.

### Seguridad y formato

* **`secureTextEntry`**: oculta texto (password). Ojo con UX (toggle “ver contraseña”).
* **`autoCapitalize`**: (`none`, `sentences`, `words`, `characters`). Para emails: `none`.
* **`autoCorrect`**: autocorrección. Para emails/usuario: `false`.
* **`maxLength`**: límite de caracteres (OTP, códigos, etc.).

### Foco y eventos

* **`onFocus`**, **`onBlur`**: entrada/salida de foco. Útiles para validación.
* **`editable`**: false = deshabilitado (pero no siempre “se ve” deshabilitado, debes estilizar).
* **`selectTextOnFocus`**: selecciona todo al enfocar (muy útil en inputs numéricos).

### Multi-línea

* **`multiline`**: habilita varias líneas.
* **`numberOfLines`**: altura sugerida (Android influye más).
* **`textAlignVertical`**: importante en Android para que el texto empiece arriba.

### UX avanzada

* **`placeholder`** y **`placeholderTextColor`**
* **`selectionColor`** (color del cursor/selección)
* **`inputMode`** (cuando esté disponible en tu versión) para más control.

---

# Prop `style` en TextInput

## Rol de `style`

Define **cómo se ve** el input: borde, padding, color, tamaño, alineación, etc. En RN, el `TextInput` es una vista nativa con propiedades de texto, así que mezcla estilos de “View” y de “Text”.

### Estilos más comunes que sí funcionan

* **Layout**: `width`, `height`, `minHeight`, `flex`, `alignSelf`
* **Espaciado**: `padding`, `paddingHorizontal`, `margin`
* **Borde**: `borderWidth`, `borderColor`, `borderRadius`
* **Fondo**: `backgroundColor`
* **Texto**: `fontSize`, `fontWeight`, `color`, `fontFamily`
* **Alineación**: `textAlign` (texto), `textAlignVertical` (Android)

### Estilos que suelen confundir (o no aplican como esperas)

* `gap` no aplica dentro de un solo TextInput (gap es para contenedores).
* Sombras: en iOS `shadow*`, en Android `elevation` (y a veces no se ve bien en TextInput según wrapper).
* Algunas propiedades de texto avanzadas varían por plataforma.

### Diferencias iOS vs Android (relevantes)

* **Altura/padding**: Android puede “centrar raro” el texto; `textAlignVertical: 'top'` ayuda en multiline.
* **Autofill/autoComplete**: se comporta distinto; iOS suele requerir `textContentType`.
* **Underline en Android**: ciertos estilos o wrappers pueden mostrar subrayado por defecto en Android (depende del tema); normalmente se controla con `underlineColorAndroid="transparent"` (si lo necesitas).

---

# Errores típicos (al menos 8) + síntomas + solución

1. **No controlar el valor (`value` sin `onChangeText`)**

* Síntoma: no escribe, se “congela”.
* Solución: usar `value` + `onChangeText`.

2. **Usar `defaultValue` esperando que se actualice**

* Síntoma: cambias estado pero el input no refleja.
* Solución: pasar a modo controlado con `value`.

3. **Validar y mostrar error desde el primer render**

* Síntoma: el usuario ve errores antes de tocar el input.
* Solución: usar `touched` (como en el ejemplo).

4. **No ajustar `keyboardType`**

* Síntoma: usuario batalla para escribir números/email.
* Solución: `keyboardType="email-address"` / `numeric` / etc.

5. **Olvidar `autoCapitalize="none"` en emails**

* Síntoma: se capitaliza la primera letra del correo y falla login.
* Solución: `autoCapitalize="none"` y a veces `autoCorrect={false}`.

6. **`multiline` sin altura o sin `textAlignVertical` (Android)**

* Síntoma: el texto aparece centrado verticalmente o se ve raro.
* Solución: `multiline`, `minHeight`, y en Android `textAlignVertical: 'top'`.

7. **Renderizados excesivos por cada tecla**

* Síntoma: lag al escribir.
* Solución: memorizar componentes, evitar recalcular todo; validar con debounce; no disparar lógica pesada en `onChangeText`.

8. **Inputs dentro de `ScrollView` sin manejar teclado**

* Síntoma: el teclado tapa el input.
* Solución: `KeyboardAvoidingView`, o librerías como `react-native-keyboard-aware-scroll-view`.

9. **No manejar foco entre inputs**

* Síntoma: usuario tiene que tocar manualmente el siguiente campo.
* Solución: `returnKeyType="next"`, `onSubmitEditing` + `ref.focus()`.

---

# Componentes y herramientas auxiliares

### Componentes que van junto con TextInput

* **`KeyboardAvoidingView`**: evita que teclado tape campos.
* **`ScrollView`** / **`FlatList`**: para formularios largos (con cuidado).
* **`Pressable`**: para botones de “Enviar”, “Buscar”, “Mostrar contraseña”.
* **`Text`**: labels, errores, ayudas.
* **`View`**: contenedor y layout.

### Hooks / APIs relacionadas

* **`useState`**: para controlar el valor.
* **`useRef`**: para `focus()`, `blur()`, navegación entre inputs.
* **`Keyboard` API** (`import { Keyboard } from 'react-native'`): escuchar show/hide, cerrar teclado.
* **`InteractionManager`**: si necesitas posponer focus hasta que termine una transición.
* **`useMemo` / `useCallback`**: para evitar renders/recalculos caros en cada tecla.

### Librerías que lo complementan

* **React Hook Form** (`react-hook-form`): formularios performantes, control/validación.
* **Formik + Yup**: validación declarativa (más pesado que RHF, pero popular).
* **React Native Paper / UI Kitten / NativeBase**: inputs con estilos, iconos, helpers.
* **`react-native-keyboard-aware-scroll-view`**: scroll inteligente con teclado.
* **Zod/Yup**: esquemas de validación.

---

# Buenas prácticas (proyectos reales)

### 1) Prefiere inputs controlados

* `value` + `onChangeText`
* Más predecible, fácil de validar.

### 2) Optimiza validación

* No valides pesado en cada tecla.
* Usa `debounce` en búsqueda.
* Usa `touched` para UX.

### 3) Aprovecha el teclado correcto

* Email: `keyboardType="email-address"`, `autoCapitalize="none"`, `autoCorrect={false}`
* Tel: `keyboardType="phone-pad"`
* Numérico: `keyboardType="numeric"` o `number-pad` según plataforma.

### 4) Maneja foco y submit

* `returnKeyType="next"` y `onSubmitEditing` para flujo natural.

### 5) Cuida el teclado en pantallas con scroll

* `KeyboardAvoidingView` (iOS) y/o keyboard-aware scroll.

### 6) Separación de estilos

* Centraliza estilos en `StyleSheet.create` (más orden, potencialmente más eficiente).
* Evita objetos inline gigantes en renders muy frecuentes (como inputs en lista).

### 7) Rendimiento

* Evita que el padre del input re-renderice por cosas no relacionadas.
* Divide en componentes y usa `React.memo` si hace falta.
* Si tienes muchos inputs, considera arquitectura de formulario (RHF).

---

Si me dices tu caso (por ejemplo: **formulario de configuración en Expo Router**, o **búsqueda en una lista**, o **capturar números / preguntas Q01..Q63**), te armo un ejemplo “a tu estilo” con:

* validación,
* teclado ideal,
* scroll + teclado,
* y un patrón de componentes reutilizables (`InputField` + `ErrorText`).
