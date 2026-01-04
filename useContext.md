# `useContext` en React / React Native

## 1) Definición clara

### ¿Qué es este Hook?

`useContext` es un Hook de React que permite **leer (consumir) el valor de un Contexto** dentro de un componente funcional.

Un Contexto es un mecanismo para **inyectar un valor a un subárbol de componentes** sin pasar ese valor como props a través de cada nivel.

En una frase:

> `useContext` te permite acceder a un valor global “del subárbol” sin prop drilling.

---

### ¿Qué problema resuelve dentro del modelo de componentes funcionales?

Resuelve el problema de **prop drilling** (pasar props por muchos niveles):

* Componentes intermedios reciben props que no usan
* La estructura del árbol condiciona el flujo de datos
* Cambiar el árbol rompe rutas de props

Con Context:

* El valor viaja “por debajo” del árbol
* Los consumidores lo leen directamente donde lo necesitan

---

### ¿Por qué existe este Hook y qué limitación del enfoque anterior soluciona?

Antes de Hooks, consumir Context era más verboso:

* `MyContext.Consumer` (render prop)
* Anidaciones de funciones
* Menor legibilidad

`useContext` existe para:

* Consumir Context de forma directa en funciones
* Integrarse al modelo de Hooks
* Facilitar encapsulación con Hooks personalizados (`useAuth`, `useTheme`)

---

### ¿En qué casos se debe usar y en cuáles NO?

**Úsalo cuando:**

* Varios componentes (en diferentes niveles) necesitan el mismo dato
* El dato es “global” para una parte de la app
* El dato cambia con baja o media frecuencia

Casos típicos:

* Tema (dark/light)
* Usuario autenticado y permisos
* Idioma
* Feature flags
* Configuración global de UI

**No lo uses cuando:**

* Solo un hijo directo necesita el dato (pasa props y ya)
* El valor cambia constantemente (ej. animaciones, scroll, coordenadas en tiempo real)
* Quieres un store global complejo (acciones, historial, selectores finos)
* Estás usando Context para “evitar pensar” la arquitectura

> Regla importante:
> **Context no es un manejador de estado; es un canal de distribución.**
> El estado puede vivir dentro del Provider, pero eso es otra cosa (`useState`/`useReducer`).

---

### Relación del Hook con el ciclo de renderizado y la reconciliación de React

* Cuando cambia el `value` de un `<Context.Provider>`, React **notifica a todos los consumidores**.
* Todo componente que haga `useContext(EseContexto)` se vuelve a renderizar cuando el valor cambia.
* La comparación es por referencia (si `value` es un objeto nuevo, cuenta como cambio).

Consecuencia práctica:

* Si tu Provider hace `value={{...}}` en cada render sin memoizar, disparas renders en cascada.

---

## 2) Ejemplo mínimo funcional

```jsx
import React, { createContext, useContext } from "react";
import { View, Text } from "react-native";

const TemaContext = createContext("light");

export default function App() {
  return (
    <TemaContext.Provider value="dark">
      <Pantalla />
    </TemaContext.Provider>
  );
}

function Pantalla() {
  const tema = useContext(TemaContext);
  return (
    <View>
      <Text>Tema actual: {tema}</Text>
    </View>
  );
}
```

### Explicación breve

* `createContext("light")`: define un Contexto con valor por defecto (“fallback”).
* `<TemaContext.Provider value="dark">`: provee “dark” a todo el subárbol.
* `useContext(TemaContext)`: lee el valor del Provider más cercano.

### ¿Qué ocurre internamente cuando el Hook se ejecuta?

* React registra que `Pantalla` **depende** de `TemaContext`.
* Si el Provider cambia su valor, React:

  * agenda re-render de `Pantalla` y cualquier otro consumidor.
* Si no hay Provider, `useContext` devuelve el valor por defecto del Contexto.

---

## 3) Ejemplo complejo y realista

Caso real: **Auth + estado + eventos + Hook personalizado**
(esto es lo típico en RN: toda la app necesita saber si hay usuario y cómo cerrar sesión)

```jsx
import React, { createContext, useContext, useMemo, useState } from "react";
import { View, Text, Pressable } from "react-native";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [usuario, setUsuario] = useState(null);

  const login = (nombre) => setUsuario({ nombre });
  const logout = () => setUsuario(null);

  // IMPORTANTÍSIMO: memoizar value para no re-renderizar consumidores por referencia nueva
  const value = useMemo(() => {
    return { usuario, login, logout };
  }, [usuario]);

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) {
    throw new Error("useAuth debe usarse dentro de <AuthProvider>");
  }
  return ctx;
}

// ----------------- Consumidores -----------------

function Header() {
  const { usuario } = useAuth();
  return (
    <View style={{ padding: 12 }}>
      <Text style={{ fontWeight: "700" }}>
        {usuario ? `Hola, ${usuario.nombre}` : "Bienvenido"}
      </Text>
    </View>
  );
}

function BotonLogin() {
  const { login } = useAuth();
  return (
    <Pressable onPress={() => login("Fausto")}>
      <Text>Iniciar sesión</Text>
    </Pressable>
  );
}

function BotonLogout() {
  const { logout } = useAuth();
  return (
    <Pressable onPress={logout}>
      <Text>Cerrar sesión</Text>
    </Pressable>
  );
}

export default function App() {
  return (
    <AuthProvider>
      <Header />
      <BotonLogin />
      <BotonLogout />
    </AuthProvider>
  );
}
```

