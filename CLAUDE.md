# CLAUDE.md

Gestor de tareas pendientes en Java 17 para mentoría técnica de Generation México (CH70 Java). Arquitectura en capas sin frameworks.

## Stack

- Java 17
- Maven 3.8+
- JUnit 5.10.2
- Sin dependencias de frameworks (Java vanilla)

## Comandos

```bash
mvn compile              # Compilar
mvn test                 # Ejecutar tests
mvn -q exec:java         # Correr demo
mvn clean compile        # Limpiar y recompilar
```

## Arquitectura

```
App → TareaServicio → TareaRepositorio → Tarea
```

## Convenciones

- Todo el código y las pruebas están en español (nombres de clases, métodos, variables, comentarios)
- JUnit 5: una prueba por comportamiento, nombres descriptivos en español
- No agregues dependencias nuevas sin preguntarme primero
- **Regla obligatoria**: cualquier cambio debe dejar `mvn test` en verde antes del commit

## Bugs conocidos (no corregir sin consultar)

1. `TareaRepositorio.buscarPorTitulo()` línea 44: usa `.contains()` sin normalizar mayúsculas (test deshabilitado)
2. `TareaServicio.listarPorPrioridadMinima()` línea 51: usa `>` en lugar de `>=`, excluye la prioridad pedida
3. `TareaServicio.diasRestantes()` línea 67: parámetros invertidos en `between()`, retorna valores negativos

## Notas técnicas

- `maven-compiler-plugin` 3.13.0 requerido: versiones antiguas ignoran `release=17` y compilan como Java 5
- `Tarea.setId()` es package-private para que solo `TareaRepositorio` asigne IDs
- IDs autoincrementales inician en 1, persistencia en memoria con `LinkedHashMap`
