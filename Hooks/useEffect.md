# `useEffect` en React / React Native

## 1) Definición clara

### ¿Qué es este Hook?

`useEffect` es un Hook de React para ejecutar **efectos secundarios** en componentes funcionales.

Un efecto secundario es cualquier operación que:

* **No pertenece al render** (no es “describir UI”)
* Interactúa con el mundo exterior o con APIs del entorno
* Puede requerir **limpieza** (cleanup)

Ejemplos típicos:

* Llamadas HTTP (`fetch`)
* Timers (`setTimeout`, `setInterval`)
* Suscripciones/listeners (teclado, red, sockets)
* Sincronización con almacenamiento (AsyncStorage)
* Cambios en el título, analytics, etc. (más web)

> Regla mental:
> **Render = describir qué se ve.**
> **useEffect = describir qué debe pasar después.**

---

### ¿Qué problema resuelve dentro del modelo de componentes funcionales?

Los componentes funcionales deberían ser **puros**:

* Mismas entradas → mismo JSX
* Sin “hacer cosas” durante el render

Pero las apps reales necesitan:

* Cargar datos
* Suscribirse a eventos
* Iniciar y limpiar procesos

`useEffect` separa claramente:

* **UI (render)** de **acciones/side-effects (effect)**

---

### ¿Por qué existe este Hook y qué limitación del enfoque anterior soluciona?

Antes de Hooks (componentes de clase), el ciclo de vida se fragmentaba:

* `componentDidMount` (montaje)
* `componentDidUpdate` (actualización)
* `componentWillUnmount` (limpieza)

Problemas:

* Lógica relacionada dispersa en varios métodos
* Fácil duplicar o desordenar efectos
* Difícil reutilizar lógica

Con `useEffect`:

* Unificas el comportamiento de ciclo de vida con una sola abstracción
* Declaras dependencias explícitas
* Puedes extraer lógica a Hooks personalizados

---

### ¿En qué casos se debe usar y en cuáles NO?

**Úsalo cuando:**

* Necesitas sincronizar tu componente con algo externo:

  * API / base de datos / almacenamiento
  * timers
  * listeners/suscripciones
* Debes ejecutar algo cuando un valor cambia (estado/props) y eso “algo” es un efecto real
* Debes limpiar recursos al desmontar o al cambiar dependencias

**No lo uses cuando:**

* Solo estás calculando un valor derivado (usa cálculo directo o `useMemo`)
* Estás intentando reemplazar lógica de eventos (un botón debe manejarse en `onPress`, no con `useEffect`)
* Estás duplicando estado (“cuando X cambie, actualiza Y”) sin necesidad
* Tu efecto solo existe por arquitectura incorrecta

> Regla rápida:
> **Si no tocas nada “externo”, probablemente no necesitas `useEffect`.**

---

### Relación del Hook con el ciclo de renderizado y la reconciliación de React

Flujo típico:

1. React ejecuta el componente (render) → produce JSX
2. React reconcilia → actualiza UI (commit)
3. **Después del commit**, React ejecuta los efectos

Cuando cambian dependencias:

* React primero renderiza de nuevo
* Luego, antes de aplicar el nuevo efecto:

  * ejecuta el **cleanup** del efecto anterior (si existe)
* finalmente ejecuta el nuevo efecto

Importante:

* `useEffect` corre **después** de pintar (no bloquea el render)
* En modo desarrollo con Strict Mode, React puede ejecutar efectos más de una vez para detectar problemas (no confundir con “bug”)

---

## 2) Ejemplo mínimo funcional

```jsx
import React, { useEffect } from "react";
import { Text } from "react-native";

export default function App() {
  useEffect(() => {
    console.log("Montado: este efecto corre una vez");
  }, []);

  return <Text>Hola</Text>;
}
```

### Explicación breve

* `useEffect(() => {...}, [])`

  * el array vacío significa: **sin dependencias**
  * corre una vez tras el primer render (montaje)
* El render retorna `<Text>Hola</Text>`

### ¿Qué ocurre internamente cuando se ejecuta?

* React “registra” el efecto durante el render
* Después del commit, llama a la función del efecto
* Como `deps` es `[]`, no vuelve a correr (salvo remount o comportamientos de dev)

