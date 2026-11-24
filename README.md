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

-- =====================================================
-- BASE DE DATOS: NISSI ACCESORIOS
-- MOTOR: SQLITE
-- =====================================================

PRAGMA foreign_keys = ON;

-- =====================================================
-- 1. TABLA USUARIOS
-- Almacena los usuarios que ingresan al sistema.
-- =====================================================

CREATE TABLE IF NOT EXISTS usuarios (
    id_usuario INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    usuario TEXT NOT NULL UNIQUE,
    password TEXT NOT NULL,
    rol TEXT NOT NULL DEFAULT 'ADMIN',
    estado INTEGER NOT NULL DEFAULT 1
);

-- =====================================================
-- 2. TABLA PRODUCTOS
-- Almacena los productos del inventario.
-- =====================================================

CREATE TABLE IF NOT EXISTS productos (
    id_producto INTEGER PRIMARY KEY AUTOINCREMENT,
    codigo TEXT NOT NULL UNIQUE,
    nombre TEXT NOT NULL,
    categoria TEXT NOT NULL,
    precio REAL NOT NULL CHECK(precio >= 0),
    stock INTEGER NOT NULL DEFAULT 0 CHECK(stock >= 0),
    descripcion TEXT,
    estado INTEGER NOT NULL DEFAULT 1
);

-- =====================================================
-- 3. TABLA CLIENTES
-- Almacena la información de los clientes.
-- =====================================================

CREATE TABLE IF NOT EXISTS clientes (
    id_cliente INTEGER PRIMARY KEY AUTOINCREMENT,
    documento TEXT NOT NULL UNIQUE,
    nombre TEXT NOT NULL,
    telefono TEXT,
    correo TEXT,
    direccion TEXT,
    estado INTEGER NOT NULL DEFAULT 1
);

-- =====================================================
-- 4. TABLA FACTURAS
-- Almacena la información general de cada factura.
-- =====================================================

CREATE TABLE IF NOT EXISTS facturas (
    id_factura INTEGER PRIMARY KEY AUTOINCREMENT,
    fecha TEXT NOT NULL DEFAULT
        (datetime('now','localtime')),
    usuario TEXT NOT NULL,
    total REAL NOT NULL DEFAULT 0 CHECK(total >= 0),
    id_cliente INTEGER,
    cliente_nombre TEXT NOT NULL DEFAULT 'Consumidor final',

    FOREIGN KEY (id_cliente)
        REFERENCES clientes(id_cliente)
        ON DELETE SET NULL
);

-- =====================================================
-- 5. TABLA DETALLE_FACTURA
-- Almacena los productos incluidos en cada factura.
-- =====================================================

CREATE TABLE IF NOT EXISTS detalle_factura (
    id_detalle INTEGER PRIMARY KEY AUTOINCREMENT,
    id_factura INTEGER NOT NULL,
    id_producto INTEGER,
    codigo TEXT NOT NULL,
    nombre TEXT NOT NULL,
    cantidad INTEGER NOT NULL CHECK(cantidad > 0),
    precio_unitario REAL NOT NULL
        CHECK(precio_unitario >= 0),
    subtotal REAL NOT NULL CHECK(subtotal >= 0),

    FOREIGN KEY (id_factura)
        REFERENCES facturas(id_factura)
        ON DELETE CASCADE,

    FOREIGN KEY (id_producto)
        REFERENCES productos(id_producto)
        ON DELETE SET NULL
);

-- =====================================================
-- 6. TABLA VENTAS
-- Almacena los movimientos de venta de los productos.
-- =====================================================

CREATE TABLE IF NOT EXISTS ventas (
    id_venta INTEGER PRIMARY KEY AUTOINCREMENT,
    id_producto INTEGER NOT NULL,
    codigo TEXT NOT NULL,
    nombre TEXT NOT NULL,
    cantidad INTEGER NOT NULL CHECK(cantidad > 0),
    precio_unitario REAL NOT NULL,
    total REAL NOT NULL,
    usuario TEXT,
    fecha TEXT NOT NULL DEFAULT
        (datetime('now','localtime')),
    id_cliente INTEGER,
    cliente_nombre TEXT NOT NULL DEFAULT 'Consumidor final',

    FOREIGN KEY (id_cliente)
        REFERENCES clientes(id_cliente)
        ON DELETE SET NULL
);

-- =====================================================
-- ÍNDICES PARA OPTIMIZAR LAS CONSULTAS
-- =====================================================

CREATE INDEX IF NOT EXISTS idx_facturas_fecha
ON facturas(fecha);

CREATE INDEX IF NOT EXISTS idx_detalle_factura
ON detalle_factura(id_factura);

-- =====================================================
-- USUARIO ADMINISTRADOR INICIAL
-- =====================================================

INSERT OR IGNORE INTO usuarios
(nombre, usuario, password, rol, estado)
VALUES
('Administrador', 'admin', '12345', 'ADMIN', 1);
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
