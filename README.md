# 📊 Sistema Generador de Reportes PDF - Admisiones Unillanos

Sistema automatizado para la generación de reportes estadísticos de admisiones universitarias en formato PDF, con consultas directas a base de datos MySQL y herramientas de carga masiva de datos.

## 🎯 Características Principales

- **Generación Automática de PDF**: Creación de documentos profesionales con tablas paginadas y encabezados repetidos
- **Consultas Optimizadas**: Agregaciones y agrupaciones eficientes sobre base de datos MySQL
- **Carga Masiva de Datos**: Scripts automatizados para importar +11,000 registros desde CSV
- **Soporte UTF-8 Completo**: Manejo correcto de caracteres especiales y tildes
- **Reportes Estadísticos**: 
  - Cantidad de admitidos discriminados por programa académico
  - Distribución de aspirantes por género (hombres/mujeres) en cada facultad

## 🛠️ Tecnologías

- **Java 11+**: Lenguaje principal del proyecto
- **Maven**: Gestión de dependencias y construcción
- **Apache PDFBox 2.0.30**: Generación de documentos PDF
- **MySQL Connector/J 8.4.0**: Conectividad con base de datos
- **Python 3.13+**: Scripts de procesamiento de datos
- **MySQL 8.0+**: Sistema de gestión de base de datos

## 📁 Estructura del Proyecto

```
Taller PDF/
├── src/main/java/com/taller/
│   ├── ReportesPDF.java          # Generador principal de reportes PDF
│   ├── PDFAplicacion.java        # Punto de entrada de la aplicación
│   └── PDF.java                  # Utilidades para creación de PDFs
├── scripts/
│   ├── create_and_load_unillanos.sql      # DDL y carga desde CSV
│   ├── generar_insert_admitidos.py        # Generador de INSERTs masivos
│   ├── insert_admitidos_unillanos.sql     # INSERTs para carga manual
│   └── views_unillanos.sql                # Vistas de análisis SQL
├── admitidos_unillanos.csv        # Dataset con 11,928 registros
├── pom.xml                        # Configuración Maven
└── README.md
```

## 🚀 Instalación y Configuración

### Prerrequisitos

- Java JDK 11 o superior
- Apache Maven 3.6+
- MySQL Server 8.0+
- Python 3.7+ (para scripts de generación)

### 1. Clonar el Repositorio

```bash
git clone https://github.com/tu-usuario/taller-pdf-admisiones.git
cd taller-pdf-admisiones
```

### 2. Configurar Base de Datos

#### Opción A: Carga desde CSV (Recomendado)

```sql
-- Ejecutar en MySQL Workbench o cliente MySQL
SET GLOBAL local_infile = 1;
SOURCE scripts/create_and_load_unillanos.sql;
SOURCE scripts/views_unillanos.sql;
```

#### Opción B: Carga con INSERTs

```bash
# Generar archivo de INSERTs
python scripts/generar_insert_admitidos.py

# Importar en MySQL
mysql -u root -p < scripts/insert_admitidos_unillanos.sql
mysql -u root -p < scripts/views_unillanos.sql
```

### 3. Configurar Credenciales

Editar `src/main/java/com/taller/ReportesPDF.java` si es necesario:

```java
String host = "localhost";
int port = 3306;
String db = "unillanos_admisiones";
String user = "root";
String pass = "1234";  // Cambiar por tu contraseña
```

### 4. Compilar el Proyecto

```bash
mvn clean package
```

## 💻 Uso

### Ejecutar desde Maven

```bash
mvn org.codehaus.mojo:exec-maven-plugin:3.5.0:java \
  -Dexec.mainClass=com.taller.PDFAplicacion \
  -Dexec.args="reporte_admisiones.pdf"
```

### Ejecutar desde IDE

1. Abrir `PDFAplicacion.java` en tu IDE
2. Ejecutar el método `main()`
3. El PDF se generará en la raíz del proyecto

### Ejecutar JAR Compilado

```bash
java -cp target/taller-pdf-1.0-SNAPSHOT.jar com.taller.PDFAplicacion reporte_admisiones.pdf
```

