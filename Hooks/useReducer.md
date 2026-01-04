# `useReducer` en React / React Native

## 1) Definición clara

### ¿Qué es este Hook?

`useReducer` es un Hook de React para manejar **estado complejo** mediante un patrón de **reductor** (reducer), similar a Redux pero **local al componente**.

En una frase:

> `useReducer` te permite actualizar estado describiendo *qué pasó* (acción) y una función pura decide *cómo cambia* el estado.

La idea central:

* Tú no “setéas” campos sueltos por todos lados
* Tú **despachas acciones** (`dispatch({ type: ... })`)
* Un **reducer** calcula el nuevo estado de forma predecible

---

### ¿Qué problema resuelve dentro del modelo de componentes funcionales?

Cuando usas `useState` para cosas complejas, aparecen síntomas:

* Tienes muchos `useState` relacionados
* Actualizar algo requiere tocar varios setters
* El flujo de cambios se vuelve difícil de seguir
* Empiezan bugs por estados desincronizados

Ejemplo típico con `useState` (mala señal):

* `email`, `password`, `error`, `loading`, `touchedEmail`, `touchedPassword`, `isValid`, `attempts`, etc.
* Reglas de negocio que cruzan campos

`useReducer` resuelve esto:

* Centraliza la lógica de actualización en un solo lugar
* Hace explícitas las transiciones de estado
* Mejora legibilidad y testeo (reducer es función pura)

---

### ¿Por qué existe este Hook y qué limitación del enfoque anterior soluciona?

En clases y en Redux se usaba el patrón:

* estado + acciones + reducer

En funciones, `useState` es perfecto para lo simple, pero se vuelve incómodo para:

* transiciones complejas
* múltiples campos acoplados
* reglas de validación
* estados tipo “máquina de estados”

`useReducer` existe para:

* aportar un modelo más estructurado
* reemplazar “spaghetti de setters”
* facilitar escalabilidad sin meter Redux necesariamente

---

### ¿En qué casos se debe usar y en cuáles NO?

**Úsalo cuando:**

* El estado tiene **múltiples campos relacionados**
* Una acción cambia **varias partes del estado**
* La lógica de actualización es compleja o repetitiva
* Quieres que el estado sea **predecible** y fácil de testear
* Modelas “modos” o “etapas” (idle/loading/success/error)

Casos típicos en RN:

* Formularios grandes con validación
* Flujos de onboarding / wizard
* Carrito de compras local
* Manejo de requests: `idle → loading → success/error`
* Editor con undo/redo (local)

**No lo uses cuando:**

* Tienes 1–2 valores simples (mejor `useState`)
* La lógica no se complica
* Estás haciendo un “mini-Redux” sin necesidad
* Lo usas solo por estilo

> Regla rápida:
> **Si cada cambio del estado es “setear un valor”, `useState`.**
> **Si cada cambio es “aplicar una transición”, `useReducer`.**

---

### Relación con el ciclo de renderizado y la reconciliación de React

* `dispatch(action)` agenda una actualización.
* React ejecuta el reducer con:

  * estado actual
  * acción
* El reducer devuelve un **nuevo estado**
* React re-renderiza el componente con ese nuevo estado
* Reconciliación actualiza la UI

Puntos importantes:

* El reducer debe ser **puro** (sin efectos secundarios)
* Si devuelves el **mismo objeto** (misma referencia), React puede evitar renders innecesarios
* Si siempre creas un objeto nuevo aunque nada cambie, generas renders extra

---

## 2) Ejemplo mínimo funcional

```jsx
import React, { useReducer } from "react";
import { View, Text, Pressable } from "react-native";

function reducer(state, action) {
  switch (action.type) {
    case "incrementar":
      return { count: state.count + 1 };
    case "decrementar":
      return { count: state.count - 1 };
    default:
      return state;
  }
}

export default function App() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <View style={{ padding: 20, gap: 12 }}>
      <Text>Contador: {state.count}</Text>

      <Pressable onPress={() => dispatch({ type: "incrementar" })}>
        <Text>+1</Text>
      </Pressable>

      <Pressable onPress={() => dispatch({ type: "decrementar" })}>
        <Text>-1</Text>
      </Pressable>
    </View>
  );
}
```

### Explicación breve

* `reducer(state, action)`:

  * define cómo cambia el estado según la acción
* `useReducer(reducer, { count: 0 })`:

  * crea el estado inicial y retorna:

    * `state`: estado actual
    * `dispatch`: función para enviar acciones

### ¿Qué ocurre internamente cuando el Hook se ejecuta?

