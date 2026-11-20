# 📚 Sistema de Biblioteca — API REST

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![SQLModel](https://img.shields.io/badge/SQLModel-ORM-4B8BBE?style=for-the-badge)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![Pytest](https://img.shields.io/badge/Pytest-Testing-0A9EDC?style=for-the-badge&logo=pytest&logoColor=white)

API RESTful desarrollada con **FastAPI** y **SQLModel** para gestionar el catálogo de una biblioteca, sus autores, socios y el circuito transaccional de préstamos y devoluciones con cálculo automático de multas.

> **Trabajo Práctico N° 2** — Programación IV · Tecnicatura Universitaria en Programación (UTN).

---

## ✨ Características Principales

- **CRUD completo** para `Autores`, `Libros` y `Socios` con paginación (`offset` y `limit`).
- **Relación N-N** entre libros y autores mediante tabla intermedia (`LibroAutor`), devolviendo los autores anidados en las consultas de libros.
- **Arquitectura en capas** (`models/`, `routers/`, `services/`) con inyección de dependencias (`Depends`).
- **Reglas de negocio estrictas** con códigos de estado HTTP semánticos (`200`, `201`, `403`, `404`, `409`).
- **Suite de pruebas automatizadas** con `pytest` y base de datos SQLite en memoria (`StaticPool`).

---

## 🏗️ Arquitectura del Proyecto

```text
UTN-TP2-PROGRAMACION-IV/
├── main.py                  # Punto de entrada FastAPI, ciclo de vida (lifespan) y routers
├── database.py              # Engine SQLite, creación de tablas y SessionDep
├── models/                  # Entidades SQLModel (tablas ORM + esquemas DTO)
│   ├── autor.py             # Autor, LibroAutor (tabla intermedia N-N)
│   ├── libro.py             # Libro, LibroRead (con autores anidados)
│   ├── socio.py             # Socio (validación EmailStr)
│   └── prestamo.py          # Prestamo, PrestamoCreate, DevolucionRequest
├── routers/                 # Controladores HTTP (sin lógica de negocio)
│   ├── autores.py
│   ├── libros.py
│   ├── socios.py
│   └── prestamos.py
├── services/                # Lógica de negocio, validaciones y transacciones
│   ├── autores.py
│   ├── libros.py
│   ├── socios.py
│   └── prestamos.py
├── tests/                   # Tests de integración con base en memoria
│   ├── conftest.py          # Fixtures de sesión aislada y TestClient
│   └── test_biblioteca.py   # Casos felices y de error para todas las reglas
├── requirements.txt
└── pyproject.toml
```

---

## 📊 Modelo de Datos

- **Libro - Autor (`N-N`)**: Un libro puede tener múltiples autores y un autor puede participar en múltiples libros (`LibroAutor`).
- **Préstamo - Socio (`N-1`)**: Cada préstamo pertenece a un único socio.
- **Préstamo - Libro (`N-1`)**: Cada préstamo corresponde a un único libro.

---

## 📋 Reglas de Negocio

| # | Regla | Código HTTP |
| :-: | :--- | :-: |
| 1 | Un socio no puede tener más de **3 préstamos activos** en simultáneo. | `409 Conflict` |
| 2 | No se puede prestar un libro sin stock; al prestar se **descuenta 1** unidad. | `409 Conflict` |
| 3 | Un socio **inactivo** (`activo = false`) no puede solicitar préstamos. | `403 Forbidden` |
| 4 | El plazo estándar de préstamo es de **14 días** (`fecha_vencimiento = fecha_prestamo + 14 días`). | — |
| 5 | Al devolver se **repone el stock** y se calcula la multa: **$100 por día de atraso** (`0` si llega a tiempo). | — |
| 6 | No se puede devolver un préstamo que **ya fue devuelto**. | `409 Conflict` |
| 7 | Cualquier operación sobre un **recurso inexistente** devuelve error. | `404 Not Found` |
| 8 | No se permiten **ISBN** ni **emails** duplicados (en alta o modificación). | `409 Conflict` |

---

## 🔌 Endpoints Principales

| Método | Ruta | Descripción | Códigos HTTP |
| :--- | :--- | :--- | :--- |
| `CRUD` | `/autores`, `/libros`, `/socios` | Alta, consulta, modificación (`PUT`/`PATCH`) y baja | `200`, `201`, `404`, `409` |
| `POST` | `/libros/{id}/autores/{autor_id}` | Asocia un autor existente a un libro | `200`, `404`, `409` |
| `GET` | `/libros?disponibles=true` | Lista libros con `stock > 0` | `200` |
| `POST` | `/prestamos` | Registra un préstamo y descuenta stock | `201`, `403`, `404`, `409` |
| `GET` | `/prestamos/{id}` | Consulta el detalle de un préstamo | `200`, `404` |
| `POST` | `/prestamos/{id}/devolucion` | Registra la devolución, repone stock y calcula multa | `200`, `404`, `409` |
| `GET` | `/socios/{id}/prestamos?activos=true` | Lista los préstamos activos del socio | `200`, `404` |

---

## 🚀 Instalación y Ejecución

### 1. Clonar el repositorio

```bash
git clone https://github.com/IvanClar1a/UTN-TP2-PROGRAMACION-IV.git
cd UTN-TP2-PROGRAMACION-IV
```

### 2. Crear y activar el entorno virtual

```bash
python3 -m venv .venv
```

- **Linux / macOS:**
  ```bash
  source .venv/bin/activate
  ```
- **Windows (PowerShell):**
  ```powershell
  .\.venv\Scripts\Activate.ps1
  ```

### 3. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 4. Levantar el servidor

```bash
uvicorn main:app --reload
```

Una vez iniciado, accedé a la documentación interactiva en:
- **Swagger UI:** [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- **ReDoc:** [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

---

## 🧪 Pruebas y Cobertura

Los tests utilizan una base de datos **SQLite en memoria** aislada por cada prueba (`sqlite://` + `StaticPool`), sin modificar el archivo `biblioteca.db` local.

- **Ejecutar todos los tests:**
  ```bash
  pytest -v
  ```

- **Ejecutar con reporte de cobertura (objetivo >= 80%):**
  ```bash
  pytest --cov=. --cov-report=term-missing
  ```

---

## 👤 Autor

**Iván Clariá** — [GitHub (@IvanClar1a)](https://github.com/IvanClar1a)