use DBTarjeta
go

/* ===============================================================================================================================         
NOMBRE            : spu_BI_afiliacion_Compras_internet_Yape  
PROPOSITO         : Reporte de afiliaciones a compras por internet y afiliacion YAPE  
USADO POR         : iBusiness  
FECHA CREACION    : 30/09/2020 USUARIO: NRDE  
  
EJECUTAR EN       : DBTarjeta  
FECHA MODIFICACION: 05/10/2020 nrde se agregar fecha de afiliacion YAPE  
     22/10/2020 INICIO NRDE se agrega filtro para diferenciar APP movil, call center,ventanilla  
     22/10/2020 NRDE se agrega with(nolock)  
     22/10/2020 NRDE se modifica validacion rango de fecha COMPRAS INTERNET  
     22/10/2020 NRDE se modifica validacion rango de fecha YAPE  
     22/10/2020 NRDE se agrega diferenciador de agencia actual APP MOVIL CALL CENTER  
     22/10/2020 NRDE se cambia el filtro de agencias   
     22/10/2020 NRDE se cambia el rango de fechas, el filtro se aplica a fecha afiliacion YAPE  
     27/10/2020 NRDE se comenta validacion de fecha de afiliacion a compras por internet, se consideran todas las afiliaciones a compras por internet  
     05/11/2020 NRDE se cambia usuario, colaborador, cargo para afiliacion por App Movil  
     07/12/2020 NRDE se modifica por rotacion de colaborador (PALS por MMAR) para call center   
     29/12/2020 NRDE se modifica por rotacion de colaborador (CVAP regresa a agencia San Sebastian)  
	 26/01/2021 NRDE se agrega validacion de afiliacion a compras desde ventanilla para colaboradores temporales que regresaron a sus agencias
	 06/12/2021 HSPC Se modifica la seleccion de resultados. Se comenta la agrupacion de los resultados
	 06/01/2022 HSPC Se hace la regresion del script a estado anterior de la fecha 26/01/2021 NRDE
  
MODO EJECUCION    : EXEC dbo.spu_BI_afiliacion_Compras_internet_Yape '20200128','20201022','00D','' --APP MOVIL  
     EXEC dbo.spu_BI_afiliacion_Compras_internet_Yape '20200128','20201022','00B','' --Ventanilla  
     EXEC dbo.spu_BI_afiliacion_Compras_internet_Yape '20200128','20201022','00C','' --Call Center  
     EXEC dbo.spu_BI_afiliacion_Compras_internet_Yape '20200128','20201022','00A','' --TODOS  
  
================================================================================================================================= */

ALTER PROCEDURE [dbo].[spu_BI_afiliacion_Compras_internet_Yape]
    @dFechaInicio DATETIME,
    @dFechaFin DATETIME,
    @cAgeCods VARCHAR(600) = '',
    @cUser varchar(4) = ''
