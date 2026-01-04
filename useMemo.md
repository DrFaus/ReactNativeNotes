# `useMemo` en React / React Native

## 1) Definición clara

### ¿Qué es este Hook?

`useMemo` es un Hook de React que **memoriza (cachea) el resultado de un cálculo** y lo vuelve a calcular **solo cuando cambian sus dependencias**.

En una frase:

> `useMemo` sirve para evitar recalcular **valores derivados** costosos en cada render.

No “hace tu app más rápida por arte de magia”; solo evita trabajo repetido **cuando ese trabajo es significativo**.

---

### ¿Qué problema resuelve dentro del modelo de componentes funcionales?

En React, cada vez que cambia el estado o las props, el componente **se vuelve a ejecutar**. Eso significa que:

* Todo el código dentro del componente se vuelve a correr
* Todos los cálculos se repiten aunque sus entradas no hayan cambiado

Si tienes:

* filtros pesados
* transformaciones de arrays grandes
* parsing o normalización
* cálculos estadísticos
* mapeos complejos para UI

…puedes estar gastando CPU en cada render.

`useMemo` evita ese costo:

* conserva el resultado previo
* recalcula solo cuando debe

---

### ¿Por qué existe este Hook y qué limitación del enfoque anterior soluciona?

Antes de Hooks, en clases se usaba:

* caching manual en variables de instancia
* getters con memoización externa
* `PureComponent` / `shouldComponentUpdate`
* librerías como `memoize-one`

Problemas:

* más boilerplate
* difícil de mantener
* fácil equivocarse y cachear mal

`useMemo` trae memoización **de primera clase**, integrada al modelo declarativo de React.

---

### ¿En qué casos se debe usar y en cuáles NO?

**Úsalo cuando:**

* El cálculo es **costoso** y se repite en renders
* El resultado depende de pocas entradas claras (dependencias)
* Necesitas **estabilidad de referencia** para evitar renders en hijos (`React.memo`)
* Estás construyendo datos para `FlatList`, gráficas, tablas, etc.

Ejemplos típicos en React Native:

* Filtrar y ordenar listas grandes
* Preparar datos para `SectionList`
* Construir diccionarios `{id: item}`
* Calcular métricas sobre datasets

**NO lo uses cuando:**

* El cálculo es trivial
* La UI es pequeña y rápida
* Lo usas “por costumbre”
* Las dependencias son inestables y se recalcula igual

> Regla práctica:
> **Primero hazlo simple. Usa `useMemo` cuando haya un motivo real.**

---

### Relación con el ciclo de renderizado y la reconciliación de React

* `useMemo` se evalúa **durante el render**
* Si las dependencias no cambian, React devuelve el **mismo valor previo**
* Ayuda a:

  1. Reducir cálculo
  2. Reducir renders en hijos

Importante:

* `useMemo` **no evita el render del componente padre**
* solo evita recomputar y estabiliza referencias

---

## 2) Ejemplo mínimo funcional

```jsx
import React, { useMemo, useState } from "react";
import { View, Text, Pressable } from "react-native";

export default function App() {
  const [n, setN] = useState(5);

  const cuadrado = useMemo(() => {
    return n * n;
  }, [n]);

  return (
    <View>
      <Text>n: {n}</Text>
      <Text>n²: {cuadrado}</Text>

      <Pressable onPress={() => setN(prev => prev + 1)}>
        <Text>Incrementar</Text>
      </Pressable>
    </View>
  );
}
```

### Explicación

* `useMemo(() => n * n, [n])`

  * calcula `cuadrado` solo si `n` cambia
* El cálculo es trivial aquí; sirve solo para ilustrar el patrón

### ¿Qué ocurre internamente?

* React guarda:

  * dependencias
  * valor calculado
* En el siguiente render:

  * compara dependencias
  * si no cambian → reutiliza el valor
  * si cambian → recalcula

---

## 3) Ejemplo complejo y realista

