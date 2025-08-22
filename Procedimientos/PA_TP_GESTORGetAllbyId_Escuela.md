### PA_TP_GESTORGetAllbyId_Escuela

Procedimiento específico del proyecto BPM4US que obtiene gestores por escuela asociada a un caso BPM. Utiliza el número de caso para encontrar el caso padre, luego busca la escuela correspondiente y retorna los gestores con información completa del usuario desde el servidor remoto.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: PA_TP_GESTORGetAllbyId_Escuela] --> B[Recibe @NumeroCaso]
    B --> C[Declara @NumCasoPadre]
    C --> D[EXEC PA_BPMCasoPadre<br/>para obtener caso padre]
    D --> E{¿@NumCasoPadre es NULL?}
    
    E -->|Sí| F[Asigna @NumCasoPadre = @NumeroCaso]
    E -->|No| G[Usa @NumCasoPadre obtenido]
    
    F --> H[JOIN TP_GESTOR + ASPNETUSERS remoto + TM_MaestroUnificado]
    G --> H
    H --> I[WHERE Estado = 1 AND NumeroCaso = @NumCasoPadre]
    I --> J[Retorna gestores con datos completos]
    J --> K[Fin]
    
    style A fill:#e1f5fe
    style K fill:#e8f5e8
    style E fill:#fff3e0
    style H fill:#f3e5f5
    style J fill:#e8f5e8
```
#### Procedimiento almacenado
```sql

-- |
-- |PA_TP_GESTORGetAllbyId_Escuela|/_
-- Empresa: TiGlobal SAS
-- Procedimiento: [CUN].[PA_TP_GESTORGetAllbyId_Escuela]
-- Creado Por: María Cristina Díaz Torres
-- Fecha: Aug 12 2024 9:55AM
-- Proyecto: Bpm4UsCun
-- Descripcion: Dado el número del caso, consulta el maestro unificado para tomar la escuela y trae los Gestores con
--sus nombres.
-- [CUN].[PA_TP_GESTORGetAllbyId_Escuela] '000000003655 '

    */

CREATE Procedure [CUN].[PA_TP_GESTORGetAllbyId_Escuela] @NumeroCaso varchar(250)
AS
Set Nocount ON
----------------------------------------------
Declare @NumCasoPadre varchar(250)
----------------------------------------------
BEGIN
----------------------------------------------
exec [CUN].[PA_BPMCasoPadre] @NumeroCaso, @NumCasoPadre output
if isnull(@NumCasoPadre,'') = ''
set @NumCasoPadre = @NumeroCaso
-----------------------------------------------
Select ges.Id ,
ges.Id_Escuela ,
ges.Id_gestor collate Modern_Spanish_CI_AS Id_gestor,
USR.UserName,
usr.PrimerNombre firstName,
usr.SegundoNombre MiddleName,
usr.PrimerApellido LastName,
usr.SegundoApellido SecondSurname  
 From CUN.TP_GESTOR ges
inner join [172.16.41.4\SQLEXPRESS].[Bpm4usCun].DBO.ASPNETUSERS USR on GES.Id_gestor collate Modern_Spanish_CI_AS = USR.ID
inner join CUN.TM_MaestroUnificado muf on ges.id_Escuela = muf.IdEscuela
Where GES.Estado = 1
and muf.NumeroCaso = @NumCasoPadre
order by 1 desc
-----------------------------------------
END
```
#### Operaciones Principales

- Búsqueda de caso padre: Ejecuta PA_BPMCasoPadre para navegar en jerarquía BPM
- Fallback de caso: Si no hay caso padre, usa el caso original
- JOIN complejo: Combina gestores locales con usuarios remotos y maestro unificado
- Datos completos: Retorna información detallada del gestor y usuario

#### Tablas afectadas

- [CUN].[TP_GESTOR]: Tabla principal de gestores (lectura)
- [CUN].[TM_MaestroUnificado]: Maestro unificado de casos BPM (lectura)
- [172.16.41.4\SQLEXPRESS].[Bpm4usCun].DBO.ASPNETUSERS: Usuarios remotos (lectura)

#### Procedimientos Almacenados Anidados

- [CUN].[PA_BPMCasoPadre]: Procedimiento para obtener el caso padre del número de caso proporcionado