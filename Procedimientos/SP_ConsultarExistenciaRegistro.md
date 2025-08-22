### SP_ConsultarExistenciaRegistro

Procedimiento utilitario que verifica dinámicamente la existencia de registros por TR_SNIES en cualquier tabla del esquema RCAL. Utiliza SQL dinámico con parámetro OUTPUT para retornar resultado booleano, incluye manejo de errores y funcionalidad de debug.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: SP_ConsultarExistenciaRegistro] --> B[Recibe Identificador, TR_SNIES, @Existe OUTPUT]
    B --> C[BEGIN TRY para manejo errores]
    C --> D[SET @Existe = 0 inicialización]
    D --> E[Construir SQL dinámico con QUOTENAME]
    E --> F[PRINT @SQL para debug]
    F --> G[EXEC sp_executesql con parámetros OUTPUT]
    G --> H[EXISTS en tabla RCAL.@Identificador]
    H --> I[PRINT resultado para debug]
    I --> J[Return @Existe via OUTPUT]
    J --> K[Fin exitoso]
    
    C --> L[BEGIN CATCH]
    L --> M[SET @Existe = 0]
    M --> N[THROW error]
    N --> O[Fin con error]
    
    style A fill:#e1f5fe
    style E fill:#fff3e0
    style G fill:#f3e5f5
    style K fill:#e8f5e8
    style O fill:#ffcdd2
```
#### Procedimiento almacenado
```sql
CREATE PROCEDURE Dev.SP_ConsultarExistenciaRegistro
@Identificador NVARCHAR(100),
@TR_SNIES NVARCHAR(50),
@Existe BIT OUTPUT
AS
BEGIN
SET NOCOUNT ON;

    BEGIN TRY
        DECLARE @SQL NVARCHAR(MAX);
        SET @Existe = 0; -- Inicializar en falso

        SET @SQL = N'
            SELECT @ExisteOUT = CASE
                WHEN EXISTS (
                    SELECT 1
                    FROM RCAL.' + QUOTENAME(@Identificador) + ' WITH (NOLOCK)
                    WHERE TR_SNIES = @TR_SNIES_param
                )
                THEN 1
                ELSE 0
            END';

        -- Debug
        PRINT @SQL;

        EXEC sp_executesql
             @SQL,
             N'@TR_SNIES_param NVARCHAR(50), @ExisteOUT BIT OUTPUT',
             @TR_SNIES_param = @TR_SNIES,
             @ExisteOUT = @Existe OUTPUT;

        -- Debug
        PRINT 'Valor encontrado: ' + CAST(@Existe AS VARCHAR(1));
    END TRY
    BEGIN CATCH
        SET @Existe = 0;
        THROW;
    END CATCH

END
```
#### Operaciones Principales

- Inicialización: SET @Existe = 0 como valor por defecto
- Construcción dinámica: SQL con QUOTENAME para seguridad de nombres de tabla
- Consulta EXISTS: Verificación eficiente de existencia por TR_SNIES
- Ejecución parametrizada: sp_executesql con parámetros tipados y OUTPUT
- Debug integrado: PRINT para mostrar SQL generado y resultado
- Manejo errores: TRY/CATCH con THROW para propagación controlada

#### Tablas afectadas

##### Consultadas (dinámicamente):

- RCAL.@Identificador: Cualquier tabla del esquema RCAL según parámetro de entrada

#### Procedimientos Almacenados Anidados