---

## 3) Ejemplo complejo y realista

Caso real: **búsqueda con debounce + fetch + cleanup** (muy común en RN)

```jsx
import React, { useEffect, useMemo, useState } from "react";
import { View, Text, TextInput, FlatList } from "react-native";

export default function Buscador({ endpointBase }) {
  const [query, setQuery] = useState("");
  const [resultados, setResultados] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // debounce simple: retrasar la búsqueda
  const queryNormalizada = useMemo(() => query.trim(), [query]);

  useEffect(() => {
    if (queryNormalizada.length === 0) {
      setResultados([]);
      setError(null);
      return;
    }

    let cancelado = false;
    const controller = new AbortController();

    setLoading(true);
    setError(null);

    const id = setTimeout(async () => {
      try {
        const res = await fetch(
          `${endpointBase}?q=${encodeURIComponent(queryNormalizada)}`,
          { signal: controller.signal }
        );

        if (!res.ok) throw new Error("Respuesta no OK");

        const data = await res.json();

        if (!cancelado) {
          setResultados(Array.isArray(data) ? data : []);
        }
      } catch (e) {
        if (!cancelado && e.name !== "AbortError") {
          setError("Error al buscar");
          setResultados([]);
        }
      } finally {
        if (!cancelado) setLoading(false);
      }
    }, 400);

    return () => {
      // Cleanup:
      // 1) cancela el timeout (debounce)
      // 2) aborta fetch si seguía en vuelo
      // 3) evita setState en efecto viejo
      cancelado = true;
      clearTimeout(id);
      controller.abort();
    };
  }, [endpointBase, queryNormalizada]);

  return (
    <View style={{ padding: 16, gap: 12 }}>
      <Text style={{ fontSize: 18, fontWeight: "700" }}>Buscador</Text>

      <TextInput
        value={query}
        onChangeText={setQuery}
        placeholder="Escribe para buscar…"
        style={{ borderWidth: 1, padding: 10, borderRadius: 10 }}
      />

      {loading && <Text>Cargando…</Text>}
      {error && <Text style={{ color: "red" }}>{error}</Text>}

      <FlatList
        data={resultados}
        keyExtractor={(item, index) => String(item.id ?? index)}
        renderItem={({ item }) => (
          <View style={{ paddingVertical: 8 }}>
            <Text style={{ fontWeight: "700" }}>{item.title ?? "Sin título"}</Text>
          </View>
        )}
      />
    </View>
  );
}
```

### Qué demuestra este ejemplo

* `useEffect` reaccionando a cambios (`queryNormalizada`, `endpointBase`)
* Efecto real: fetch externo
* Debounce con `setTimeout`
* Cleanup sólido:

  * cancela timeout
  * aborta fetch
  * evita “setState en componente desmontado/efecto viejo”
* Interacción con `useState` y `useMemo`

---

## 4) API del Hook

### Firma

```ts
useEffect(effect, deps?)
```

### Parámetros

* `effect: () => void | (() => void)`

  * función del efecto
  * puede retornar un cleanup
* `deps?: any[]`

  * dependencias
  * controlan cuándo corre

### Retorno

* `useEffect` **no retorna** un valor útil al componente.
* El “retorno” importante es la **función de cleanup**, que React guarda y llama cuando toca.

### Qué pasa si se usan valores incorrectos

* Dependencias incompletas → valores obsoletos (stale closures)
* Dependencias excesivas → efecto se ejecuta de más
* Side-effects mal ubicados → loops o glitches
* No limpiar → fugas de memoria, listeners duplicados

---

## 5) Errores típicos (mínimo 8)

1. **Omitir dependencias necesarias**

* Síntoma: usa valores viejos, “no se actualiza”
* Por qué: closure captura variables antiguas
* Solución: incluir dependencias correctas (o reestructurar)

2. **Meter dependencias inestables**

* Síntoma: efecto se dispara en cada render
* Por qué: objetos/funciones nuevas por referencia
* Solución: `useMemo` / `useCallback` o mover fuera

3. **Crear un loop infinito**

* Síntoma: “Maximum update depth exceeded”
* Por qué: `setState` dentro del efecto y deps hacen que se repita
* Solución: revisar deps y lógica; a veces necesitas condicionales

