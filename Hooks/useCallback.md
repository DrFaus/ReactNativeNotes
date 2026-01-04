# `useCallback` en React / React Native

## 1) Definición clara

### ¿Qué es este Hook?

`useCallback` es un Hook de React que **memoriza una función** y devuelve **la misma referencia de esa función** mientras sus dependencias no cambien.

En una frase:

> `useCallback` sirve para evitar que una función se vuelva a crear en cada render.

No memoriza el *resultado* de la función (eso es `useMemo`), sino **la función en sí**.

---

### ¿Qué problema resuelve dentro del modelo de componentes funcionales?

En React, **cada render vuelve a ejecutar el componente**.
Eso significa que **todas las funciones declaradas dentro del componente se recrean**:

```jsx
function MiComponente() {
  const manejarClick = () => {
    console.log("click");
  };
}
```

Aunque el código “sea igual”, **la referencia de `manejarClick` es nueva** en cada render.

Esto causa problemas cuando:

* Pasas funciones como props a componentes hijos
* Usas componentes memoizados (`React.memo`)
* Usas funciones en dependencias de `useEffect`
* Usas funciones dentro de Context (`useContext`)

`useCallback` soluciona esto al **estabilizar la referencia** de la función.

---

### ¿Por qué existe este Hook y qué limitación del enfoque anterior soluciona?

Antes de Hooks:

* En clases, los métodos tenían referencia estable
* Se usaba `.bind()` o propiedades de clase
* No existía este problema de recreación por render

Con componentes funcionales:

* Todo es una función
* Cada render recrea closures
* Se pierde estabilidad de identidad

`useCallback` existe para:

* Recuperar la **estabilidad referencial**
* Permitir optimizaciones reales con `React.memo`
* Evitar ejecuciones innecesarias de efectos

---

### ¿En qué casos se debe usar y en cuáles NO?

**Úsalo cuando:**

* Pasas funciones como props a componentes hijos **memoizados**
* Pasas funciones dentro de un `Context.Provider`
* Usas funciones como dependencias de `useEffect`
* Renderizas listas (`FlatList`) con callbacks estables
* Necesitas evitar renders innecesarios por identidad de función

**NO lo uses cuando:**

* La función **no sale del componente**
* El componente hijo **no está memoizado**
* No hay problemas reales de rendimiento
* Lo usas “por costumbre” o “porque React lo recomienda”

> Regla clave:
> **Si la identidad de la función no importa, `useCallback` no aporta nada.**

---

### Relación del Hook con el ciclo de renderizado y la reconciliación de React

* En cada render, React compara dependencias
* Si no cambian:

  * devuelve **la misma función** (misma referencia)
* Si cambian:

  * crea una **nueva función**
* Esto permite:

  * que `React.memo` detecte props “iguales”
  * que `useEffect` no se dispare innecesariamente

Importante:

* `useCallback` **no evita el render del componente padre**
* solo evita que cambien las referencias de funciones

---

## 2) Ejemplo mínimo funcional

```jsx
import React, { useCallback, useState } from "react";
import { View, Text, Pressable } from "react-native";

export default function App() {
  const [count, setCount] = useState(0);

  const incrementar = useCallback(() => {
    setCount((prev) => prev + 1);
  }, []);

  return (
    <View>
      <Text>Contador: {count}</Text>
      <Pressable onPress={incrementar}>
        <Text>Incrementar</Text>
      </Pressable>
    </View>
  );
}
```

### Explicación

* `useCallback(() => {...}, [])`

  * crea una función una sola vez
  * la referencia se mantiene estable
* La forma funcional de `setCount` evita depender de `count`

### ¿Qué ocurre internamente cuando el Hook se ejecuta?

* React guarda:

  * la función
  * el array de dependencias
* En renders posteriores:

  * si las dependencias no cambian → devuelve la **misma función**
  * si cambian → crea una nueva

---

## 3) Ejemplo complejo y realista

Caso real: **lista grande + item memoizado + callbacks estables**
(esto es MUY típico en React Native con `FlatList`)

```jsx
import React, { memo, useCallback, useState } from "react";
import { FlatList, Text, Pressable, View } from "react-native";

const Item = memo(function Item({ item, onSelect }) {
  console.log("Render Item", item.id);

  return (
    <Pressable onPress={() => onSelect(item.id)}>
      <Text>{item.nombre}</Text>
    </Pressable>
  );
});

export default function Lista({ data }) {
  const [seleccionado, setSeleccionado] = useState(null);

  const manejarSeleccion = useCallback((id) => {
    setSeleccionado(id);
  }, []);

  const renderItem = useCallback(
    ({ item }) => (
      <Item item={item} onSelect={manejarSeleccion} />
    ),
    [manejarSeleccion]
  );

  return (
    <View>
      <Text>Seleccionado: {seleccionado}</Text>

      <FlatList
        data={data}
        keyExtractor={(item) => String(item.id)}
        renderItem={renderItem}
      />
    </View>
  );
}
```

