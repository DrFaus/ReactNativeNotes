# `useRef` en React / React Native

## 1) Definición clara

### ¿Qué es este Hook?

`useRef` es un Hook de React que crea un **objeto estable** con una propiedad `.current`, cuyo valor **persiste entre renders**.

En una frase:

> `useRef` te da una “cajita” (`{ current: ... }`) que React conserva entre renders, y actualizarla **no dispara re-render**.

`useRef` tiene **dos usos principales**:

1. **Referenciar** elementos/instancias (ej. un `TextInput` para hacer `focus()`).
2. Guardar **valores mutables** que quieres recordar, pero que **no deben causar render** (ids de timers, flags, valores previos, etc.).

---

### ¿Qué problema resuelve dentro del modelo de componentes funcionales?

En componentes funcionales:

* Las variables locales se reinician en cada render.
* El estado (`useState`) provoca re-render al cambiar.
* A veces necesitas guardar algo **entre renders** pero **sin re-renderizar**.

Ejemplos típicos:

* Guardar `setInterval`/`setTimeout` id
* Guardar la última query enviada (para evitar carreras)
* Guardar un “flag” `isMounted`
* Guardar el valor previo de una prop/estado
* Guardar una referencia a un input o componente nativo

`useRef` resuelve exactamente eso.

---

### ¿Por qué existe este Hook y qué limitación del enfoque anterior soluciona?

En clases existían:

* Variables de instancia: `this.algo = ...`
* Referencias con `createRef()`

En funciones:

* No hay `this`
* No hay instancia persistente “natural”
* Todo se re-ejecuta en cada render

`useRef` reemplaza esa “memoria de instancia” de forma explícita y segura.

---

### ¿En qué casos se debe usar y en cuáles NO?

**Úsalo cuando:**

* Necesitas mantener un valor entre renders **sin re-render**
* Necesitas acceder a un elemento/instancia (focus, measure, scrollTo…)
* Quieres almacenar ids de timers
* Quieres leer “el valor más reciente” dentro de callbacks/efectos sin re-suscribirte
* Necesitas guardar el valor anterior (prev state/prev props)

**No lo uses cuando:**

* El valor debe mostrarse en UI y actualizarse (usa `useState`)
* Estás intentando “evadir” el flujo reactivo (puede volverse impredecible)
* Estás guardando estado de negocio que otros componentes necesitan (context/store)
* Estás mutando datos complejos que deberían ser inmutables (puede causar bugs)

> Regla clave:
> **Si el cambio debe reflejarse en pantalla → `useState`.**
> **Si solo necesitas recordar algo internamente → `useRef`.**

---

### Relación del Hook con el ciclo de renderizado y la reconciliación de React

* `useRef` crea un objeto **estable**: la referencia del objeto no cambia entre renders.
* Cambiar `ref.current` **no provoca render**.
* React no “observa” `.current` para reconciliar UI.

Consecuencia:

* Es perfecto para datos internos (timers, flags).
* Es peligroso si intentas usarlo como “estado invisible” que afecta la UI, porque la UI no se actualizará.

---

## 2) Ejemplo mínimo funcional

```jsx
import React, { useRef } from "react";
import { View, TextInput, Pressable, Text } from "react-native";

export default function App() {
  const inputRef = useRef(null);

  const enfocar = () => {
    inputRef.current?.focus();
  };

  return (
    <View style={{ padding: 20, gap: 12 }}>
      <TextInput
        ref={inputRef}
        placeholder="Escribe algo…"
        style={{ borderWidth: 1, padding: 10, borderRadius: 10 }}
      />

      <Pressable onPress={enfocar}>
        <Text>Enfocar input</Text>
      </Pressable>
    </View>
  );
}
```

### Explicación breve

* `const inputRef = useRef(null)` crea un ref.
* `ref={inputRef}` conecta el ref al `TextInput`.
* `inputRef.current` apunta a la instancia nativa del input.
* `focus()` se llama sin necesidad de re-render.

