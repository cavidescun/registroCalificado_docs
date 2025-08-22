### Get_Infraestructura_Sede

Este procedimiento almacenado permite consultar información de infraestructura física de una sede específica o todas las sedes de Bogotá. Utiliza consultas dinámicas para seleccionar únicamente las columnas relevantes de la tabla Infraestructura_Fisica_Sedes basándose en el parámetro de sede proporcionado.

Si se especifica una sede que contiene "Bogotá", retorna todas las columnas relacionadas con Bogotá. Para otras sedes, busca la columna específica correspondiente. El procedimiento incluye validación de existencia de columnas y manejo de errores.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: Get_Infraestructura_Sede] --> B[Recibe @Sede como parámetro]
    B --> C[Declara variables SQL dinámico]
    C --> D{¿@Sede contiene 'Bogot'?}
    
    D -->|Sí| E[Busca todas las columnas<br/>que contengan 'Bogot%'<br/>usando STRING_AGG]
    D -->|No| F[Busca columna específica<br/>con nombre = @Sede]
    
    E --> G[Construye @columnList con<br/>todas las columnas de Bogotá]
    F --> H[Valida existencia de<br/>columna específica]
    
    G --> I{¿@columnList tiene valor?}
    H --> I
    
    I -->|Sí| J[Construye consulta SQL dinámica:<br/>SELECT Infraestructura_Física + columnas]
    I -->|No| K[Lanza RAISERROR:<br/>No se encontraron columnas]
    
    J --> L[Ejecuta consulta con sp_executesql]
    L --> M[Retorna datos de infraestructura<br/>para sede(s) especificada(s)]
    M --> N[Fin exitoso]
    
    K --> O[Fin con error]
    
    style A fill:#e1f5fe
    style N fill:#e8f5e8
    style O fill:#ffebee
    style J fill:#f3e5f5
    style K fill:#ffebee
    style D fill:#fff3e0
```

#### Procedimiento almacenado

```sql
CREATE PROCEDURE [rcal].[Get_Infraestructura_Sede]
@Sede NVARCHAR(100)
AS
BEGIN
DECLARE @sql NVARCHAR(MAX);
DECLARE @columnList NVARCHAR(MAX);

    IF @Sede LIKE '%Bogot%'
    BEGIN
        SELECT @columnList = STRING_AGG(QUOTENAME(name), ', ')
        FROM sys.columns
        WHERE object_id = OBJECT_ID('rcal.Infraestructura_Fisica_Sedes')
          AND name LIKE '%Bogot%';
    END
    ELSE
    BEGIN
        SELECT @columnList = QUOTENAME(@Sede)
        WHERE EXISTS (
            SELECT 1
            FROM sys.columns
            WHERE object_id = OBJECT_ID('rcal.Infraestructura_Fisica_Sedes')
              AND name = @Sede
        );
    END

    IF @columnList IS NOT NULL
    BEGIN
        SET @sql = '
            SELECT [Infraestructura_Física], ' + @columnList + '
            FROM [rcal].[Infraestructura_Fisica_Sedes];';

        EXEC sp_executesql @sql;
    END
    ELSE
    BEGIN
        RAISERROR('No se encontraron columnas que coincidan con la sede especificada.', 16, 1);
    END

END;
```
#### Lógica de Funcionamiento
##### Casos de consulta:

1. Sede de Bogotá (parámetro contiene "Bogot"):

- Busca todas las columnas que contengan "Bogot%" en su nombre
- Usa STRING_AGG para concatenar múltiples columnas
- Retorna infraestructura de todas las sedes bogotanas


2. Sede específica (cualquier otro valor):

- Busca una columna con el nombre exacto del parámetro
- Valida existencia antes de continuar
- Retorna infraestructura solo de esa sede

##### Estructura de consulta generada:
```sql
SELECT [Infraestructura_Física], [Columna1], [Columna2], ...
FROM [rcal].[Infraestructura_Fisica_Sedes]
```

##### Validaciones implementadas:

- Verificación de existencia de columnas en sys.columns
- Control de errores con RAISERROR si no se encuentran columnas
- Uso de QUOTENAME para prevenir inyección SQL

##### Tabla fuente: rcal.Infraestructura_Fisica_Sedes

##### Columnas retornadas:

- Infraestructura_Física: Descripción del elemento de infraestructura
- Columnas dinámicas según la sede consultada