### Qué demuestra este ejemplo

* `Item` está memoizado (`React.memo`)
* `onSelect` es estable gracias a `useCallback`
* `renderItem` también es estable
* Al cambiar `seleccionado`:

  * solo se re-renderiza lo necesario
* Sin `useCallback`, **todos los items se re-renderizarían**

---

## 4) API del Hook

### Firma

```ts
const memoizedFn = useCallback(fn, deps);
```

### Parámetros de entrada

* `fn: (...args) => any`

  * la función a memorizar
* `deps: any[]`

  * dependencias que determinan cuándo recrear la función

### Valor que retorna

* Retorna **la misma función** mientras las dependencias no cambien

### Qué pasa si se usan valores incorrectos

* Omitir dependencias → función usa valores obsoletos (stale closure)
* Dependencias inestables → función se recrea siempre
* Usar `useCallback` con lógica impura → bugs difíciles

---

## 5) Errores típicos (mínimo 8)

1. **Usar `useCallback` en todas las funciones**

* Síntoma: código más complejo sin mejora
* Por qué: sobre-optimización
* Solución: úsalo solo donde la identidad importa

2. **Olvidar dependencias**

* Síntoma: función usa valores viejos
* Por qué: closures
* Solución: incluir todas las dependencias reales

3. **Incluir dependencias innecesarias**

* Síntoma: la función se recrea siempre
* Por qué: objetos/funciones nuevas
* Solución: estabilizar dependencias

4. **Pensar que evita renders**

* Síntoma: “igual se renderiza”
* Por qué: `useCallback` no evita renders del padre
* Solución: combinar con `React.memo`

5. **No usar forma funcional del setter**

* Síntoma: dependencias innecesarias (`count`)
* Solución: `setCount(prev => ...)`

6. **Pasar callbacks inestables a Context**

* Síntoma: renders masivos de consumidores
* Solución: `useCallback` + `useMemo` en Provider

7. **Usar `useCallback` sin `memo` en hijos**

* Síntoma: ninguna mejora
* Solución: memoizar componentes relevantes

8. **Depender de mutaciones**

* Síntoma: comportamiento inconsistente
* Solución: inmutabilidad

---

## 6) Hooks, APIs y herramientas relacionadas

Hooks relacionados:

* `useMemo` → memoriza valores
* `useEffect` → callbacks como dependencias
* `useContext` → callbacks en Providers
* `useRef` → referencias mutables sin render
* `useState` / `useReducer` → estado usado por callbacks

APIs relacionadas:

* `React.memo`
* `FlatList` (RN)
* `Pressable`, `TouchableOpacity`

Casos donde conviene combinarlos:

* `useCallback` + `React.memo` (patrón clásico)
* `useCallback` + `useEffect` (deps estables)
* `useCallback` + Context Providers

---

## 7) Buenas prácticas

### Reglas oficiales del Hook

* Mantener dependencias correctas
* No llamar Hooks condicionalmente
* Usar funciones puras

### Patrones recomendados

* Callbacks estables solo si salen del componente
* Usar forma funcional del setter
* Extraer lógica compleja a Hooks personalizados

Ejemplo recomendado:

```jsx
const onSave = useCallback(() => {
  setDatos(prev => ({ ...prev, guardado: true }));
}, []);
```

### Antipatrones comunes

* `useCallback` por defecto
* Callbacks gigantes dentro del Hook
* Usarlo para “arreglar” arquitectura

### Consideraciones de rendimiento

* `useCallback` tiene overhead
* Beneficio solo cuando evita renders reales
* Medir antes de optimizar

### Código legible y predecible

* Nombres claros (`handleSubmit`, `onSelect`)
* Dependencias explícitas
* Hooks pequeños y enfocados

---

## 8) Cuándo NO usar este Hook

* Funciones internas simples
* Componentes pequeños sin memoización
* Cuando no hay props/callbacks compartidos
* Cuando el problema es diseño, no rendimiento

Alternativas:

* Función normal
* `useRef` (si solo necesitas persistencia)
* Reestructurar componentes
* Optimizar primero con arquitectura

---

### Modelo mental final (clave)

* `useState` → guarda estado
* `useEffect` → ejecuta efectos
* `useMemo` → memoriza valores
* `useCallback` → memoriza funciones
* `useContext` → propaga valores

> **`useCallback` no acelera React**
> **Evita trabajo innecesario cuando la identidad de una función importa**

Si quieres, el siguiente paso natural es **`useReducer`** (cuando `useState` empieza a romperse) o una **comparativa real `useMemo` vs `useCallback`** con reglas de decisión claras.