### ¿Qué ocurre internamente cuando el Hook se ejecuta?

* En el primer render React crea `{ current: null }`.
* En renders siguientes React devuelve **el mismo objeto**.
* Al montar el `TextInput`, React asigna la instancia a `inputRef.current`.

---

## 3) Ejemplo complejo y realista

Caso real: **búsqueda con debounce + evitar respuestas viejas**
Aquí `useRef` guarda:

* el id del timeout
* un “token” incremental para descartar respuestas antiguas

```jsx
import React, { useEffect, useRef, useState } from "react";
import { View, Text, TextInput, FlatList } from "react-native";

export default function BuscadorSeguro({ endpointBase }) {
  const [query, setQuery] = useState("");
  const [resultados, setResultados] = useState([]);
  const [loading, setLoading] = useState(false);

  const timeoutRef = useRef(null);
  const requestIdRef = useRef(0);

  useEffect(() => {
    // Limpieza del timeout previo (debounce)
    if (timeoutRef.current) clearTimeout(timeoutRef.current);

    const q = query.trim();
    if (q.length === 0) {
      setResultados([]);
      setLoading(false);
      return;
    }

    timeoutRef.current = setTimeout(async () => {
      // “token” para esta solicitud
      const myRequestId = ++requestIdRef.current;

      setLoading(true);

      try {
        const res = await fetch(
          `${endpointBase}?q=${encodeURIComponent(q)}`
        );
        const data = await res.json();

        // Si mientras tanto hubo otra request más nueva, ignora esta
        if (myRequestId !== requestIdRef.current) return;

        setResultados(Array.isArray(data) ? data : []);
      } finally {
        // Solo la request más reciente apaga loading
        if (myRequestId === requestIdRef.current) {
          setLoading(false);
        }
      }
    }, 400);

    return () => {
      if (timeoutRef.current) clearTimeout(timeoutRef.current);
    };
  }, [endpointBase, query]);

  return (
    <View style={{ padding: 16, gap: 12 }}>
      <Text style={{ fontSize: 18, fontWeight: "700" }}>Búsqueda segura</Text>

      <TextInput
        value={query}
        onChangeText={setQuery}
        placeholder="Busca…"
        style={{ borderWidth: 1, padding: 10, borderRadius: 10 }}
      />

      {loading && <Text>Cargando…</Text>}

      <FlatList
        data={resultados}
        keyExtractor={(item, index) => String(item.id ?? index)}
        renderItem={({ item }) => (
          <Text style={{ paddingVertical: 6 }}>
            {item.title ?? "Sin título"}
          </Text>
        )}
      />
    </View>
  );
}
```

### Qué demuestra este ejemplo

* `useEffect` coordina el efecto (fetch + debounce).
* `useRef` guarda datos persistentes **sin renders**:

  * `timeoutRef.current`
  * `requestIdRef.current`
* Se evitan bugs típicos:

  * respuestas “viejas” pisando las nuevas
  * timeouts acumulados

---

## 4) API del Hook

### Firma

```ts
const ref = useRef(initialValue);
```

### Parámetros de entrada

* `initialValue`: valor inicial de `ref.current` (solo se usa al primer render)

### Valor que retorna

* Retorna un objeto estable:

```ts
{ current: initialValue }
```

### Significado de cada parámetro y retorno

* `ref.current` es el valor mutable persistente
* `ref` (el objeto) mantiene identidad estable

### Qué pasa si se usan valores incorrectos

* Si esperas que cambiar `.current` re-renderice → la UI no se actualiza (bug)
* Si lo usas como “estado oculto” → comportamiento impredecible
* Si guardas cosas mutables complejas y las mutas → puede generar inconsistencias difíciles

---

## 5) Errores típicos (mínimo 8)

1. **Esperar re-render al cambiar `ref.current`**

* Síntoma: UI no cambia
* Por qué: React no observa `.current`
* Solución: usa `useState` si debe verse en UI

2. **Usar `useRef` como estado para UI**

