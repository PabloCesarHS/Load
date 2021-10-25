# Load
Repositorio temporal
Dictionary<string, Object> Parametros = new Dictionary<string, object>();
                    Parametros.Add("usuario", _usuario);
                    Parametros.Add("codigoInstitucion", _codigoInstitucion);
                    Parametros.Add("canal", _canal);
                    Parametros.Add("codigoAlumno", _codigoAlumno);
                    Parametros.Add("codigoConcepto", _codigoConcepto);

                    String sParams = JsonConvert.SerializeObject(Parametros);

                    Byte[] byteParams = Encoding.UTF8.GetBytes(sParams);


                    //HttpWebRequest request = (HttpWebRequest)WebRequest.Create(_UrlConDeuda);
                    HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://192.168.116.10/mspagos/pagoservicio/consultarDeuda");
                    //request.ContentType = "application/json";
                    //request.Headers.Add("Authorization", "Bearer " + oAtutenticacion.access_token);
                    request.Method = "POST";
                    //request.ContentLength = byteParams.Length;


                    request.Accept = "application/json;charset=UTF-8";
                    request.Headers["Authorization"] = "Bearer " + oAtutenticacion.access_token;
                    //request.ContentType = "application/json";

                    var data = @"{""usuario"": C430,""codigoInstitucion"": 1061100097462,""canal"":05,""codigoAlumno"": 110002217,""codigoConcepto"": 00}";

                    using (var streamWriter = new StreamWriter(request.GetRequestStream()))
                    {
                        streamWriter.Write(sParams);
                    }

                    var httpResponse = request.GetResponse();
                    using (var streamReader = new StreamReader(httpResponse.GetResponseStream()))
                    {
                        var result = streamReader.ReadToEnd();
                    }

                    //using (Stream dataStream = request.GetRequestStream())
                    //{
                    //    dataStream.Write(byteParams, 0, byteParams.Length);
                    //}

                    using (HttpWebResponse response = (HttpWebResponse)request.GetResponse())
                    {
                        if (response.StatusCode == HttpStatusCode.OK)
                        {
                            using (Stream dataStream = response.GetResponseStream())
                            {
                                StreamReader reader = new StreamReader(dataStream, Encoding.UTF8);
                                oConsultaDeuda = JsonConvert.DeserializeObject<ClaBusInt_ConsultaDeuda>(reader.ReadToEnd());
                                ETMensajeError = "Respuesta Consulta Deuda Obtenida";
                                ETEstadoAutenticacion = 0;
                            }
                        }
                        response.Close();
                    }

                }
                catch (WebException ex)
                {
                    using (Stream dataStream = ex.Response.GetResponseStream())
                    {
                        StreamReader reader = new StreamReader(dataStream, Encoding.UTF8);
                        oConsultaDeuda = JsonConvert.DeserializeObject<ClaBusInt_ConsultaDeuda>(reader.ReadToEnd());
                        ETMensajeError = ex.Message;
                        ETEstadoAutenticacion = 2;
                    }
                }
