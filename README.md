# Lightweight Scala TLS HTTP 1.1 Web Server based on ZIO async fibers and Java NIO sockets.

It uses ZIO-JSON to support JSON encoding/decodig. Please, refer to the README at the main branch.

fromJSON() extracts object A from Chunk[Byte] Request's body.

    sealed case class Request(headers: Headers, body: Chunk[Byte], ch: Channel) {
        ... 
        def fromJSON[A : JsonDecoder] : A
        ...
                     
    }

asJsonBody() mutates a Response by adding Chunk[Byte] body to Response, which is built from object of type B, with assumption that JsonEncoder type class instance exists for B.

    sealed case class Response(code: StatusCode, headers: Headers, body: Option[Chunk[Byte]] ) {
        ...   
        def asJsonBody[B : JsonEncoder]( body0 : B ) : Response
        ...
    } 
  
  To make it work: for JSON case class you will need to create companion object.
  Example:
  
    object DataBlock {
        implicit val decoder: JsonDecoder[DataBlock] = DeriveJsonDecoder.gen[DataBlock]
        implicit val encoder: JsonEncoder[DataBlock] = DeriveJsonEncoder.gen[DataBlock
    }

    case class DataBlock(val name: String, val address: String, val colors : Chunk[String] )
    
   Actual Route, text and JSON.
    
     val app_route_JSON = HttpRoutes.ofWithFilter(proc1) { 

       case GET -> Root / "test2" =>
         ZIO(Response.Ok.asTextBody( "Health Check" ) )

        
       case GET -> Root / "test" =>
         ZIO(Response.Ok.asJsonBody( DataBlock("Thomas", "1001 Dublin Blvd", Chunk( "red", "blue", "green" ) ) ) )
                                                
       case req @ POST -> Root / "test" =>
         ZIO.effect { //need to wrap up everything in the effect to have proper error handling
           val db : DataBlock = req.fromJSON[DataBlock]
           val name = db.name
           Response.Ok.asTextBody( s"JSON for $name accepted" )     
         }                                  
      }   
    
    

