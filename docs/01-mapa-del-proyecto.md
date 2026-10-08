# Mapa del Proyecto

## 1. Qué hace el proyecto

Gestor de tareas pendientes desarrollado en Java 17 con arquitectura en capas sin frameworks. Permite crear tareas con prioridades, marcarlas como completadas, buscarlas por título y generar reportes de estado.

## 2. Tabla de clases

| Clase | Responsabilidad | Depende de |
|-------|----------------|------------|
| `Tarea` | Entidad que representa una tarea pendiente con título, descripción, prioridad y fecha límite | — |
| `Prioridad` | Enumeración de niveles de prioridad: BAJA, MEDIA, ALTA | — |
| `TareaNoEncontradaException` | Excepción lanzada cuando se busca una tarea por ID inexistente | — |
| `TareaRepositorio` | Persistencia en memoria con LinkedHashMap, asigna IDs autoincrementales desde 1 | `Tarea` |
| `TareaServicio` | Casos de uso del gestor: crear, completar, eliminar, listar, generar reportes | `TareaRepositorio`, `Tarea`, `Prioridad`, `TareaNoEncontradaException` |
| `App` | Demo de consola que crea tareas de ejemplo y muestra el reporte | `TareaServicio`, `TareaRepositorio` |
| `TareaRepositorioTest` | Suite de tests unitarios del repositorio | `TareaRepositorio`, `Tarea`, `Prioridad` |
| `TareaServicioTest` | Suite de tests unitarios del servicio | `TareaServicio`, `TareaRepositorio`, `Tarea`, `Prioridad` |

## 3. Cómo se compila y se prueba

```bash
# Compilar el proyecto
mvn compile

# Ejecutar todos los tests
mvn test

# Ejecutar un test específico
mvn test -Dtest=TareaServicioTest
mvn test -Dtest=TareaRepositorioTest#guardarYBuscar

# Ejecutar la aplicación demo
mvn -q exec:java

# Limpiar y recompilar
mvn clean compile
```

## 4. Tres cosas sospechosas o incompletas

1. **`TareaRepositorio.buscarPorTitulo()` no es case-insensitive**: En la línea 44 de `TareaRepositorio.java` se usa `t.getTitulo().contains(texto)` sin normalizar a minúsculas, aunque el Javadoc promete búsqueda sin distinguir mayúsculas. Existe un test deshabilitado en `TareaRepositorioTest.java:33-39` que documenta este fallo.

2. **`TareaServicio.listarPorPrioridadMinima()` excluye la prioridad solicitada**: En la línea 51 de `TareaServicio.java` se usa `>` en lugar de `>=` para comparar prioridades, lo que excluye incorrectamente las tareas con la prioridad mínima indicada. Si pido MEDIA, no devuelve las MEDIA, solo las ALTA.

3. **`TareaServicio.diasRestantes()` retorna valores negativos invertidos**: En la línea 67 de `TareaServicio.java` el orden de parámetros en `ChronoUnit.DAYS.between(tarea.getFechaLimite(), hoy)` está invertido, causando que retorne `-3` cuando debería ser `3`. El test en `TareaServicioTest:64` espera el valor negativo erróneo.
