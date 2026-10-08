# CLAUDE.md

Este archivo proporciona contexto a Claude Code (claude.ai/code) para trabajar con el código de este repositorio.

## Descripción del Proyecto

Gestor de tareas pendientes desarrollado en Java 17. Proyecto de práctica para la mentoría técnica de Generation México (CH70 Java). Implementa un sistema de gestión de tareas con arquitectura en capas sin dependencias externas de frameworks.

## Comandos Principales

```bash
# Compilar el proyecto
mvn compile

# Ejecutar tests
mvn test

# Ejecutar un test específico
mvn test -Dtest=TareaServicioTest
mvn test -Dtest=TareaRepositorioTest#guardarYBuscar

# Ejecutar la aplicación demo
mvn -q exec:java

# Limpiar y recompilar
mvn clean compile
```

## Arquitectura

Arquitectura en capas sin frameworks (Java vanilla):

```
App (demo) → TareaServicio (lógica de negocio) → TareaRepositorio (persistencia en memoria) → Tarea (modelo)
```

### Componentes:

- **Modelo**: `Tarea` (entidad principal), `Prioridad` (enum: BAJA/MEDIA/ALTA), `TareaNoEncontradaException` (excepción de dominio)
- **Repositorio**: Implementación en memoria usando `LinkedHashMap`, asigna IDs autoincrementales iniciando en 1
- **Servicio**: Capa de lógica de negocio con casos de uso (crear, completar, listar, generar reportes). Recibe el repositorio mediante inyección por constructor

**Nota de diseño**: El método `Tarea.setId()` tiene visibilidad package-private para que únicamente `TareaRepositorio` pueda asignar identificadores, preservando el control del autoincremento.

## Defectos Conocidos

### Bugs confirmados con evidencia en tests:

1. **`TareaRepositorio.buscarPorTitulo()`** (línea 44): Utiliza `.contains()` sin normalización a minúsculas. No realiza búsqueda case-insensitive como especifica el Javadoc. Test correspondiente deshabilitado con `@Disabled` en `TareaRepositorioTest:33-39`.

2. **`TareaServicio.listarPorPrioridadMinima()`** (línea 51): Operador de comparación incorrecto (`>` en lugar de `>=`). Excluye erróneamente tareas con la prioridad mínima solicitada.

3. **`TareaServicio.diasRestantes()`** (línea 67): Orden incorrecto de parámetros en `ChronoUnit.DAYS.between()`. Retorna valores negativos cuando deberían ser positivos. El test en `TareaServicioTest:64` espera `-3` pero la semántica correcta sería `3`.

4. **`Tarea` constructor** (línea 19): Ausencia de validación para `titulo` vacío o blank. Actualmente acepta cadenas vacías sin rechazarlas.

### Cobertura de tests incompleta (TODOs documentados):

- `TareaServicioTest:39` - Verificar que `completar(999)` lance `TareaNoEncontradaException`
- `TareaServicioTest:102` - Tests del método `eliminar(int)` no implementados
- `TareaRepositorio.todas()` sin cobertura de tests

## Configuración Maven

- **maven-compiler-plugin versión 3.13.0**: Versión explícita requerida. Las versiones antiguas (3.1 incluida en distribuciones Ubuntu/Debian) ignoran `maven.compiler.release` y compilan con Java 5, causando errores "Source option 5 is no longer supported".
- **exec-maven-plugin**: Configurado con `mainClass=mx.generation.tareas.App` para ejecutar la demo directamente con `mvn exec:java`.