4. **No hacer cleanup**

* Síntoma: listeners duplicados, fugas, comportamiento fantasma
* Por qué: el efecto deja recursos vivos
* Solución: retornar cleanup siempre que abras algo

5. **Hacer fetch sin cancelación**

* Síntoma: warnings de setState tras un unmount o resultados mezclados
* Por qué: la respuesta llega tarde
* Solución: abortar o invalidar efecto (AbortController/flags)

6. **Usar `useEffect` para derivar estado**

* Síntoma: estado duplicado y desincronizado
* Por qué: se intenta “sincronizar” valores que ya se pueden calcular
* Solución: derivar en render o `useMemo`

7. **Efectos gigantes**

* Síntoma: difícil de mantener y testear
* Por qué: mezcla responsabilidades
* Solución: dividir efectos o extraer un Hook

8. **Depender de la ejecución exacta en dev**

* Síntoma: “se ejecuta dos veces”
* Por qué: Strict Mode puede re-invocar para detectar efectos impuros
* Solución: escribir efectos idempotentes + cleanup correcto

---

## 6) Hooks, APIs y herramientas relacionadas

Hooks comunes junto con `useEffect`:

* `useState` (estado que dispara o recibe resultados del efecto)
* `useRef` (guardar valores persistentes sin re-render, por ejemplo “isMounted”)
* `useCallback` (estabilizar handlers usados en deps)
* `useMemo` (estabilizar valores derivados y entradas del efecto)
* `useLayoutEffect` (cuando necesitas correr antes del pintado; más raro en RN)

APIs relacionadas en RN:

* `fetch`
* `AbortController`
* `AppState`, `Keyboard`, `Dimensions` (listeners)
* AsyncStorage (si lo usas) para persistencia
* Librerías como NetInfo (suscripciones de red)

Casos donde conviene combinarlos:

* `useEffect + useState` para carga remota
* `useEffect + useRef` para manejar cancelación/flags
* `useEffect + useCallback` para deps estables

---

## 7) Buenas prácticas

### Reglas oficiales del Hook (y por qué existen)

* Llamar Hooks en el nivel superior
* No llamarlos condicionalmente
* Declarar dependencias correctas
  Porque React necesita saber cuándo el efecto es válido respecto a entradas actuales.

### Patrones recomendados

* Un efecto = una responsabilidad
* Cleanup siempre que exista suscripción/timer/fetch en vuelo
* Extraer lógica a Hooks personalizados (`useDebouncedSearch`, `useAuth`, etc.)
* Mantener el efecto legible: funciones internas claras

### Antipatrones comunes

* Efecto “Dios” que hace todo
* “Watcher” estilo Angular: efecto para todo cambio aunque no sea un efecto
* Duplicar estado: “cuando A cambia, setB(...)” sin necesidad

### Consideraciones de rendimiento

* Efectos corren después del render: pueden suceder muchas veces si deps cambian mucho
* Evita dependencias inestables
* Evita disparar fetch por cada tecla sin debounce

### Cómo mantener código legible y predecible

* Divide en varios `useEffect` por intención
* Nombra funciones internas (`cargarDatos`, `suscribir`, `limpiar`)
* Usa early returns (`if (!condición) return;`) para efectos condicionales limpios

---

## 8) Cuándo NO usar este Hook

* Para cálculos puros o valores derivados → render directo / `useMemo`
* Para manejar una acción directa de usuario (tap) → handler (`onPress`)
* Para reemplazar arquitectura (“tengo estados por todos lados, lo arreglo con efectos”)
* Para sincronizar estado redundante

Alternativas más simples:

* Cálculo directo en render
* `useMemo` para derivados
* `useRef` para valores que no deben renderizar
* Lógica en handlers, no en efectos

---

### Modelo mental final (clave)

> **Render** describe el estado actual de la UI.
> **`useEffect`** describe cómo sincronizar ese estado con el mundo exterior, *después* de renderizar.

Si quieres, el siguiente paso natural sería que hagamos **`useCallback`** (hermano conceptual de `useMemo`) o **`useReducer`** (cuando `useState` se te queda corto), con la misma plantilla.