Caso real: **lista filtrable y ordenable con buscador**

```jsx
import React, { useMemo, useState } from "react";
import { View, Text, TextInput, FlatList, Pressable } from "react-native";

function normaliza(s) {
  return s.toLowerCase().trim();
}

export default function ListaAlumnos({ alumnos }) {
  const [query, setQuery] = useState("");
  const [soloAprobados, setSoloAprobados] = useState(false);

  const alumnosFiltrados = useMemo(() => {
    const q = normaliza(query);

    let arr = alumnos;

    if (soloAprobados) {
      arr = arr.filter(a => a.calificacion >= 70);
    }

    if (q.length > 0) {
      arr = arr.filter(a => normaliza(a.nombre).includes(q));
    }

    arr = [...arr].sort((a, b) => b.calificacion - a.calificacion);

    return arr;
  }, [alumnos, query, soloAprobados]);

  return (
    <View style={{ padding: 16 }}>
      <Text style={{ fontSize: 18, fontWeight: "700" }}>Alumnos</Text>

      <TextInput
        value={query}
        onChangeText={setQuery}
        placeholder="Buscar por nombre…"
        style={{ borderWidth: 1, padding: 10, borderRadius: 10 }}
      />

      <Pressable onPress={() => setSoloAprobados(v => !v)}>
        <Text>
          {soloAprobados ? "Mostrando aprobados" : "Mostrando todos"}
        </Text>
      </Pressable>

      <FlatList
        data={alumnosFiltrados}
        keyExtractor={item => String(item.id)}
        renderItem={({ item }) => (
          <View style={{ paddingVertical: 8 }}>
            <Text style={{ fontWeight: "700" }}>{item.nombre}</Text>
            <Text>Calificación: {item.calificacion}</Text>
          </View>
        )}
      />
    </View>
  );
}
```

### Qué demuestra este ejemplo

* `useMemo` evita recalcular filtros y ordenamientos
* Interacción con estado
* Caso real con `FlatList`
* Mejora real cuando la lista es grande

---

## 4) API del Hook

### Firma

```ts
const memoizedValue = useMemo(factory, deps);
```

### Parámetros

* `factory: () => T`

  * función pura que calcula el valor
* `deps: any[]`

  * dependencias del cálculo

### Retorna

* El valor memoizado

### Uso incorrecto

* Omitir dependencias → valores obsoletos
* Incluir dependencias inestables → recalcula siempre
* Side-effects dentro del factory → error conceptual

---

## 5) Errores típicos

1. Usar `useMemo` para todo
2. Olvidar dependencias
3. Dependencias inestables
4. Side-effects dentro de `useMemo`
5. Creer que evita renders
6. Usarlo como estado
7. Mutar datos
8. Memorizar cálculos triviales

---

## 6) Hooks, APIs y herramientas relacionadas

* `React.memo`
* `useCallback`
* `useRef`
* `useEffect`
* `FlatList`

Se combinan para:

* estabilidad de referencias
* reducción de renders innecesarios

---

## 7) Buenas prácticas

* Factory **pura**
* Dependencias completas
* Memoizar solo cuando aporta valor

### Antipatrones

* `useMemo` por default
* `useMemo` para “arreglar” arquitectura

### Rendimiento

* Tiene overhead
* Útil cuando el cálculo es caro o evita renders reales

---

## 8) Cuándo NO usar este Hook

* Cálculos simples
* UI pequeña
* Cuando el problema es arquitectura, no CPU

**Alternativas:**

* Cálculo directo
* `React.memo`
* `useCallback`
* `useReducer`

---

### Modelo mental final

* `useState` → estado
* `useEffect` → efectos
* `useMemo` → valores derivados
* `useCallback` → funciones derivadas
* `useContext` → propagación de datos

---

Si luego quieres, puedo hacer lo mismo en **Markdown** para `useCallback`, `useReducer` o dejarte un **documento completo de Hooks** listo para clase o apuntes.
