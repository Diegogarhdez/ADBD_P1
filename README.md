# ADBD_P1

## Diego García Hernández y Marcos Barbuzano Socorro

### 1. CREACIÓN DE LA BASE DE DATOS

```bash
createdb biblioteca
\c biblioteca
```

2. CREACIÓN DE USUARIOS
Crear dos usuarios:
- admin_biblio con permisos de administrador sobre la base de datos.
```sql
  CREATE USER admin_biblio WITH PASSWORD 'admin1234';
```
- usuario_biblio con permisos solo de lectura.
```sql
CREATE USER usuario_biblio WITH PASSWORD 'usuario1234';
```
Crear un rol llamado lectores con permisos únicamente de consulta sobre todas las tablas de la base de datos.
```sql
CREATE ROLE lectores;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lectores; *
```
*(Este comando se hará una vez se hayan creado las tablas).
Asignar el usuario usuario_biblio a este rol.
```sql
GRANT lectores TO usuario_biblio;
```
Consultar las tablas del sistema para listar todos los usuarios creados (pg_roles).
```sql
SELECT rolname FROM pg_roles;
```
Cambiar la contraseña del usuario usuario_biblio.
```sql
ALTER USER usuario_biblio WITH PASSWORD 'usuario123';
```
Configurar permisos de tal forma que el usuario usuario_biblio no pueda eliminar registros en ninguna tabla.
```
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lectores;
(Haremos esto cuando se creen las tablas).
```

### 3. CREACIÓN DE TABLAS

- Crear las siguientes tablas con sus respectivas claves primarias y establecer las claves foráneas correspondientes:

  1. **autores** (`id_autor`, `nombre`, `nacionalidad`)
     ```sql
     CREATE TABLE autores (
         id_autor SERIAL PRIMARY KEY,
         nombre VARCHAR(100) NOT NULL,
         nacionalidad VARCHAR(50)
     );
     ```

  2. **libros** (`id_libro`, `titulo`, `año_publicacion`, `id_autor`)
     ```sql
     CREATE TABLE libros (
         id_libro SERIAL PRIMARY KEY,
         titulo VARCHAR(200) NOT NULL,
         anyo_publicacion INT,
         id_autor INT,
         FOREIGN KEY (id_autor) REFERENCES autores(id_autor)
     );
     ```

  3. **prestamos** (`id_prestamo`, `id_libro`, `fecha_prestamo`, `fecha_devolucion`, `usuario_prestatario`)
     ```sql
     CREATE TABLE prestamos (
         id_prestamo SERIAL PRIMARY KEY,
         id_libro INT,
         fecha_prestamo DATE NOT NULL,
         fecha_devolucion DATE,
         usuario_prestatario VARCHAR(100),
         FOREIGN KEY (id_libro) REFERENCES libros(id_libro) ON DELETE CASCADE
     );
     ```

### 4. INSERCIÓN DE DATOS

- Insertar al menos 5 autores, 8 libros y 5 préstamos de ejemplo.

#### Autores
```sql
INSERT INTO autores (nombre, nacionalidad) VALUES 
('Gabriel García Márquez', 'Colombiana'),
('Jorge Luis Borges', 'Argentina'),
('Miguel de Cervantes', 'Española'),
('George Orwell', 'Británica'),
('Julio Cortázar', 'Argentina');
```
### Libros
```sql
INSERT INTO libros (titulo, año_publicacion, id_autor) VALUES
('Cien años de soledad', 1967, 1),
('El amor en los tiempos del cólera', 1985, 1),
('Ficciones', 1944, 2),
('El Aleph', 1949, 2),
('Don Quijote de la Mancha', 1605, 3),
('1984', 1949, 4),
('Rebelión en la granja', 1945, 4),
('Rayuela', 1963, 5);
```
### Préstamos
```sql
INSERT INTO prestamos (id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario)
VALUES
(1, '2026-09-01', '2026-09-10', 'Marcos'),
(2, '2026-09-05', NULL, 'Ana'),
(3, '2026-09-07', '2026-09-14', 'Luis'),
(6, '2026-09-10', NULL, 'Marcos'),
(8, '2026-09-12', NULL, 'Carlos');
```

### 5. CONSULTAS BÁSICAS

- Listar todos los libros con su autor correspondiente:

``` sql
SELECT libros.titulo, autores.nombre AS autor 
FROM libros 
JOIN autores ON libros.id_autor = autores.id_autor;
```
Mostrar los préstamos que aún no tienen fecha de devolución:
```sql
SELECT * FROM prestamos 
WHERE fecha_devolucion IS NULL;
```
Obtener los autores que tienen más de un libro registrado:
```sql
SELECT autores.nombre, COUNT(libros.id_libro) AS numero_libros 
FROM autores 
JOIN libros ON autores.id_autor = libros.id_autor 
GROUP BY autores.id_autor, autores.nombre 
HAVING COUNT(libros.id_libro) > 1;
```

