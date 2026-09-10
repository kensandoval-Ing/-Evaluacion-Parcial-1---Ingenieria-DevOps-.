# Guía de comandos — paso a paso (proyecto Biblioteca)

Sigue esto en orden. Repártanse los pasos con su pareja: uno sube la base (paso 1-3), y cada
uno hace un feature y se turnan para el hotfix.

## Paso 0: Requisitos

- Git instalado.
- Cuenta de GitHub.
- (Opcional para probar localmente) JDK 21 + MySQL, siguiendo `INSTRUCCIONES EJECUCION.md`.
  No es obligatorio tenerlo corriendo para cumplir este encargo, ya que lo que se evalúa es el
  flujo de Git/GitHub, no que el sistema funcione end-to-end.

## Paso 1: Crear el repositorio en GitHub

1. Ve a github.com → New repository.
2. Nómbralo, por ejemplo, `biblioteca-devops`.
3. NO marques "Add a README" (ya tenemos uno). Crea el repo vacío.
4. Copia la URL que te da GitHub.

## Paso 2: Subir el proyecto base (rama main)

Desde la carpeta `Biblioteca-Springboot` que te entregué (ya con el `pom.xml` corregido,
`.gitignore`, `README.md` y el workflow de Actions):

```bash
cd Biblioteca-Springboot
git init
git add .
git commit -m "feat: estructura base del sistema de microservicios de biblioteca"
git branch -M main
git remote add origin https://github.com/TU-USUARIO/biblioteca-devops.git
git push -u origin main
```

## Paso 3: Crear y subir la rama develop

```bash
git checkout -b develop
git push -u origin develop
```

Con esto ya tienes `main` y `develop` en GitHub (punto 1 del encargo).

## Paso 4: Feature 1 — buscar libros por autor (ms-catalogo)

```bash
git checkout develop
git checkout -b feature/busqueda-por-autor
```

**Archivo:** `ms-catalogo/src/main/java/cl/triskeledu/catalogo/repository/LibroRepository.java`

Agrega este método a la interfaz:

```java
List<Libro> findByAutorContainingIgnoreCase(String autor);
```

(recuerda agregar `import java.util.List;` si no está)

**Archivo:** `ms-catalogo/src/main/java/cl/triskeledu/catalogo/service/LibroService.java`

Agrega este método a la clase:

```java
public List<LibroResponse> findByAutor(String autor) {
    return libroMapper.toResponseList(libroRepository.findByAutorContainingIgnoreCase(autor));
}
```

**Archivo:** `ms-catalogo/src/main/java/cl/triskeledu/catalogo/controller/LibroController.java`

Agrega este endpoint (junto a los demás `@GetMapping`):

```java
@Operation(summary = "Buscar libros por autor", description = "Retorna los libros cuyo autor coincide (parcial, sin distinguir mayusculas)")
@GetMapping("/autor/{autor}")
public ResponseEntity<List<LibroResponse>> findByAutor(
        @Parameter(description = "Nombre o parte del nombre del autor", required = true, example = "Garcia")
        @PathVariable String autor) {
    return ResponseEntity.ok(libroService.findByAutor(autor));
}
```

Luego:

```bash
git add .
git commit -m "feat(ms-catalogo): agrega busqueda de libros por autor"
git push -u origin feature/busqueda-por-autor
```

Ve a GitHub y crea el Pull Request `develop` ← `feature/busqueda-por-autor`. Tu compañero/a lo
revisa y aprueba, y lo mergean con "Squash and merge". Verás que se dispara el workflow.

## Paso 5: Feature 2 — listar solo usuarios activos (ms-usuarios)

```bash
git checkout develop
git pull origin develop
git checkout -b feature/usuarios-activos
```

**Archivo:** `ms-usuarios/src/main/java/cl/triskeledu/usuarios/repository/UsuarioRepository.java`

```java
List<Usuario> findByActivoTrue();
```

(agrega `import java.util.List;` si no está)

**Archivo:** `ms-usuarios/src/main/java/cl/triskeledu/usuarios/service/UsuarioService.java`

```java
public List<UsuarioResponse> findActivos() {
    return usuarioMapper.toResponseList(usuarioRepository.findByActivoTrue());
}
```

**Archivo:** `ms-usuarios/src/main/java/cl/triskeledu/usuarios/controller/UsuarioController.java`

```java
@GetMapping("/activos")
public ResponseEntity<List<UsuarioResponse>> findActivos() {
    List<UsuarioResponse> usuarios = usuarioService.findActivos();
    usuarios.forEach(this::addLinks);
    return ResponseEntity.ok(usuarios);
}
```

Luego:

```bash
git add .
git commit -m "feat(ms-usuarios): agrega endpoint para listar usuarios activos"
git push -u origin feature/usuarios-activos
```

Crea el Pull Request `develop` ← `feature/usuarios-activos`, apruébenlo y mergéenlo.

## Paso 6: Hotfix — ISBN con espacios no se encuentra (ms-catalogo)

Simulemos que en producción (`main`) se detecta que si el ISBN llega con espacios al inicio o
final (por ejemplo, copiado y pegado desde otra fuente), la búsqueda falla y devuelve "no
encontrado" aunque el libro sí existe. El hotfix nace desde `main`, no desde `develop`:

```bash
git checkout main
git pull origin main
git checkout -b hotfix/isbn-con-espacios
```

**Archivo:** `ms-catalogo/src/main/java/cl/triskeledu/catalogo/service/LibroService.java`

Modifica los métodos `getLibroByIsbn` y `existsByIsbn` para limpiar el ISBN antes de buscar:

```java
private Libro getLibroByIsbn(String isbn) {
    return libroRepository.findByIsbn(isbn.trim()).orElseThrow(() -> new EntityNotFoundException("Libros", "ISBN", isbn));
}

public boolean existsByIsbn(String isbn) {
    return libroRepository.existsByIsbn(isbn.trim());
}
```

```bash
git add .
git commit -m "fix(ms-catalogo): normaliza isbn con espacios antes de buscar"
git push -u origin hotfix/isbn-con-espacios
```

Crea el Pull Request `main` ← `hotfix/isbn-con-espacios`, apruébenlo y mergéenlo. Esto
disparará el workflow (por ser PR contra `main`).

**Importante:** después de mergear el hotfix a `main`, hay que traer ese arreglo también a
`develop` para que no se pierda:

```bash
git checkout develop
git pull origin develop
git merge main
git push origin develop
```

## Paso 7: Verificar GitHub Actions

Ve a la pestaña "Actions" del repositorio en GitHub. Deberías ver ejecuciones del workflow
"CI - Biblioteca Microservicios" disparadas por:
- Los push a `develop` (al mergear los features).
- Los Pull Requests hacia `main` (el hotfix).

Si algún build falla, revisa el log — lo más común es un typo al copiar el código de esta
guía (falta un import, una llave, etc.).

## Paso 8 (opcional): Release final

```bash
git checkout main
git pull origin main
```

Crea un Pull Request `main` ← `develop` desde GitHub, apruébenlo y mergéenlo. Esto cierra el
ciclo de GitFlow para esta entrega.

## Qué entregar

El enlace del repositorio de GitHub (con `main`, `develop`, los PRs cerrados visibles en el
historial, y las Actions ejecutadas), enviado por AVA y al correo del docente.
