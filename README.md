# Load
Repositorio temporal



/*
===============================================================================================================================
     
Nombre Procedimiento : dbo.SPU_Alerta_Observaciones_ELSE 
Funcionalidad        : alerta de observaciones ELSE 
USADO POR			 : JOB de Producción
Usuario              : CVVV - NRDE
Fecha creación       : 28/08/2020
Modificación		 : 05/04/2021 CVVV - Se agrega validación de sincronización de relojes con Hiper Center
Modificación		 : 29/09/2021 LVCH - Se agrega a Jimmy Keith para el seguimiento a los pagos ELSE
Modificación		 : 17/11/2021 HSPC - Se agrega a Flor Velasquez y pablo Huamani para el seguimiento a los pagos ELSE
Modo de ejecución    : dbo.SPU_Alerta_Observaciones_ELSE '20200828'

=================================================================================================================================
*/
ALTER PROCEDURE dbo.SPU_Alerta_Observaciones_ELSE
		@dFechaProceso date
as
begin 
set nocount off

	declare @dFechaIni DATE, 
			@Year int, 
			@Month varchar(10), 
			@Day int,
			@Html          varchar(max),          
			@HtmlPie			varchar(max),          
			@HtmlCabGeneral  varchar(max),          
			@Nro             int,          
			@HtmlDet   varchar(max),                
			@FVencimiento VARCHAR(max),            
			@psParaMail varchar(max),
			@psCopiaMail varchar(max),
			@pTituloMail VARCHAR(max),                      
			@pprofile_name VARCHAR(max),
			@nSincronizacion INT = 0--05/04/2021 CVVV - Se agrega validación de sincronización de relojes con Hiper Center

	  set @dFechaIni =dateadd(day,-1,@dFechaProceso)
	  declare @dFecFinal DATE =  @dFechaProceso

	-- cabecera correo
	set @Year=datepart(year,@dFechaIni)
	set @month =datepart(MONTH,@dFechaIni)
	set @day =datepart(day,@dFechaIni)
	-- cabecera correo

	IF( month(@dFechaIni)= 1) SET @month= 'ENERO' 
	IF( month(@dFechaIni)= 2) SET @month= 'FEBRERO' 
	IF( month(@dFechaIni)= 3) SET @month= 'MARZO' 
	IF( month(@dFechaIni)= 4) SET @month= 'ABRIL' 
	IF( month(@dFechaIni)= 5) SET @month= 'MAYO' 
	IF( month(@dFechaIni)= 6) SET @month= 'JUNIO' 
	IF( month(@dFechaIni)= 7) SET @month= 'JULIO' 
	IF( month(@dFechaIni)= 8) SET @month= 'AGOSTO' 
	IF( month(@dFechaIni)= 9) SET @month= 'SETIEMBRE' 
	IF( month(@dFechaIni)= 10) SET @month= 'OCTUBRE' 
	IF( month(@dFechaIni)= 11) SET @month= 'NOVIEMBRE' 
	IF( month(@dFechaIni)= 12) SET @month= 'DICIEMBRE'

	--05/04/2021 CVVV - Se agrega validación de sincronización de relojes con Hiper Center
	SELECT TOP 1 @nSincronizacion = datediff(SECOND,P.dFechaPago,R.cFechaHora)
	FROM PagoDeudaELSE P WITH(NOLOCK) inner join RespuestaPagoDeudaELSE R WITH(NOLOCK) ON P.nIDPago = R.nIDPago
	WHERE cCanal = '05'
	ORDER BY P.nIdPago DESC
	--05/04/2021 CVVV - Se agrega validación de sincronización de relojes con Hiper Center  

	SET @psParaMail = 'yrojas@cmac-cusco.com.pe'            
	--SET @psCopiaMail = 'vcabrera@cmac-cusco.com.pe;dneira@cmac-cusco.com.pe;clupo@cmac-cusco.com.pe;kalvarez@cmac-cusco.com.pe;muscamaita@cmac-cusco.com.pe'     --29/09/2021 LVCH         
	--SET @psCopiaMail = 'vcabrera@cmac-cusco.com.pe;dneira@cmac-cusco.com.pe;clupo@cmac-cusco.com.pe;kalvarez@cmac-cusco.com.pe;muscamaita@cmac-cusco.com.pe;jescobar@cmac-cusco.com.pe'    --29/09/2021 LVCH se agrega a Jimmy Keith  
	SET @psCopiaMail = 'vcabrera@cmac-cusco.com.pe;dneira@cmac-cusco.com.pe;clupo@cmac-cusco.com.pe;kalvarez@cmac-cusco.com.pe;muscamaita@cmac-cusco.com.pe;jescobar@cmac-cusco.com.pe;flvelasquez@cmac-cusco.com.pe;phuamani@cmac-cusco.com.pe'    --17/11/2021 HSPC se agrega a Flor Velasquez y Pablo Huamani           
	
	SET @pTituloMail = 'OBSERVACIONES DE PAGOS ELSE'
	--sET @pprofile_name = 'envioMensajes' -- desarrollo
	SET @pprofile_name = 'BIM Caja Cusco' -- envio de correos internos Canales electrónicos
 
	DECLARE @nIDPago	BIGINT,
			@dfecha		varchar(20),
			@dFechaPago	DATETIME,
			@Canal		VARCHAR(100),
			@cCodAgencia	VARCHAR(100),	
			@cAgeDescripcion	VARCHAR(100),
			@cUser	VARCHAR(100),
			@NombreUsuario		VARCHAR(100),
			@cCodSuministro	VARCHAR(100),
			@cCodigoComprobante	VARCHAR(100),
			@cNombreCliente	VARCHAR(100),
			@cDetalleConsulta	VARCHAR(100),
			@nMontoPago	 MONEY,
			@nnrocob	BIGINT,
			@nmovnro	BIGINT,
			@cCodigoTransaccion VARCHAR(100),	
			@IntentosExtorno	INT,
			@ExtornoExitoso		VARCHAR(100),
			@EstadoFinal	VARCHAR(100),
			@Observacion	varchar(100)
  
		SELECT *
		into #tempELSEObservados
		FROM
		(
			select P.nIDPago,    
			isnull(convert(varchar(20),C.dfecha,103),'') dfecha,             -- Fecha de Sistema    
			P.dFechaPago,            -- Fecha real del Pago    
			CASE WHEN P.cCanal= '03' THEN 'Ventanilla'    
			WHEN P.cCanal = '05' THEN 'POS'    
			WHEN P.cCanal = '04' THEN 'HOMEBANKING'    
			WHEN P.cCanal = '91' THEN 'APP Movil'    
			ELSE '' END 'Canal',          -- Canal    
			P.cCodAgencia, 
			AG.cAgeDescripcion,
			P.cUser,   
			CASE WHEN P.cCanal = '03' THEN A.cPersNombre 
				WHEN P.cCanal = '05' THEN UP.cEstablecimiento
				WHEN P.cCanal = '04' THEN DC.cNombres
				ELSE '' END 'NombreUsuario',    
			RC.cCodSuministro,
			P.cCodigoComprobante,
			RC.cNombreCliente,
			RC.cDetalleConsulta,
			P.nMontoPago, 
			CASE WHEN RP.nnroCob = 0 THEN NULL    
				WHEN RP.nnroCob is null THEN NULL    
				ELSE RP.nnroCob END nnrocob,    
			C.nmovnro,    
    
			CASE WHEN  P.cCanal = '05' THEN DC.cCodigoTransaccion    
				ELSE NULL END 'cCodigoTransaccion',    
			(SELECT COUNT(*) FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago) as IntentosExtorno,    
			CASE WHEN (SELECT TOP 1 nCodigoMotivoExtorno FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno = 1) = 2 THEN 'Servicio Ext. Aut.'    
				WHEN (SELECT TOP 1 nCodigoMotivoExtorno FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno = 1) = 1 THEN 'Extorno Manual'    
				ELSE '' END as ExtornoExitoso,    
              
			CASE WHEN P.nEstadoPago IN (-1,0,3,4)  THEN 'Observado'    
				WHEN P.nEstadoPago=5 AND C.nEstado = 1 THEN 'Observado'    
				WHEN P.nEstadoPago=5 AND M.nMovFlag = 0 THEN 'Observado'    
				WHEN EXISTS(SELECT TOP 1 1 FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and cIdentificadorTransaccion <> P.cIdentificadorTransaccion) THEN 'Observado'    
				WHEN P.nEstadoPago = 1 AND EXISTS (SELECT TOP 1 1 FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno = 1) THEN 'Observado'    
				WHEN P.nEstadoPago = 1 AND EXISTS (SELECT TOP 1 1 FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno in (0,3,4)) THEN 'Observado'    
				WHEN P.nEstadoPago=5 AND     
				EXISTS (SELECT TOP 1 1 FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno = 1 and nCodigoMotivoExtorno = 2) THEN 'Extornado'    
				WHEN P.nEstadoPago=5 AND    
				exists (SELECT TOP 1 1 FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno = 1 and nCodigoMotivoExtorno = 1)     
				AND M.nMovFlag = 2 AND C.nEstado = 2 THEN 'Extornado'        
				WHEN P.nEstadoPago = 1 AND M.nMovFlag = 0 AND C.nEstado = 1 THEN 'Pagado'    
				WHEN P.nEstadoPago = 2  THEN 'No se registra Pago'     
				ELSE 'Observado' END 'EstadoFinal' ,
			CASE WHEN P.nEstadoPago IN (-1) then 'Extorno Fuera de tiempo (15 min)'
				 when P.nEstadoPago IN (0,3,4)  THEN 'Estados inconclusos (0,3,4)'    
				WHEN P.nEstadoPago=5 AND C.nEstado = 1 THEN 'Sincronización - Pagado CMAC - Extorno ELSE'    
				WHEN P.nEstadoPago=5 AND M.nMovFlag = 0 THEN 'Sincronización - Pagado CMAC - Extorno ELSE'    
				WHEN EXISTS(SELECT * FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and cIdentificadorTransaccion <> P.cIdentificadorTransaccion) THEN 'Revisar Identificador de Transaccion'    
				WHEN P.nEstadoPago = 1 AND EXISTS (SELECT TOP 1 1 FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno = 1) THEN 'Sincronización - Pagado CMAC - Extorno ELSE'    
				WHEN P.nEstadoPago = 1 AND EXISTS (SELECT TOP 1 1 FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno in (0,3,4)) THEN 'Extorno ELSE pendiente de confirmación'    
				WHEN P.nEstadoPago=5 AND     
				EXISTS (SELECT TOP 1 1 FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno = 1 and nCodigoMotivoExtorno = 2) THEN 'Extornado'    
				WHEN P.nEstadoPago=5 AND    
				exists (SELECT TOP 1 1 FROM ExtornoPagoELSE WITH(NOLOCK) where nIdPago = P.nIdPago and nEstadoExtorno = 1 and nCodigoMotivoExtorno = 1)     
				AND M.nMovFlag = 2 AND C.nEstado = 2 THEN 'Extornado'        
				WHEN P.nEstadoPago = 1 AND M.nMovFlag = 0 AND C.nEstado = 1 THEN 'Pagado'    
				WHEN P.nEstadoPago = 2  THEN 'No se registra Pago'     
				ELSE 'Observado' END 'Observacion' 
    
			from PagoDeudaELSE P WITH(NOLOCK)       
			INNER JOIN RespuestaConsultaDeudaELSE RC WITH(NOLOCK) ON P.cIdentificadorTransaccion  = RC.cIdTransaccion    
			LEFT JOIN RespuestaPagoDeudaELSE RP   WITH(NOLOCK) ON RP.nIDPago      = P.nIDPago    
			LEFT JOIN CobranzaWS C      WITH(NOLOCK) ON C.nnrocob      = RP.nnrocob    
			LEFT JOIN DetalleCobranzaWS DC    WITH(NOLOCK) ON DC.nnrocob      = C.nnrocob  
			LEFT JOIN dbcmac..RRHH RH  WITH(NOLOCK) ON RH.cUser      = P.cUser 
			LEFT JOIN dbcmac..Persona A  WITH(NOLOCK) ON RH.cPersCod      = A.cPersCod 
			LEFT JOIN dbcmac..UsuariosPOS UP WITH(NOLOCK) ON UP.cUserRRHH     = P.cUser 
			LEFT JOIN dbcmac..Agencias AG WITH(NOLOCK) ON AG.cAgeCod      = P.cCodAgencia
			LEFT JOIN dbcmac..Mov M   WITH(NOLOCK) ON C.nmovnro      = M.nMovNro    
			WHERE    
			P.dFechaPago >= @dFechaIni and    
			P.dFechaPago <  @dFecFinal
		) A
	  WHERE  [EstadoFinal] in ('Observado')
	  ORDER BY 1    

		 SET @HtmlCabGeneral =  
		 '
		 <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">  
	<html xmlns="http://www.w3.org/1999/xhtml" >  
	 <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>  
	 <date-util format="dd/MM/yyyy"></date-util>  
	 <head>  
	  <title>OBSERVACIONES ELSE</title>  
	  <style type="text/css">  
	   body{      padding: 0; font-size: 80%; font-family: "Verdana", "Trebuchet MS", "Helvetica", "Arial",  "sans-serif"; height:100%; color:#000; width:1600px; margin:0 AUTO;}  
	   h3{ font-size: 1em; margin:10px; color:#fff;  }   
	   h4{ color:#D01E1E;  }   
	   h2{ color:#D01E1E;  font-size: 110%;}   
	   h1{ padding: 1px 0; Color:#D01E1E; font-size: 150%;}  
	   table { border-collapse: collapse;}   
	   table th { border: 1px solid #000;  background-color:white; color:#fff; }   
	   table th h1 { background-color:#922b21; margin: 0 Auto; padding: 8px 10px; Color:#fff;}  
	   table th h3 { background-color:#fff; margin: 0 Auto; padding: 8px 10px; Color:#D01E1E; font-size: 100%; }  
     
	   table td { border: 1px solid #000; background-color:white; color: #000;}   
	   table td h3 { background-color:WhiteSmoke; margin: 0 Auto; padding: 8px 10px; Color:#000; font-family: "Verdana", "Trebuchet MS", "Helvetica", "Arial",  "sans-serif"; font-size: 80%;}  
	   p{   font-size: 120%; margin:40px 0 10px 0; color:BLACK; }  
     
	  </style>   
	 </head>   
	 <body>  
    
	  <div>  
	   <p> <b>Estimados, previo un cordial saludo. <br> Mediante el presente se adjunta la siguiente relacion de observaciones ELSE para el dia '+ convert(varchar(2),@Day)  + ' de '+ convert(varchar(10),@MONTH) +' del '+ convert(varchar(4),@Year) +'. <b>   








		 <br><br>  
		<b> Actualizado a la fecha: '+CONVERT(VARCHAR,GETDATE() ,103)+ '<b>  
	   <table>   
		<th colspan = "19" style="text-align:center;">  
		 <h1>OBSERVACIONES ELSE</h1>  
		</th>  
		<tr>  
       
		 <th colspan = "1" style="text-align:center;">  
		  <h3>IdPago</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Fecha</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">  
		  <h3>FechaPago</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Canal</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">  
		  <h3>Cod. Agencia</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Agencia</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">  
		  <h3>Cod. Usuario</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Nombre Usuario</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">  
		  <h3>Nro. Suministro</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Comprobante</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">  
		  <h3>Cliente</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Detalle Consulta</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">  
		  <h3>Monto Pago</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Nro Cobranza</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">  
		  <h3>nMovNro</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Servicio Extorno</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Estado Final</h3>  
		 </th>  
		 <th colspan = "1" style="text-align:center;">   
		  <h3>Observacion</h3>  
		 </th>  
		</tr>  
		 '
		  -- 1) Construccion de detalle                   
				  /*------------------  Detalle  -----------------------*/           
		set @Nro = 0          
		set @HtmlDet = ''          
		while(exists(select top 1 nIDPago from #tempELSEObservados))           
				  begin          
		  Set @Nro = @Nro + 1          
						  select	@nIDPago = ISNULL(nIDPago, ''),
									@dfecha		= ISNULL(dFecha, ''),
									@dFechaPago	= ISNULL(dFechaPago, '19000101'),
									@Canal		= ISNULL(Canal, ''),
									@cCodAgencia	= ISNULL(cCodAgencia, ''),
									@cAgeDescripcion	= ISNULL(cAgeDescripcion, ''),
									@cUser	= ISNULL(cUser, ''),
									@NombreUsuario		= ISNULL(NombreUsuario, ''),
									@cCodSuministro	= ISNULL(cCodSuministro, ''),
									@cCodigoComprobante	= ISNULL(cCodigoComprobante, ''),
									@cNombreCliente	= ISNULL(cNombreCliente, ''),
									@cDetalleConsulta	= ISNULL(cDetalleConsulta, ''),
									@nMontoPago	 = ISNULL(nMontoPago, 0),
									@nnrocob	= ISNULL(nnrocob, 0),
									@nmovnro	= ISNULL(nmovnro, 0),
									--@cCodigoTransaccion = ISNULL(cCodigoTransaccion, ''),
									--@IntentosExtorno	= ISNULL(IntentosExtorno, 0),
									@ExtornoExitoso		= ISNULL(ExtornoExitoso, ''),
									@EstadoFinal	= ISNULL(EstadoFinal, ''),
									@Observacion	= ISNULL(Observacion, '')
						  from  #tempELSEObservados     order by nIDPago asc
                               

							  set @HtmlDet = @HtmlDet + '
								<tr> 
						<td colspan = "1" style="text-align:justify;"> <h3>' + convert(varchar(100),@nIDPago) + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @dfecha + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + convert(varchar(100),@dFechaPago,22) + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @Canal + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @cCodAgencia + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @cAgeDescripcion + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @cUser + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @NombreUsuario + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @cCodSuministro + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @cCodigoComprobante + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @cNombreCliente + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @cDetalleConsulta + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + convert(varchar(100),@nMontoPago) + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + convert(varchar(100),@nnrocob) + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + convert(varchar(100),@nmovnro) + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @ExtornoExitoso + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @EstadoFinal + '</h3></td>
						<td colspan = "1" style="text-align:center;"><h3>' + @Observacion + '</h3></td>

					  </tr>
	'  
						delete from #tempELSEObservados where nIDPago = @nIDPago
				  end          

				  /*------------------  Fin Detalle  -----------------------*/            

	  drop table #tempELSEObservados

		set @HtmlPie = '
	   </table>
			</div>
		
		</body>
	   <footer style="margin: 0 auto;">
			<b>
			<FONT FACE="arial" SIZE=2 COLOR=#00000;><br/>Sincronización de relojes: '+CONVERT(VARCHAR,@nSincronizacion)+' segundos de diferencia con Hiper Center.<br/></FONT>
			<FONT FACE="arial" SIZE=2 COLOR=#00000;><br/>Atentamente,</FONT>
			<div> <h2>	CAJA MUNICIPAL DE AHORROS Y CRÉDITO CUSCO	</h2>	</div>
		</footer>
		</html>'          
                                       
	  set @Html = @HtmlCabGeneral +  @HtmlDet + @HtmlPie          
           
	  EXEC msdb.dbo.sp_send_dbmail                    
	  @recipients = @psParaMail,                  
	  @subject =@pTituloMail,                
	  @body = @Html,                
	  @body_format = 'HTML',                    
	  @profile_name = @pprofile_name,                 
	  @copy_recipients= @psCopiaMail;      
set nocount on
end
