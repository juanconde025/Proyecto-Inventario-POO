# Sistema de Gestión de Inventarios "Nissi Accesorios"

**Proyecto del Taller de Panel Administrativo**  
Tecnología en Desarrollo de Sistemas Informáticos  
📅 II Semestre 2026  
👨‍🏫 Profesor: Mag. Carlos Adolfo Beltrán Castro  
👨‍💻 Estudiantes: 
- Juan David Conde Martinez
- Marlene Ramirez Alvarez
- Daniel Eduardo García Chinchilla

---

## 🚀 Descripción del Proyecto

Este proyecto simula un panel administrativo y sistema de facturación/inventario desarrollado con **Java SE - SWING**. La aplicación está diseñada para la tienda **Nissi Accesorios**, permitiendo gestionar el control de stock, catálogo de productos y administración de usuarios con acceso seguro mediante login. Incluye navegación moderna mediante un panel lateral (sidebar) entre las diferentes secciones del sistema, aplicando el patrón de arquitectura **Modelo-Vista-Controlador (MVC)** y persistencia de datos local en **SQLite**.

---

## 📂 Estructura del Proyecto

### 📋 Lista de Menú de Opciones
La interfaz principal cuenta con un panel de navegación que integra el logo institucional de **Nissi Accesorios** y los siguientes ítems de navegación:
- **Dashboard / Inicio:** Vista principal con métricas generales del sistema.
- **Usuarios:** Módulo de administración de credenciales, nombres y roles del personal.
- **Productos (CRUD Activo):** Módulo completo para consultar, registrar, actualizar y eliminar artículos del catálogo.
- **Categorías:** Clasificación de mercancía (Relojería, Tecnología, Accesorios, etc.).
- **Salir:** Cierre de sesión seguro con mensaje de confirmación.

### 🖼️ Vistas - CRUD e Interfaz
- **Módulo de Autenticación (Login):**
  *Pantalla de acceso con validación de credenciales.*

- **Módulo de Gestión de Productos (CRUD):**
  *Tabla interactiva para la creación, lectura, edición y eliminación de productos.*

- **Opción Salir / Cierre de Sesión:**
  *Diálogo informativo para confirmación de cierre de sesión.*

# Módulo de Facturación (conectado al inventario)

Descuento automático de stock: 
Al generar o confirmar una factura, las cantidades vendidas deben restarse del inventario disponible de forma segura 
(idealmente dentro de una transacción para evitar inconsistencias si ocurre un error a mitad del proceso).

Integridad de precios y datos: 
Guarda en el detalle de la factura el precio unitario al momento de la venta 
(no solo la referencia al producto), así si luego cambia el precio del catálogo,
el historial contable/operativo de facturas anteriores no se altera.

Estados de factura y reversión: 
Considera manejar estados (por ejemplo: emitida/pagada vs. anulada)
y define qué pasa con el inventario si se anula una factura (devolver las unidades al stock).

Trazabilidad: Asociar cada factura a la fecha/hora, al usuario que la registró y al cliente (si aplica).

# Creación de Usuarios Solo por Administrador
Control de acceso (RBAC (control de acceso basado en roles)/ permisos): 
Verifica la validación tanto en la interfaz/menú como en la lógica de negocio/base de datos,
para que un usuario estándar no pueda crear cuentas ni escalar sus propios permisos aunque acceda directamente a la ruta o endpoint.

Roles y estado inicial: 
Al crear un usuario, asigna claramente su rol (ej. Administrador, Vendedor/Cajero, Bodeguero) 
y un estado (activo/inactivo) en lugar de borrar registros si hay historial asociado (como facturas creadas por ese usuario).

Seguridad de credenciales: 
Asegúrate de almacenar contraseñas con hash (nunca en texto plano)
y exigir datos mínimos válidos (nombre de usuario único, contraseña robusta, etc.).

---

## 🧰 Lista de Tecnologías Usadas

- **Lenguaje de Programación:** Java SE (JDK 17+)
- **Interfaz Gráfica (GUI):** Java Swing / NetBeans Matisse Builder
- **Base de Datos:** SQLite 3
- **Conector BD:** Driver JDBC SQLite (`sqlite-jdbc`)
- **Arquitectura:** Modelo-Vista-Controlador (MVC)
- **Control de Versiones:** Git & GitHub

---

## 🗄️ Script de Base de Datos y Diagrama Entidad-Relación

### Script SQL (SQLite)
```sql
-- Tabla de Usuarios
CREATE TABLE IF NOT EXISTS usuarios (
    id_usuario INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL UNIQUE,
    password TEXT NOT NULL,
    nombre_completo TEXT NOT NULL,
    rol TEXT CHECK(rol IN ('Administrador', 'Empleado')) DEFAULT 'Empleado'
);

-- Tabla de Categorías
CREATE TABLE IF NOT EXISTS categorias (
    id_categoria INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre_categoria TEXT NOT NULL UNIQUE,
    descripcion TEXT
);

-- Tabla de Productos
CREATE TABLE IF NOT EXISTS productos (
    id_producto INTEGER PRIMARY KEY AUTOINCREMENT,
    codigo TEXT UNIQUE NOT NULL,
    nombre TEXT NOT NULL,
    id_categoria INTEGER NOT NULL,
    precio_compra REAL NOT NULL,
    precio_venta REAL NOT NULL,
    stock INTEGER NOT NULL DEFAULT 0,
    stock_minimo INTEGER NOT NULL DEFAULT 5,
    FOREIGN KEY (id_categoria) REFERENCES categorias(id_categoria)
);

-- Datos Iniciales de Prueba
INSERT INTO usuarios (username, password, nombre_completo, rol) 
VALUES ('admin', 'admin123', 'Administrador Nissi', 'Administrador');

INSERT INTO categorias (nombre_categoria, descripcion) 
VALUES ('Relojería', 'Relojes analógicos y digitales'),
       ('Tecnología', 'Audífonos, cargadores y periféricos');
```

---

## 🔧 Instalación y Ejecución

1. **Clonar el repositorio:**
   ```bash
   git clone https://github.com/juanconde025/Proyecto-Inventario-POO.git
   ```
2. **Abrir el proyecto:**
   - Abrir **Apache NetBeans IDE**.
   - Ir a `File > Open Project` y seleccionar la carpeta clonada.
3. **Configurar Librerías:**
   - Asegurarse de tener agregado el JAR `sqlite-jdbc.jar` en las librerías del proyecto.
4. **Ejecutar la Aplicación:**
   - Hacer clic derecho en el proyecto -> `Run` o ejecutar directamente desde la clase `Main.java`.
   - Credenciales por defecto:
     - **Usuario:** `admin`
     - **Contraseña:** `admin123`