* React guarda el estado inicial la primera vez
* Cuando haces `dispatch(action)`:

  * React llama a `reducer(estadoActual, action)`
  * guarda el resultado como nuevo estado
  * re-renderiza

---

## 3) Ejemplo complejo y realista

Caso real: **formulario de login con validación + request async + estados de UI**
(este tipo de caso se vuelve “spaghetti” con muchos `useState`)

```jsx
import React, { useEffect, useReducer } from "react";
import { View, Text, TextInput, Pressable } from "react-native";

const initialState = {
  email: "",
  password: "",
  touched: { email: false, password: false },
  status: "idle", // idle | loading | success | error
  error: null,
};

function validateEmail(email) {
  return email.includes("@");
}

function reducer(state, action) {
  switch (action.type) {
    case "set_email": {
      const email = action.value;
      return { ...state, email };
    }
    case "set_password": {
      const password = action.value;
      return { ...state, password };
    }
    case "touch": {
      return {
        ...state,
        touched: { ...state.touched, [action.field]: true },
      };
    }
    case "submit_start": {
      return { ...state, status: "loading", error: null };
    }
    case "submit_success": {
      return { ...state, status: "success", error: null };
    }
    case "submit_error": {
      return { ...state, status: "error", error: action.message };
    }
    case "reset": {
      return initialState;
    }
    default:
      return state;
  }
}

export default function LoginForm({ onSubmit }) {
  const [state, dispatch] = useReducer(reducer, initialState);

  const emailValido = validateEmail(state.email);
  const passwordValido = state.password.length >= 6;
  const puedeEnviar = emailValido && passwordValido && state.status !== "loading";

  const mostrarErrorEmail = state.touched.email && !emailValido;
  const mostrarErrorPassword = state.touched.password && !passwordValido;

  const handleSubmit = async () => {
    if (!puedeEnviar) {
      dispatch({ type: "touch", field: "email" });
      dispatch({ type: "touch", field: "password" });
      return;
    }

    dispatch({ type: "submit_start" });

    try {
      await onSubmit({ email: state.email, password: state.password });
      dispatch({ type: "submit_success" });
    } catch (e) {
      dispatch({ type: "submit_error", message: "Credenciales inválidas" });
    }
  };

  useEffect(() => {
    // Ejemplo de efecto: si éxito, podrías navegar o limpiar el form
    // Aquí solo demostramos que el estado “success” es una etapa del flujo.
  }, [state.status]);

  return (
    <View style={{ padding: 20, gap: 10 }}>
      <Text style={{ fontSize: 18, fontWeight: "700" }}>Login</Text>

      <TextInput
        value={state.email}
        onChangeText={(v) => dispatch({ type: "set_email", value: v })}
        onBlur={() => dispatch({ type: "touch", field: "email" })}
        placeholder="Correo"
        style={{ borderWidth: 1, padding: 10, borderRadius: 10 }}
        autoCapitalize="none"
        keyboardType="email-address"
      />
      {mostrarErrorEmail && (
        <Text style={{ color: "red" }}>Correo inválido</Text>
      )}

      <TextInput
        value={state.password}
        onChangeText={(v) => dispatch({ type: "set_password", value: v })}
        onBlur={() => dispatch({ type: "touch", field: "password" })}
        placeholder="Contraseña"
        secureTextEntry
        style={{ borderWidth: 1, padding: 10, borderRadius: 10 }}
      />
      {mostrarErrorPassword && (
        <Text style={{ color: "red" }}>Mínimo 6 caracteres</Text>
      )}

      {state.status === "error" && (
        <Text style={{ color: "red" }}>{state.error}</Text>
      )}

      <Pressable
        onPress={handleSubmit}
        disabled={!puedeEnviar}
        style={{
          padding: 12,
          borderWidth: 1,
          borderRadius: 10,
          opacity: puedeEnviar ? 1 : 0.5,
        }}
      >
        <Text>{state.status === "loading" ? "Enviando..." : "Entrar"}</Text>
      </Pressable>

      {state.status === "success" && <Text>¡Listo!</Text>}

      <Pressable onPress={() => dispatch({ type: "reset" })}>
        <Text>Reset</Text>
      </Pressable>
    </View>
  );
}
```

### Qué demuestra este ejemplo

* Estado con múltiples campos relacionados
* Transiciones claras por acción (`submit_start`, `submit_success`, `submit_error`)
* Validación derivada (no duplicada en estado)
* Interacción con eventos (`onChangeText`, `onBlur`, `onPress`)
* Integración con efecto (`useEffect` escuchando `status`)
* Flujo cercano a producción: idle → loading → success/error

---

## 4) API del Hook

### Firma

