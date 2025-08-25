### sp_AddObservacionExpertoDirector

Procedimiento especializado que registra observaciones diferenciadas por tipo de usuario (Experto o Director) en el sistema BPM. Procesa observaciones JSON, determina el origen según el flag @pEsExperto y actualiza la traza de observaciones en el maestro unificado con categorización específica.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: sp_AddObservacionExpertoDirector] --> B[Recibe NumCaso, NombreUsuario, EsExperto, Observaciones]
    B --> C{¿Existe registro en TM_MaestroUnificado?}
    C -->|No| D[INSERT nuevo registro con NumeroCaso]
    C -->|Sí| E[Continuar con existente]
    D --> E
    E --> F[SELECT Id, IdDirector, Director del maestro]
    F --> G[Configurar Condición 'Introducción' e IdFormulario 49]
    G --> H[Procesar JSON observaciones con OPENJSON]
    H --> I{¿ @pEsExperto = 1?}
    I -->|Sí| J[Usuario = @pNombreUsuario, Condición = 'Observacion Experto']
    I -->|No| K[Usuario = @DirectorNombre, Condición = 'Observacion Final Director']
    J --> L[INSERT en TM_Observaciones con datos diferenciados]
    K --> L
    L --> M[UPDATE trazaDeObservaciones con FOR JSON PATH]
    M --> N[SELECT registro actualizado]
    N --> O[Fin]
    
    style A fill:#e1f5fe
    style I fill:#fff3e0
    style L fill:#f3e5f5
    style O fill:#e8f5e8
```
#### Procedimiento almacenado
```sql
/*--=================================================================================================================================================================
Author: María Cristina Díaz Torres
Create Date: 15/08/2024
Description: Procedimiento almacenado para el llamado de observaciones de los subprocesos (c/u de las condiciones) y realizar almacenamiento en TM_Observaciones
Se realiza además el proceso de actualización de los campos que requiera actualizar en cada condición en TM_MaestroUnificado
Este sp hace un llamado a CUN.sp_ConsultaObservaciones de la db BPM4UsCun
Version: 01
Modificado por: María Cristina Díaz Torres
Fecha Modificacion: 20240904
Observacion: Se elimina los llamados de procedimientos a través del link server.
Version: 02

EXEC [CUN].[sp_MaestroObsSubprocC10] '000000002088'
EXEC [CUN].[sp_MaestroObsSubprocC10] '000000002098'

--=================================================================================================================================================================\*/

CREATE Procedure [CUN].[sp_AddObservacionExpertoDirector] @pNumCaso VARCHAR(250), @pNombreUsuario varchar(500),  
 @pEsExperto bit, @pObservaciones varchar(max)

AS
Set Nocount ON

---

-- DECLARACION DE TABLAS

---

CREATE TABLE #ListaObservaciones (IdFormulario INT
,NumCaso VARCHAR(250)
,Condicion VARCHAR(250)
,API VARCHAR(200)
,Nombre VARCHAR(500)
,FchObservacion DATETIME
,Observaciones VARCHAR(MAX)
,Usuario VARCHAR(200));

---

-- DECLARACION DE VARIABLES

---

DECLARE @NumCasoSubP VARCHAR(250)  
 DECLARE @Id_TM_MaestroUnificado INT
DECLARE @Condicion VARCHAR(MAX)
DECLARE @IdFormulario int
declare @Id_Director nvarchar(450)
Declare @DirectorNombre varchar(300)
--------------------------------------------------------------
BEGIN -- PROCEDIMIENTO

---

-- Inserta solo si no existe un registro con el mismo NumeroCaso
IF NOT EXISTS ( SELECT 1
FROM [CUN].[TM_MaestroUnificado]
WHERE NumeroCaso = @pNumCaso)
BEGIN
INSERT INTO [CUN].[TM_MaestroUnificado] (NumeroCaso)
VALUES (@pNumCaso);
END
-- director con nombre de la tabla Maestro unificado
-- Captura el Id de maestro unificado a partir del número de caso
SELECT @Id_TM_MaestroUnificado = Id
,@Id_Director = IdDirector
,@DirectorNombre = Director
FROM [CUN].[TM_MaestroUnificado]
WHERE NumeroCaso = @pNumCaso

---

Set @Condicion = 'Introducción'
Set @IdFormulario = 49
--Almacena en la tabla Observaciones los datos enviados por parámetro
INSERT INTO [CUN].[TM_Observaciones] (Fecha,Usuario,Condicion,Observacion,Estado,Auditoria,IdFormulario,Id_TM_MaestroUnificado)
SELECT CONVERT(datetime, SWITCHOFFSET(CONVERT(datetimeoffset, fechaDeLaObservacion), DATENAME(TzOffset, SYSDATETIMEOFFSET())))
,Case when @pEsExperto = 1
then @pNombreUsuario
else @DirectorNombre
end
,Case when @pEsExperto = 1
then 'Observacion Experto'
else 'Observacion Final Director'
end
,observacionesH
,1 estado
,'CUN.sp_AddObservacionExpertoDirector: '+@pNumCaso+convert(varchar(200), getdate(), 121) Auditoria
,@IdFormulario
,@Id_TM_MaestroUnificado
FROM OPENJSON(CASE WHEN ISJSON(@pObservaciones) = 1 THEN @pObservaciones ELSE '[]' END) --valida que el campo C.Valor sea un JSON
WITH (personalORolQueHizoLaObservacion NVARCHAR(MAX) '$.personalORolQueHizoLaObservacion',
  		  fechaDeLaObservacion             NVARCHAR(MAX) '$.fechaDeLaObservacion',
observacionesH NVARCHAR(MAX) '$.observacionesH',
  		  personaObservacion               NVARCHAR(MAX) '$.personaObservacion')
--Proceso de actualización campos editables condicion 1
UPDATE A
SET A.trazaDeObservaciones = (SELECT B.Fecha AS fechaDeLaObservacion
,B.Usuario AS personaObservacion
,B.Condicion AS condicionOFormularioDeOrigen
,B.Observacion AS observacionesH
FROM CUN.TM_Observaciones B
WHERE B.Id_TM_MaestroUnificado = A.Id
FOR JSON PATH )
FROM CUN.TM_MaestroUnificado A
WHERE A.NumeroCaso = @pNumCaso

---

SELECT \*
FROM CUN.TM_MaestroUnificado A
WHERE A.NumeroCaso = @pNumCaso;

---

END; --- FINAL PROCEDIMIENTO
```
#### Operaciones Principales

- Gestión registro: Inserta en TM_MaestroUnificado si no existe el caso
- Diferenciación usuarios: CASE WHEN determina Usuario y Condición según @pEsExperto
- Procesamiento JSON: OPENJSON extrae observaciones con estructura estándar
- Categorización: 'Observacion Experto' vs 'Observacion Final Director'
- Inserción diferenciada: TM_Observaciones con metadatos específicos por tipo usuario
- Consolidación traza: UPDATE trazaDeObservaciones con histórico JSON completo

#### Tablas afectadas

##### Actualizadas:

- CUN.TM_MaestroUnificado: Campos trazaDeObservaciones y registro base si no existe
- CUN.TM_Observaciones: Inserción de observaciones categorizadas por tipo usuario

#### Procedimientos Almacenados Anidados