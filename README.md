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
- Mostrar los préstamos que aún no tienen fecha de devolución:
```sql
SELECT * FROM prestamos 
WHERE fecha_devolucion IS NULL;
```
- Obtener los autores que tienen más de un libro registrado:
```sql
SELECT autores.nombre, COUNT(libros.id_libro) AS numero_libros 
FROM autores 
JOIN libros ON autores.id_autor = libros.id_autor 
GROUP BY autores.id_autor, autores.nombre 
HAVING COUNT(libros.id_libro) > 1;
```
### 6. Consultas con agregación

- Calcular el número total de préstamos realizados.
```sql
SELECT COUNT(*) AS total_prestamos 
FROM prestamos;
```
- Obtener el número de libros prestados por cada usuario.
```sql
SELECT usuario_prestatario, COUNT(id_libro) AS total_libros_prestados 
FROM prestamos 
GROUP BY usuario_prestatario;
```
### 7. Modificación de datos

- Actualizar la fecha de devolución de un préstamo pendiente.
```sql
UPDATE prestamos
SET fecha_devolucion = '2026-09-20'
WHERE usuario_prestatario = 'Ana' AND fecha_devolucion IS NULL;
```
- Eliminar un libro y comprobar el efecto en la tabla de préstamos (usar ON DELETE CASCADE o justificar el comportamiento). (ya se hace por la definición de la tabla)
```sql
DELETE FROM libros 
WHERE id_libro = 6;
```
### 8. Creación de vistas

- Crear una vista llamada vista_libros_prestados que muestre: título del libro, autor y nombre del prestatario.
```sql
CREATE VIEW vista_libros_prestados AS
SELECT 
    l.titulo AS titulo_libro, 
    a.nombre AS autor, 
    p.usuario_prestatario
FROM 
    prestamos p
JOIN 
    libros l ON p.id_libro = l.id_libro
JOIN 
    autores a ON l.id_autor = a.id_autor;
```
- Conceder permisos de consulta sobre esta vista únicamente a usuario_biblio.
```sql
REVOKE ALL ON vista_libros_prestados FROM PUBLIC;
GRANT SELECT ON vista_libros_prestados TO usuario_biblio;
```
### 9. Funciones y consultas avanzadas

- Crear una función que reciba el nombre de un autor y devuelva todos los libros escritos por él.
```sql
CREATE OR REPLACE FUNCTION obtener_libros_autor(p_nombre_autor VARCHAR)
RETURNS TABLE (titulo_libro VARCHAR) AS $$
BEGIN
    RETURN QUERY 
    SELECT l.titulo::VARCHAR
    FROM libros l
    JOIN autores a ON l.id_autor = a.id_autor
    WHERE a.nombre = p_nombre_autor;
END;
$$ LANGUAGE plpgsql;
```
- Crear una consulta que devuelva los tres libros más prestados.
```sql
SELECT l.titulo, COUNT(p.id_prestamo) AS numero_prestamos
FROM libros l
JOIN prestamos p ON l.id_libro = p.id_libro
GROUP BY l.id_libro, l.titulo
ORDER BY numero_prestamos DESC
LIMIT 3;
```
### 10. Exportación e importación de datos

- Exportar el contenido de la tabla libros a un archivo CSV.
```sql
\copy libros TO '/tmp/libros.csv' CSV HEADER;
```
- Importar datos adicionales de autores desde un archivo CSV externo.
```sql
\copy autores(nombre, nacionalidad) FROM '/tmp/nuevos_autores.csv' CSV HEADER;
```
nombre,nacionalidad
Isabel Allende,Chilena
Mario Vargas Llosa,Peruana
J.K. Rowling,Británica
