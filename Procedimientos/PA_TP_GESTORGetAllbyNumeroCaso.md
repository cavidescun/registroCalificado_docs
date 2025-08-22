### PA_TP_GESTORGetAllbyNumeroCaso

Procedimiento del proyecto BPM4US que obtiene gestores asociados a un caso específico o su caso padre. Utiliza el maestro unificado para relacionar casos con escuelas y retorna información básica de gestores, con flexibilidad para usar caso padre si está disponible.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: PA_TP_GESTORGetAllbyNumeroCaso] --> B[Recibe @NumeroCaso, @NumeroCasoPadre]
    B --> C[SET NOCOUNT ON]
    C --> D[JOIN TP_GESTOR + TM_MaestroUnificado]
    D --> E[WHERE Estado = 1]
    E --> F[AND NumeroCaso = ISNULL(@NumeroCasoPadre, @NumeroCaso)]
    F --> G[ORDER BY Id DESC]
    G --> H[Retorna gestores básicos]
    H --> I[Fin]
    
    style A fill:#e1f5fe
    style I fill:#e8f5e8
    style D fill:#f3e5f5
    style H fill:#e8f5e8
```
#### Procedimiento almacenado
```sql
-- |PA_TP_GESTORGetAllbyNumeroCaso|/_
-- Empresa: TiGlobal SAS
--Procedimiento: [CUN].[PA_TP_GESTORGetAllbyId_Escuela]
--Creado Por: María Cristina Díaz Torres
--Fecha: Aug 12 2024 9:55AM
--Proyecto: Bpm4UsCun
--Descripcion: Dado el número del caso, consulta el maestro unificado para tomar la escuela y trae los Gestores con
--sus nombres.
--[CUN].[PA_TP_GESTORGetAllbyNumeroCaso] '000000000636', '000000000637 '
--[CUN].[PA_TP_GESTORGetAllbyNumeroCaso]'000000007808 ', Null
_/
CREATE Procedure [CUN].[PA_TP_GESTORGetAllbyNumeroCaso] @NumeroCaso varchar(250), @NumeroCasoPadre varchar(250)
AS
Set Nocount ON
----------------------------------------------
--Declare @NumCasoPadre varchar(250)
----------------------------------------------
BEGIN
----------------------------------------------
--print '@NumCasoPadre1'
--print @NumeroCasoPadre
Select ges.Id ,
ges.Id_Escuela ,
ges.Id_gestor collate Modern_Spanish_CI_AS Id_gestor,  
 ges.nombreGestor Username
From CUN.TP_GESTOR ges
inner join CUN.TM_MaestroUnificado muf on ges.id_Escuela = muf.IdEscuela
Where GES.Estado = 1
and muf.NumeroCaso = isnull(@NumeroCasoPadre,@NumeroCaso)
order by 1 desc

---
END
```
#### Operaciones Principales

- Lógica de fallback: Usa ISNULL(@NumeroCasoPadre, @NumeroCaso) para determinar caso a buscar
- JOIN directo: Combina gestores con maestro unificado por escuela
- Filtro por estado: Solo gestores activos (Estado = 1)
- Datos básicos: Retorna información esencial del gestor sin datos remotos

#### Tablas afectadas

- [CUN].[TP_GESTOR]: Tabla principal de gestores (lectura)
- [CUN].[TM_MaestroUnificado]: Maestro unificado de casos BPM (lectura)

#### Procedimientos Almacenados Anidados