as
begin
    set nocount on

    --declare @dFechaInicio DATETIME='20201001'  
    --declare @dFechaFin DATETIME='20201030'  
    --declare @cuser varchar(4)=''  
    ----declare @dFechaFin DATETIME=getdate()  
    ----declare @cAgencia VARCHAR(4) = '01'  
    --declare @cAgeCods VARCHAR(600) = '00D'  
    DECLARE @Agencias table (cAgeCod varchar(3) primary key)
    -- 22/10/2020 INICIO NRDE se agrega filtro para diferenciar APP movil, call center,ventanilla  
    declare @ncanal int

    if @cAgeCods = '00D' --App Movil  
    begin
        INSERT INTO @Agencias
        SELECT cAgeCod
        FROM dbcmac..agencias with (nolock)
        set @ncanal = 91
    end
    else if @cAgeCods = '00B' -- Ventanilla  
    begin
        INSERT INTO @Agencias
        SELECT cAgeCod
        FROM dbcmac..agencias with (nolock)
        set @ncanal = 1
    end
    else if @cAgeCods = '00C' --Call Center  
    begin
        INSERT INTO @Agencias
        SELECT cAgeCod
        FROM dbcmac..agencias with (nolock)
        set @ncanal = 2
    end
    else if @cAgeCods = '00A' -- Todos  
    begin
        INSERT INTO @Agencias
        SELECT cAgeCod
        FROM dbcmac..agencias with (nolock)
        set @ncanal = 0
    end
    else
    begin
        INSERT INTO @Agencias
        SELECT *
        FROM dbcmac.dbo.fn_cadenatabla(@cAgeCods, ',')
        set @ncanal = 0
    end

    --INSERT INTO @Agencias    
    --SELECT cagecod FROM dbcmac.dbo.fn_cadenatabla(@cAgeCods, ',')  

    -- 22/10/2020 FIN NRDE se agrega filtro para diferenciar APP movil, call center,ventanilla  
    DECLARE @dFechaFinAux datetime = DATEADD(d, 1, CONVERT(DATETIME, CONVERT(VARCHAR(10), @dFechaFin, 112)))

    select distinct
        (cnumtarjeta),
        max(dfecha) dfecha
    into #tempafilCOMPRAS
    from Log_TarjetaCuenta with (nolock) -- 22/10/2020 NRDE se agrega with(nolock)  
    --where dFecha>=@dFechaInicio and dfecha<@dFechaFinAux  
    --where dFecha>=@dFechaInicio and dfecha<=@dFechaFinAux -- 22/10/2020 NRDE se modifica validacion rango de fecha COMPRAS INTERNET  
    -- 27/10/2020 NRDE se comenta validacion de fecha de afiliacion a compras por internet, se consideran todas las afiliaciones a compras por internet  
    where bComprasPorInternet = 1
          and nprior = 1
    group by cnumtarjeta

    --select distinct (cpan)  collate Modern_Spanish_CI_AS'cpan',max(dFechaHora) dfechahora -- 05/10/2020 nrde se agregar fecha de afiliacion YAPE 
    --select	cpan collate Modern_Spanish_CI_AS 'cpan', dFechaHora --06/12/2021 HSPC Se comento la linea de arriba para obtener todas la afiliaciones en sus fechas respectivas (este retroceso se hizo en coordinacion y confimacion con Gary)
    select distinct
        (cpan) collate Modern_Spanish_CI_AS 'cpan',
        max(dFechaHora) dfechahora --06/01/2022 HSPC Se vuelve a poner el cambio de la fecha 05/10/2020 nrde
    into #tmpafilYAPE
    from atmmov atm with (nolock)
        left join DBCMAC..movref mr with (nolock)
            on atm.nmovnro = mr.nMovNroRef
    where ccardlocation like '%yape%'
          and nMovNroRef is null
          and atm.cPrCode = '001000'
          and nMontoTran = 0
          --and dfechaHora>=@dFechaInicio and dfechaHora<@dFechaFinAux   
          and dfechaHora >= @dFechaInicio
          and dfechaHora <= @dFechaFinAux -- 22/10/2020 NRDE se modifica validacion rango de fecha YAPE  
    --group by cPan -- 05/10/2020 nrde se agregar fecha de afiliacion YAPE
    -- 06/12/2021 HSPC Se comenta la linea de arriba para no agrupar los resultados
    group by cPan -- 06/01/2022 HSPC Se vuelve a póner el cambio de la fecha 05/10/2020 nrde

    select 'REPORTE AFILIACIoN COMPRAS POR INTERNET-YAPE' cTitulo,
           pe.cPersNombre nombreCliente,
           left(ltc.cnumtarjeta, 6) + '******' + right(ltc.cnumtarjeta, 4) cNumTarjeta,
           case cPersIDTpo
               when 1 then
                   'DNI'
               else
                   'Otro'
           End TipoDoc,
           pid.cPersIDnro dniCliente,
           ltc.dfecha fechaAfil,
           dfechahora fechaAfilYAPE, -- 05/10/2020 nrde se agregar fecha de afiliacion YAPE  
           pe.cPersTelefono2 telefono,
           pe.cPersEmail correo,
                                     --ltc.ccoduser usuario,  
           case
               when ltc.nCanalComprasInternet = 91 then
                   'APP1'
               else
                   ltc.ccoduser
           end usuario,              -- 05/11/2020 NRDE se cambia usuario, colaborador, cargo para afiliacion por App Movil  
                                     --isnull(prh.cPersNombre,'') colaborador,  
           case
               when ltc.nCanalComprasInternet = 91 then
                   ''
               else
                   isnull(prh.cPersNombre, '')
           end colaborador,          -- 05/11/2020 NRDE se cambia usuario, colaborador, cargo para afiliacion por App Movil  
           case
               when ltc.nCanalComprasInternet = 91 then
                   ''
               else
                   isnull(
                   (
                       select top 1
                           cRHCargoDescripcion
                       from DBCMAC..rrhh RR with (nolock) -- 05/11/2020 NRDE se cambia usuario, colaborador, cargo para afiliacion por App Movil  
                           inner join DBCMAC..RHCargos RC WITH (NOLOCK)
                               ON RC.cPersCod = RR.cPersCod
                           inner join DBCMAC..RHCargosTabla RT WITH (NOLOCK)
                               ON RC.cRHCargoCod = RT.cRHCargoCod
                       where RR.cuser = RH.cUser
                       order by dRHCargoFecha desc
                   ),
                   ''
                         )
           end as Cargo,
                                     --A.cAgeCod as codAgencia,  
           case
               when ltc.nCanalComprasInternet = 91 then
                   '01'
               else
                   A.cAgeCod
           end codAgencia,           -- 05/11/2020 NRDE se cambia usuario, colaborador, cargo para afiliacion por App Movil  
                                     -- 22/10/2020 INICIO NRDE se agrega diferenciador de agencia actual APP MOVIL CALL CENTER  
                                     --isnull(A.cAgeDescripcion,'') as cAgenciaActual,  
           case
               when isnull(ltc.nCanalComprasInternet, 1) = 1 then
                   isnull(A.cAgeDescripcion, '')
               when ltc.nCanalComprasInternet = 2 then
                   A.cAgeDescripcion -- 26/01/2021 NRDE se agrega validacion de afiliacion a compras desde ventanilla para colaboradores temporales que regresaron a sus agencias
               --when ltc.ccoduser in('PMEV','CVIE','BRRA','CTEL','MMAR','CBJS','VCIV','CVAP') then 'CALL CENTER' -- 8 colaboradores que pertenecen a call center  
               --when ltc.ccoduser in('PMEV','CVIE','BRRA','CTEL','PALS','CBJS','VCIV','CVAP') then 'CALL CENTER' -- 07/12/2020 NRDE se modifica por rotacion de colaborador (PALS por MMAR) para call center   
               when ltc.ccoduser in ( 'PMEV', 'CVIE', 'BRRA', 'CTEL', 'PALS', 'CBJS', 'VCIV' ) then
                   'CALL CENTER'     -- 29/12/2020 NRDE se modifica por rotacion de colaborador (CVAP regresa a agencia San Sebastian)  

               when ltc.nCanalComprasInternet = 91 then
                   'APP MOVIL'
           end as cAgenciaActual
    -- 22/10/2020 FIN NRDE se agrega diferenciador de agencia actual  

    from #tempafilCOMPRAS TAC
        inner join log_tarjetacuenta ltc with (nolock)
            on tac.cnumtarjeta = ltc.cnumtarjeta
               and tac.dfecha = ltc.dfecha
        inner join #tmpafilYAPE TAY
            on tac.cnumtarjeta = tay.cpan collate Latin1_General_CI_AS
        inner join tarjeta TA with (nolock)
            on ta.cNumTarjeta = ltc.cNumTarjeta collate Latin1_General_CI_AS
        inner join dbcmac..persona pe with (nolock)
            on ta.cPersCod = pe.cPersCod
        inner join DBCMAC..PersID pid with (nolock)
            on pe.cPersCod = pid.cPersCod
               and pid.cPersIDTpo = 1
        LEFT join DBCMAC..rrhh RH with (nolock)
            on rh.cUser = ltc.ccoduser collate Latin1_General_CI_AS
               and rh.nRHEstado < 700
        LEFT join dbcmac..persona pRH with (nolock)
            on RH.cPersCod = pRH.cPersCod
        INNER JOIN DBCMAC..Agencias A WITH (NOLOCK)
            ON LTC.cCodAge = A.cAgeCod collate Latin1_General_CI_AS
               AND (
                       (
                           ltc.ccoduser = @cUser
                           and @cUser <> ''
                       )
                       or (@cUser = '')
                   )
               --and (isnull(A.cAgeCod,'01') in (select cAgeCod COLLATE SQL_Latin1_General_CP1_CI_AS from @Agencias)))  
               AND (
                       (isnull(ltc.NCANALcOMPRASINTERNET, 1) = @ncanal)
                       or (@ncanal = 0)
                          and isnull(a.cAgeCod, '01') in (
                                                             select cAgeCod COLLATE SQL_Latin1_General_CP1_CI_AS from @Agencias
                                                         )
                   ) -- 22/10/2020 NRDE se cambia el filtro de agencias   
    --order by ltc.dfecha desc  
    order by dfechahora desc -- 22/10/2020 NRDE se cambia el rango de fechas, el filtro se aplica a fecha afiliacion YAPE  

    drop table #tempafilCOMPRAS
    drop table #tmpafilYAPE

    set nocount off
end