### Qué demuestra este ejemplo

* Context como canal de distribución: `usuario`, `login`, `logout`.
* Estado dentro del Provider: `useState`.
* Eventos reales: botones que disparan login/logout.
* Hook personalizado (`useAuth`) para:

  * mejorar ergonomía
  * validar uso dentro del Provider
* `useMemo` para estabilizar `value` (evitar renders innecesarios).

---

## 4) API del Hook

### Firma

```ts
const value = useContext(Context);
```

### Parámetros de entrada

* `Context`: el objeto retornado por `createContext(...)`.
* Debe ser **la misma instancia** (misma referencia) creada por `createContext`.

### Valor(es) que retorna

* Retorna el valor del Provider más cercano.
* Si no hay Provider arriba: retorna el valor por defecto del Context.

### Qué pasa si se usan valores incorrectos

* Si consumes fuera del Provider:

  * obtendrás el default (y podrías no darte cuenta → bug silencioso)
  * por eso se recomienda lanzar error en hooks tipo `useAuth`.
* Si el Provider crea un objeto nuevo en cada render:

  * re-renders masivos aunque no cambie “lo importante”.

---

## 5) Errores típicos (mínimo 8)

1. **Usar Context como store global para todo**

* Síntoma: re-renders por todos lados
* Por qué: todo consumidor se re-renderiza cuando cambia el value
* Solución: dividir contextos o usar store con selectores (Zustand/Redux)

2. **No memoizar `value` del Provider**

* Síntoma: renders constantes de consumidores
* Por qué: `value={{...}}` crea objeto nuevo cada render
* Solución: `useMemo` para `value`

3. **Provider colocado demasiado arriba**

* Síntoma: cambios pequeños re-renderizan grandes secciones
* Solución: mover Provider más cerca de donde se necesita

4. **Context “Dios” con demasiadas responsabilidades**

* Síntoma: difícil de mantener, acoplamiento
* Solución: varios contextos (Auth, Theme, Settings…)

5. **Consumir contexto fuera del Provider sin darte cuenta**

* Síntoma: `null`, valores default, fallos raros
* Solución: hook seguro que valide y lance error

6. **Mutar el objeto del contexto**

* Síntoma: UI no reacciona correctamente o bugs intermitentes
* Por qué: mutación no cambia referencia; React puede no re-renderizar como esperas
* Solución: inmutabilidad (crear nuevos objetos)

7. **Meter datos de alta frecuencia en Context**

* Síntoma: la app se vuelve lenta
* Por qué: demasiados renders por segundo
* Solución: `useRef`, stores con selectores, o arquitectura distinta

8. **Usar Context para evitar pasar props cuando no es necesario**

* Síntoma: complejidad extra sin beneficio
* Solución: props directas en árboles pequeños

---

## 6) Hooks, APIs y herramientas relacionadas

Hooks que suelen usarse con `useContext`:

* `useState` (estado dentro del Provider)
* `useReducer` (estado complejo dentro del Provider)
* `useMemo` (memoizar `value`)
* `useCallback` (estabilizar funciones expuestas en `value`)
* Hooks personalizados (`useAuth`, `useTheme`)

APIs relacionadas:

* `createContext`
* `<Context.Provider>`

Casos donde conviene combinarlos:

* AuthProvider: `useState` + `useMemo` + `useContext`
* ThemeProvider: `useReducer` + `useContext`
* SettingsProvider: persistencia con `useEffect` + `useContext`

---

## 7) Buenas prácticas

### Reglas oficiales del Hook (y por qué existen)

* No llamar `useContext` condicionalmente
* Mantener el orden de Hooks estable
* Encapsular “contratos globales” de app en Providers claros

### Patrones recomendados

* **Provider + Hook personalizado**:

  * `AuthProvider` + `useAuth`
  * `ThemeProvider` + `useTheme`
* Separar contextos por dominio y frecuencia de cambio
* Memoizar `value` y, si es necesario, callbacks

Ejemplo recomendado:

```jsx
const value = useMemo(() => ({ usuario, login, logout }), [usuario, login, logout]);
```

(En muchos casos `login/logout` son estables, pero el patrón es válido.)

### Antipatrones comunes

* `GlobalContext` con todo el estado de la app
* Providers anidados sin criterio
* Context para reemplazar arquitectura de estado

### Consideraciones de rendimiento

* Cada cambio del Provider re-renderiza consumidores
* Minimiza cambios de referencia
* Divide contextos: uno para “tema”, otro para “usuario”, etc.

### Cómo mantener el código legible y predecible

* Nombres claros (`AuthContext`, `ThemeContext`)
* Hook de acceso (`useAuth`)
* Valores del contexto con contrato estable (no cambiar estructura sin razón)

---

## 8) Cuándo NO usar este Hook

* Si pasar props es simple y directo
* Si el dato cambia muy seguido
* Si necesitas selectores finos o “partial subscriptions”
* Si requieres herramientas avanzadas de debugging/arquitectura

Alternativas:

* Pasar props
* `useReducer` local
* Stores externos con selectores (Zustand/Redux)
* `useRef` para valores que no deben re-renderizar

---

### Modelo mental final (clave)

> **Props**: fluyen padre → hijo por estructura
> **Context**: inyecta un valor a un subárbol
> **`useContext`**: lee ese valor y re-renderiza cuando cambia

Si quieres, el siguiente paso natural es **`useCallback`** (porque es el que más se usa para estabilizar funciones que terminas pasando por Context o a listas), con la misma plantilla.
