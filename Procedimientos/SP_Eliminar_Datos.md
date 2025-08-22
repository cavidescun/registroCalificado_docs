

### SP_Eliminar_Datos

Procedimiento dinámico que elimina registros de múltiples tablas basándose en configuración centralizada. Construye automáticamente sentencias DELETE para todas las tablas registradas en la tabla de configuración, ejecutando eliminación masiva por TR_SNIES específico.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: SP_Eliminar_Datos] --> B[Recibe @TR_SNIES]
    B --> C[DECLARE @SQL para construcción dinámica]
    C --> D[SELECT desde RCAL.Configuracion_Tablas]
    D --> E[Concatenar DELETE por cada Tabla_Origen]
    E --> F[Agregar WHERE TR_SNIES = parámetro]
    F --> G[PRINT @SQL para depuración]
    G --> H[EXEC sp_executesql con SQL dinámico]
    H --> I[Fin]
    
    style A fill:#e1f5fe
    style D fill:#fff3e0
    style H fill:#f3e5f5
    style I fill:#e8f5e8
```
#### Procedimiento almacenado
```sql
CREATE PROCEDURE [RCAL].[SP_Eliminar_Datos]
@TR_SNIES VARCHAR(50)
AS
BEGIN
SET NOCOUNT ON;

    DECLARE @SQL NVARCHAR(MAX) = '';

    -- Construcción dinámica de los DELETE
    SELECT @SQL = @SQL +
        'DELETE FROM ' + Tabla_Origen + ' WHERE TR_SNIES = ''' + @TR_SNIES + '''; ' + CHAR(13)
    FROM RCAL.Configuracion_Tablas

    -- Ejecutar el SQL dinámico
    PRINT @SQL;  -- Para depuración, muestra los DELETE generados
    EXEC sp_executesql @SQL;

END;
```
#### Operaciones Principales

- Construcción dinámica: SELECT concatena múltiples DELETE desde tabla configuración
- Parametrización: Inyecta @TR_SNIES en cada sentencia WHERE generada
- Concatenación: += para agregar cada DELETE con CHAR(13) como separador
- Depuración: PRINT muestra SQL generado antes de ejecución
- Ejecución masiva: sp_executesql ejecuta todos los DELETE concatenados
- Eliminación controlada: Solo registros con TR_SNIES específico

#### Tablas afectadas

##### Eliminadas (dinámicamente):

- Múltiples tablas: Según configuración en RCAL.Configuracion_Tablas (campo Tabla_Origen)

##### Consultadas:

- RCAL.Configuracion_Tablas: Tabla de configuración con listado de tablas objetivo

#### Procedimientos Almacenados Anidados

- sp_executesql: Ejecuta SQL dinámico construido con múltiples DELETE concatenados