```ts
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

### Parámetros de entrada

* `reducer: (state, action) => newState`

  * función pura que calcula el nuevo estado
* `initialArg`

  * estado inicial, o argumento inicial para `init`
* `init?: (initialArg) => initialState`

  * inicializador perezoso (lazy init)

Ejemplo de lazy init (útil si el estado inicial cuesta):

```jsx
const [state, dispatch] = useReducer(reducer, initialArg, (arg) => {
  return construirEstadoInicial(arg);
});
```

### Valores que retorna

* `state`: estado actual
* `dispatch(action)`: función para enviar acciones

### Qué pasa si se usan valores incorrectos

* Reducer con side-effects → comportamiento impredecible
* Mutar el estado en el reducer → bugs por referencias
* Acciones sin `type` o inconsistentes → difícil de mantener

---

## 5) Errores típicos (mínimo 8)

1. **Reducer no puro (side-effects dentro)**

* Síntoma: llamadas duplicadas, bugs raros
* Por qué: React puede renderizar más de una vez en dev
* Solución: efectos en `useEffect`, reducer puro

2. **Mutar el estado**

* Síntoma: UI no se actualiza o se rompe la lógica
* Por qué: React depende de nuevas referencias
* Solución: retornar nuevos objetos/arrays (inmutabilidad)

3. **Acciones sin contrato**

* Síntoma: switch enorme e incoherente
* Solución: definir tipos de acción claros y consistentes

4. **Guardar valores derivados en el estado**

* Síntoma: inconsistencias
* Solución: derivar en render (`const puedeEnviar = ...`)

5. **Usar `useReducer` para algo trivial**

* Síntoma: boilerplate excesivo
* Solución: `useState` es suficiente

6. **Reducer demasiado grande (“Dios”)**

* Síntoma: difícil de mantener
* Solución: dividir por dominios o extraer lógica

7. **No manejar acción por defecto**

* Síntoma: acciones desconocidas no hacen nada y nadie lo nota
* Solución: default con log o throw en dev

8. **Acciones que actualizan múltiples cosas sin intención clara**

* Síntoma: difícil de depurar
* Solución: acciones semánticas (“submit_start”, “field_change”)

---

## 6) Hooks, APIs y herramientas relacionadas

Hooks relacionados:

* `useState` (para estados simples)
* `useEffect` (para efectos derivados de estados/transiciones)
* `useContext` (para compartir state/dispatch globalmente en un subárbol)
* `useMemo` (derivar datos costosos del estado)
* `useCallback` (handlers estables que dispatchan)
* `useRef` (timers, flags auxiliares)

APIs relacionadas:

* Patrón Redux (acciones/reducer)
* `React.memo` (optimización)
* En RN: `TextInput`, `Pressable`, `FlatList` (eventos y UI)

Casos donde conviene combinarlos:

* `useReducer + useContext` para “mini-store” por módulo (auth, settings)
* `useReducer + useEffect` para flujos async (loading/success/error)
* `useReducer + useCallback` para handlers estables

---

## 7) Buenas prácticas

### Reglas oficiales del Hook

* Hooks no condicionales
* Reducer puro
* Mantener dependencia clara de acciones

### Patrones recomendados

* Estado por “etapas”: `idle/loading/success/error`
* Acciones semánticas (describen eventos, no “setters”)
* Validaciones derivadas en render (no duplicar estado)
* Lazy init si el estado inicial es pesado

### Antipatrones comunes

* Imitar Redux completo dentro de un componente sin necesidad
* Reducers gigantes sin modularidad
* Meter fetch dentro del reducer

### Consideraciones de rendimiento

* `dispatch` dispara re-render como `setState`
* Reducers grandes pueden volverse costosos si hacen cálculos pesados:

  * mover cálculos a `useMemo`
  * evitar recomputar sin necesidad
* Mantener retornos iguales si nada cambia (optimización fina)

### Código legible y predecible

* Definir `initialState` fuera del componente
* Acciones con nombres claros
* Reducer en archivo aparte si crece
* Mantener el reducer fácil de testear (función pura)

---

## 8) Cuándo NO usar este Hook

* Estado simple con 1–2 variables
* Interacción sencilla (toggle, contador)
* Cuando la complejidad es artificial

Alternativas:

* `useState`
* Extraer componentes
* `useContext` + store externo si el estado es realmente global
* Zustand/Redux si necesitas selectores, middlewares, escalabilidad global

---

### Modelo mental final (clave)

> `useState` es para “guardar valores”.
> `useReducer` es para “modelar transiciones”.

Si quieres, el siguiente paso natural es:

* **`useReducer + useContext`** para armar un “store por módulo” (auth/settings) bien hecho en React Native, sin Redux, y con buenas prácticas de rendimiento.
