# Biblioteca — Sistema de Microservicios

Sistema de gestión de biblioteca compuesto por varios microservicios en Spring Boot,
desarrollado como base para la Evaluación Parcial 1 de Ingeniería DevOps (DOY0101). Sirve
como punto de partida para el pipeline DevOps que se construirá durante el semestre.

## Arquitectura

| Módulo        | Descripción                                              | Puerto |
|---------------|-----------------------------------------------------------|--------|
| `common`      | Librería compartida (seguridad JWT, manejo de errores)   | -      |
| `eureka`      | Servidor de descubrimiento de servicios                  | 8761   |
| `ms-usuarios` | Gestión de usuarios y autenticación                       | 9001   |
| `ms-catalogo` | Gestión del catálogo de libros y categorías                | 9002   |
| `ms-recursos` | Gestión de recursos físicos (ejemplares) y préstamos       | 9003   |
| `api-gateway` | Punto de entrada único, enruta hacia los microservicios    | 9000   |

Instrucciones detalladas de instalación y ejecución local en `INSTRUCCIONES EJECUCION.md`.

## Estrategia de ramificación: GitFlow

Elegimos **GitFlow** en lugar de trunk-based development por los siguientes motivos:

- El equipo es reducido (2 integrantes) y trabaja de forma asíncrona, por lo que necesitamos
  una rama de integración (`develop`) separada de la rama estable (`main`), para no romper
  la versión estable mientras se prueban cambios en cualquiera de los microservicios.
- El encargo exige simular explícitamente ramas `feature/` y `hotfix/`, que son parte central
  del modelo GitFlow.
- Al tratarse de un sistema con múltiples microservicios interdependientes (usuarios,
  catálogo, recursos, gateway), un cambio mal probado puede afectar a varios servicios a la
  vez. GitFlow da más control y trazabilidad para este escenario que trunk-based development,
  que exige una madurez de pipelines y feature flags que todavía no tenemos en esta etapa del
  curso.
- Al ser un proyecto que evolucionará durante todo el semestre (con múltiples entregas),
  GitFlow ordena mejor el trabajo colaborativo entre los dos integrantes del equipo.

### Ramas del repositorio

- **main**: código estable de todo el sistema de microservicios. Solo recibe merges vía Pull
  Request desde `develop` (releases) o desde `hotfix/*` (arreglos urgentes).
- **develop**: rama de integración. Acumula los features ya probados antes de una release.
- **feature/<nombre>**: una por cada nueva funcionalidad, en el microservicio que corresponda.
  Nace desde `develop` y vuelve a `develop` vía Pull Request.
- **hotfix/<nombre>**: una por cada arreglo urgente sobre producción. Nace desde `main` y se
  mergea a `main` **y** a `develop` vía Pull Request.

## Convenciones de commits

Usamos [Conventional Commits](https://www.conventionalcommits.org/), incluyendo el
microservicio afectado como alcance:

```
<tipo>(<microservicio>): <descripción corta en presente>
```

Tipos usados en este proyecto:

- `feat`: nueva funcionalidad (ej: `feat(ms-catalogo): agrega busqueda de libros por autor`)
- `fix`: corrección de errores (ej: `fix(ms-catalogo): normaliza isbn con espacios`)
- `docs`: cambios en documentación (ej: `docs(readme): agrega convenciones de commits`)
- `ci`: cambios en configuración de integración continua

## Naming de ramas

- `feature/<nombre-corto-en-minuscula-con-guiones>` — ej: `feature/busqueda-por-autor`
- `hotfix/<nombre-corto-en-minuscula-con-guiones>` — ej: `hotfix/isbn-con-espacios`
- No se trabaja directo sobre `main` ni `develop`.

## Estructura de carpetas

```
Biblioteca-Springboot/
├── .github/workflows/   # Pipelines de GitHub Actions
├── common/              # Libreria compartida
├── eureka/               # Servidor de descubrimiento
├── ms-usuarios/          # Microservicio de usuarios
├── ms-catalogo/          # Microservicio de catalogo
├── ms-recursos/          # Microservicio de recursos
├── api-gateway/          # API Gateway
├── init-multi-db/        # Scripts SQL de inicializacion
├── postman/               # Colecciones de Postman para pruebas manuales
└── pom.xml                # POM padre (agrega todos los modulos)
```

## Estrategia de revisión (Pull Requests)

- Todo cambio entra por Pull Request, nunca por push directo a `main` o `develop`.
- Cada PR debe describir brevemente qué microservicio afecta, qué cambia y por qué.
- Se requiere al menos 1 revisión aprobada por el compañero/a de equipo antes de mergear.
- El PR debe pasar el workflow de GitHub Actions (CI) en verde antes de poder mergearse.
- Se usa "Squash and merge" para mantener el historial de `develop`/`main` limpio.

## Control de versiones

- Se usa Git como sistema de control de versiones, alojado en GitHub.
- Los tags de versión (`v1.0.0`, etc.) se crean sobre `main` cada vez que se hace una release
  desde `develop`.

## Integración continua (CI)

El workflow de GitHub Actions (`.github/workflows/ci.yml`) compila y empaqueta todos los
módulos con Maven en cada push a `develop` y en cada Pull Request hacia `main`. No ejecuta los
tests que requieren una base de datos MySQL real (`@SpringBootTest`), ya que el runner de
GitHub Actions no tiene una base de datos configurada en esta etapa del proyecto; sí ejecuta
los tests unitarios que usan Mockito. Esta es una limitación conocida, a mejorar en próximas
evaluaciones (por ejemplo, agregando un servicio de MySQL o H2 al pipeline).

## Uso de Inteligencia Artificial

Se utilizó IA (Claude, Anthropic) como apoyo para: redacción de este README, el workflow de
GitHub Actions y la guía de comandos Git. Las decisiones de diseño (elección de GitFlow,
estructura de ramas, convenciones) fueron discutidas y validadas por el equipo. El código base
del sistema de microservicios fue entregado por el docente del curso.

## Reflexiones individuales

- **Leonardo Sandoval:** El desarrollo de esta evaluación nos permitió comprender de manera práctica el ciclo de vida de un proyecto bajo la filosofía DevOps. Automatizar la integración continua mediante GitHub Actions y gestionar el flujo de trabajo con GitFlow facilitó la colaboración y el control del código, demostrando el impacto positivo de estas prácticas en la calidad y aceleración de los entregables.
- **Javier Valenzuela:** Implementar el pipeline de CI nos ayudó a dimensionar la importancia de detectar errores de forma temprana en el código. Aunque enfrentamos limitaciones técnicas con las pruebas que requerían la base de datos MySQL, esta experiencia sirvió para identificar áreas de mejora clave y entender cómo optimizar la arquitectura del pipeline para futuras iteraciones.