* Síntoma: datos desincronizados
* Solución: mover a `useState` / `useReducer`

3. **No inicializar correctamente**

* Síntoma: `ref.current` es `null` cuando lo usas
* Solución: usar optional chaining o validar antes de usar

4. **Usar `ref.current` en render como si fuera estable**

* Síntoma: valores cambian “por detrás” sin render
* Solución: si quieres mostrarlo, muévelo a estado

5. **Guardar objetos y mutarlos**

* Síntoma: lógica inconsistente
* Por qué: mutación silenciosa
* Solución: inmutabilidad o estado real

6. **Olvidar limpiar timers/listeners guardados en refs**

* Síntoma: fugas, eventos duplicados
* Solución: cleanup en `useEffect`

7. **Confundir `useRef` con `createRef`**

* Síntoma: ref nuevo cada render (si usas mal createRef en funciones)
* Solución: en funciones, usa `useRef`

8. **Guardar demasiada lógica en refs**

* Síntoma: código opaco y difícil de depurar
* Solución: refs para cosas puntuales; estado para lo demás

---

## 6) Hooks, APIs y herramientas relacionadas

Hooks frecuentes junto con `useRef`:

* `useEffect` (timers, listeners, cancelaciones, valores “latest”)
* `useCallback` (callbacks que leen refs sin cambiar dependencias)
* `useState` (UI reactiva; ref para auxiliares)
* `useMemo` (derivados; ref para cache manual si es necesario)
* `useLayoutEffect` (cuando necesitas medir/layout antes del pintado, casos específicos)

APIs RN relacionadas:

* `TextInput` (focus/blur)
* `FlatList` (`scrollToIndex`, `scrollToOffset`)
* `Animated` / `Reanimated` (algunas integraciones usan refs)
* `Keyboard` / `AppState` (listeners)

Casos donde conviene combinarlos:

* `useEffect + useRef` para cancelación/cleanup
* `useCallback + useRef` para handlers que necesitan el valor “más reciente” sin re-render

---

## 7) Buenas prácticas

### Reglas oficiales del Hook (y por qué existen)

* Llamar Hooks siempre en el mismo orden (no condicionales)
* Tratar el render como función pura; `useRef` permite guardar cosas “fuera del render”

### Patrones recomendados

* **Refs para infraestructura**, estado para UI:

  * ref: timerId, requestId, isMounted, inputRef
  * estado: loading, datos, errores
* “Latest value pattern”:

  * guardar el último valor en ref para usarlo en callbacks sin re-suscribirte

Ejemplo rápido:

```jsx
const latestQuery = useRef(query);
useEffect(() => { latestQuery.current = query; }, [query]);
```

### Antipatrones comunes

* Reemplazar `useState` por `useRef` “para que no renderice”
* Mutaciones complejas dentro de `ref.current`
* Abusar de refs para lógica de negocio

### Consideraciones de rendimiento

* `useRef` es barato y no dispara renders
* Pero su mal uso puede causar “estado invisible” y bugs más caros que cualquier render

### Mantener código legible y predecible

* Nombra refs con intención: `timeoutRef`, `inputRef`, `requestIdRef`
* Documenta por qué existe el ref
* Usa refs para 1 propósito claro

---

## 8) Cuándo NO usar este Hook

* Cuando el valor afecta lo que se renderiza → `useState` / `useReducer`
* Para compartir datos globales → `useContext` / store
* Para cálculos derivados → `useMemo`
* Para optimizar renders sin evidencia → revisa arquitectura primero

Alternativas más adecuadas:

* `useState` (UI reactiva)
* `useReducer` (estado complejo)
* `useContext` (compartir)
* `useMemo` / `useCallback` (memoización)

---

### Modelo mental final (clave)

> `useState` = cambios visibles → re-render
> `useRef` = memoria interna → **no** re-render

Si quieres, el siguiente paso natural es **`useReducer`** (cuando tienes varios `useState` relacionados) o una guía de “cuándo usar cuál” con ejemplos tipo formulario, lista, navegación y timers.