## 📊 Reportes Generados

### Reporte #1: Admitidos por Programa

Tabla con la cantidad total de estudiantes admitidos agrupados por programa académico, ordenados por cantidad (descendente) y nombre del programa.

**Consulta SQL:**
```sql
SELECT programa, COUNT(*) AS total 
FROM admitidos 
GROUP BY programa 
ORDER BY total DESC, programa;
```

### Reporte #2: Distribución por Género y Facultad

Tabla que muestra la cantidad de hombres, mujeres y total de aspirantes por cada facultad.

**Consulta SQL:**
```sql
SELECT facultad,
  SUM(CASE WHEN genero='MASCULINO' THEN 1 ELSE 0 END) AS hombres,
  SUM(CASE WHEN genero='FEMENINO' THEN 1 ELSE 0 END) AS mujeres
FROM admitidos 
GROUP BY facultad 
ORDER BY facultad;
```

## 🗃️ Esquema de Base de Datos

### Tabla: `admitidos`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | INT | Identificador único (autoincremental) |
| `anio` | SMALLINT | Año de admisión |
| `periodo` | TINYINT | Periodo académico |
| `tipo_hecho` | VARCHAR(20) | Tipo de evento |
| `genero` | VARCHAR(12) | Género del aspirante |
| `estado_civil` | VARCHAR(20) | Estado civil |
| `fecha_nacimiento` | DATE | Fecha de nacimiento |
| `pais` | VARCHAR(80) | País de origen |
| `ciudad_nacimiento` | VARCHAR(120) | Ciudad de nacimiento |
| `ciudad_residencia` | VARCHAR(120) | Ciudad de residencia |
| `departamento_colegio` | VARCHAR(120) | Departamento del colegio |
| `municipio_colegio` | VARCHAR(120) | Municipio del colegio |
| `colegio` | VARCHAR(200) | Nombre del colegio |
| `naturaleza_colegio` | VARCHAR(20) | Oficial/No Oficial |
| `snies` | INT | Código SNIES del programa |
| `programa` | VARCHAR(200) | Programa académico |
| `facultad` | VARCHAR(200) | Facultad |
| `modalidad` | VARCHAR(80) | Modalidad de estudio |
| `metodologia` | VARCHAR(80) | Metodología |
| `sede` | VARCHAR(200) | Sede universitaria |

**Índices:**
- PRIMARY KEY: `id`
- INDEX: `idx_anio` (anio)
- INDEX: `idx_programa` (programa)
- INDEX: `idx_facultad` (facultad)

## 🔧 Solución de Problemas

### Error: "Access denied for user"

Verificar credenciales MySQL en el código fuente.

### Error: "Table doesn't exist"

Ejecutar los scripts de creación de base de datos:
```bash
mysql -u root -p < scripts/create_and_load_unillanos.sql
```

### Error: "The used command is not allowed with this MySQL version"

Habilitar `local_infile` en MySQL:
```sql
SET GLOBAL local_infile = 1;
```

### PDF no se genera

Verificar permisos de escritura en el directorio de salida y que el servidor MySQL esté corriendo.

## 📈 Consultas de Análisis

El proyecto incluye vistas SQL predefinidas para análisis rápido:

```sql
-- Ver admitidos por año y periodo
SELECT * FROM v_admitidos_por_anio_periodo;

-- Ver admitidos por facultad y programa
SELECT * FROM v_admitidos_por_facultad_programa;

-- Ver top 20 colegios de origen
SELECT * FROM v_top_colegios;
```

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor:

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/nueva-funcionalidad`)
3. Commit tus cambios (`git commit -m 'Agrega nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/nueva-funcionalidad`)
5. Abre un Pull Request

## 📝 Licencia

Este proyecto es de código abierto y está disponible bajo la [Licencia MIT](LICENSE).

## 👥 Autor

- **Darwin Joan Aveiga Mora** - *Desarrollo inicial* - Corporacion Universitaria Minuto De Dios

## 📧 Contacto

Para preguntas o sugerencias, por favor abre un issue en el repositorio.

---

⭐ Si este proyecto te fue útil, considera darle una estrella en GitHub
