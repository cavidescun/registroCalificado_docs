### sp_MaestroObsSubprocC5

Procedimiento especializado para procesar observaciones y actualizaciones de la Condición 5 (Investigación) en el sistema BPM. Maneja observaciones JSON y actualiza 11 campos editables específicos relacionados con grupos de investigación, líneas de investigación, producción académica y planes investigativos.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: sp_MaestroObsSubprocC5] --> B[Recibe parámetros C5 + observaciones]
    B --> C{¿NumCasoPadre válido?}
    C -->|No| D[RAISERROR - Sin número caso]
    C -->|Sí| E[Conservar NumCasoSubP y asignar NumCaso]
    E --> F[UPDATE AccionPorCondicion si cumple condiciones]
    F --> G{¿Existe registro en TM_MaestroUnificado?}
    G -->|No| H[INSERT nuevo registro]
    G -->|Sí| I[Continuar con existente]
    H --> I
    I --> J[SELECT Id, IdDirector, Director]
    J --> K[Configurar Condición 5 e IdFormulario 18]
    K --> L[Procesar JSON observaciones con OPENJSON]
    L --> M[INSERT en TM_Observaciones]
    M --> N[UPDATE 11 campos editables C5]
    N --> O[UPDATE trazaDeObservaciones con FOR JSON PATH]
    O --> P[SELECT registro actualizado]
    P --> Q[Fin]
    
    style A fill:#e1f5fe
    style D fill:#ffcdd2
    style L fill:#f3e5f5
    style Q fill:#e8f5e8
```
#### Procedimiento almacenado
```sql
/*
|sp_MaestroObsSubprocC5|/\*--=================================================================================================================================================================
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

EXEC [CUN].[sp_MaestroObsSubprocC5] '000000002088'
EXEC [CUN].[sp_MaestroObsSubprocC5] '000000002098'

--=================================================================================================================================================================\*/

CREATE Procedure [CUN].[sp_MaestroObsSubprocC5] @pNumCaso VARCHAR(250), @pNumCasoPadre varchar(250), @pAccionPorCondicion varchar(max),
@pC5Edit_gruposInvestigacionInstitucionales VARCHAR(MAX),
@pC5Edit_analisisGruposInvestigacion VARCHAR(MAX),
@pC5Edit_lineasDeInvestigacion VARCHAR(MAX),
@pC5Edit_investigacionDeLaEscuela VARCHAR(MAX),
@pC5Edit_investigacionPrograma VARCHAR(MAX),
@pC5Edit_grupoInvestigacion VARCHAR(MAX),
@pC5Edit_planDeInvestigacionDelPrograma VARCHAR(MAX),
@pC5Edit_inversionesEstimadas1 VARCHAR(MAX),
@pC5Edit_resumenInvsProgram VARCHAR(MAX),
@pC5Edit_produccionUltimos VARCHAR(MAX),
@pC5Edit_tresProfesores VARCHAR(MAX),
@pObservaciones varchar(max)
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

IF ISNULL (@pNumCasoPadre, '') = ''
BEGIN
RAISERROR ('No existe número de caso', 18, 18);
RETURN;
END
--Conservar el número de caso del subproceso para las actualizaciones de campos editables
SET @NumCasoSubP = @pNumCaso
set @pNumCaso = @pNumCasoPadre
--Actualización en MaestroUnificado del campo AccionPorCondicion
UPDATE [CUN].[TM_MaestroUnificado]
SET AccionPorCondicion = @pAccionPorCondicion
WHERE NumeroCaso = @pNumCaso
and @pAccionPorCondicion < 4
AND @pAccionPorCondicion > isnull(AccionPorCondicion,0);

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

Set @Condicion = 'Condición 5 - Investigación ' --C5_RevPDF
Set @IdFormulario = 18
--Almacena en la tabla Observaciones los datos enviados por parámetro
INSERT INTO [CUN].[TM_Observaciones] (Fecha,Usuario,Condicion,Observacion,Estado,Auditoria,IdFormulario,Id_TM_MaestroUnificado)
SELECT CONVERT(datetime, SWITCHOFFSET(CONVERT(datetimeoffset, fechaDeLaObservacion), DATENAME(TzOffset, SYSDATETIMEOFFSET())))
,@DirectorNombre
,@Condicion
,observacionesH
,1 estado
,'CUN.sp_MaestroObsSubprocC5: '+@pNumCaso+convert(varchar(200), getdate(), 121) Auditoria
,@IdFormulario
,@Id_TM_MaestroUnificado
FROM OPENJSON(CASE WHEN ISJSON(@pObservaciones) = 1 THEN @pObservaciones ELSE '[]' END) --valida que el campo C.Valor sea un JSON
WITH (personalORolQueHizoLaObservacion NVARCHAR(MAX) '$.personalORolQueHizoLaObservacion',
  		  fechaDeLaObservacion             NVARCHAR(MAX) '$.fechaDeLaObservacion',
observacionesH NVARCHAR(MAX) '$.observacionesH',
  		  personaObservacion               NVARCHAR(MAX) '$.personaObservacion')
--Proceso de actualización campos editables condicion 1
UPDATE TMU
SET TMU.gruposInvestigacionInstitucionales = isnull(@pC5Edit_gruposInvestigacionInstitucionales ,TMU.gruposInvestigacionInstitucionales)
,TMU.analisisGruposInvestigacion = isnull(@pC5Edit_analisisGruposInvestigacion ,TMU.analisisGruposInvestigacion )
,TMU.lineasDeInvestigacion = isnull(@pC5Edit_lineasDeInvestigacion ,TMU.lineasDeInvestigacion )
,TMU.investigacionDeLaEscuela = isnull(@pC5Edit_investigacionDeLaEscuela ,TMU.investigacionDeLaEscuela )
,TMU.investigacionPrograma = isnull(@pC5Edit_investigacionPrograma ,TMU.investigacionPrograma )
,TMU.grupoInvestigacion = isnull(@pC5Edit_grupoInvestigacion ,TMU.grupoInvestigacion )
,TMU.planDeInvestigacionDelPrograma = isnull(@pC5Edit_planDeInvestigacionDelPrograma ,TMU.planDeInvestigacionDelPrograma )
,TMU.inversionesEstimadas1 = isnull(@pC5Edit_inversionesEstimadas1 ,TMU.inversionesEstimadas1 )
,TMU.resumenInvsProgram = isnull(@pC5Edit_resumenInvsProgram ,TMU.resumenInvsProgram )
,TMU.produccionUltimos = isnull(@pC5Edit_produccionUltimos ,TMU.produccionUltimos )
,TMU.tresProfesores = isnull(@pC5Edit_tresProfesores ,TMU.tresProfesores )
,TMU.ObservaInvestigacionC5 = case when @pAccionPorCondicion = 2   
 then 1
else 0
end
,TMU.trazaDeObservaciones = (SELECT TOB.Fecha AS fechaDeLaObservacion
,TOB.Condicion AS condicionOFormularioDeOrigen
,TOB.Observacion AS observacionesH
FROM [CUN].[TM_Observaciones] TOB
WHERE TOB.Id_TM_MaestroUnificado = TMU.Id
FOR JSON PATH )
FROM [CUN].[TM_MaestroUnificado] TMU
WHERE NumeroCaso = @pNumCaso

---

SELECT TMU.\*
FROM [CUN].[TM_MaestroUnificado] TMU
WHERE TMU.NumeroCaso = @pNumCaso;

---

END;-- FIN PROCEDIMIENTO
```
#### Operaciones Principales

- Validación y configuración: Verifica número caso padre y conserva subproceso
- Actualización acción: Modifica AccionPorCondicion según condiciones específicas
- Gestión registro: Inserta en TM_MaestroUnificado si no existe
- Procesamiento observaciones: Parsea JSON y almacena en TM_Observaciones
- Actualización campos C5: Modifica 11 campos específicos de investigación
- Consolidación traza: Actualiza trazaDeObservaciones con histórico JSON

#### Tablas afectadas

##### Actualizadas:

- CUN.TM_MaestroUnificado: Campos AccionPorCondicion, gruposInvestigacionInstitucionales, analisisGruposInvestigacion, lineasDeInvestigacion, investigacionDeLaEscuela, investigacionPrograma, grupoInvestigacion, planDeInvestigacionDelPrograma, inversionesEstimadas1, resumenInvsProgram, produccionUltimos, tresProfesores, ObservaInvestigacionC5, trazaDeObservaciones
- CUN.TM_Observaciones: Inserción de observaciones procesadas

#### Procedimientos Almacenados Anidados