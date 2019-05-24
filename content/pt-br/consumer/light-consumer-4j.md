---
title: "Light Consumer 4j"
date: 2018-12-05T13:04:18-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
---

# Light Consumer 4j

Light Consumer 4h é um módulo que ajuda os consumidores (consumers) a integrar-se facilmente com as apis light-4j.
Ele é construído em cima do Http2Client e java8, possui muitos recursos extras, como pool de conexão, etc.
Ele suporta tanto a URL direta quanto a descoberta de serviço (Cônsul) para chamar as APIs.

### Uso
Adicione a dependência em seu projeto.

```xml
<dependency>
    <groupId>com.networknt</groupId>
    <artifactId>light-consumer-4j</artifactId>
    <version>1.5.26</version>
</dependency>
```

Métodos disponíveis no ClientBuilder

 Future <ClientResponse> send()           // Envia requisição.
 
 String getServiceUrl()                   // obtém o serviceUrl resolvido
 
 disableHttp2()                           // Desabilita Http2, por padrão isso é habilitado.
 
 setClientRequest(ClientRequest reg)      // set requisição do cliente ao httpClientRequest.
 
 setRequestBody(String requestBody)       // set requestBody ao httpClientRequest.
 
 setHeaderValue(HttpString headerName,..) // set headerName e headerValue ao httpClientRequest.
 
 setApiHost(String apiUrl)	              // set apiUrl em httpClientRequest para chamar api com url direta.
 
 setConnectionCacheTTLms(long tTLms)      // set timeout de connexão de cache.
 
 setRequestTimeout(TimeoutDef timeout)    // set timeout da requisição.
 
 setConnectionRequestTimeout(TimeoutDef t)// set timeout da conexão de requisição.
 
 setAuthToken(String authToken)           // Set o token de autenticação na requisição.
 
 getAuthToken()                           // get token de autenticação
 
 addCCToken()                             // adicione o token de acesso no header da requisição e também checa a expiração do token de acesso.

 setServiceDef(ServiceDef serviceDef)     // set protocolo, service id, ambiente e requestKey para chamada da API via consul.

 setMaxReqCount(int maxReqCount)          // habilita cache de conexão pelo número da requisição.
 
 
 #### Aqui um exemplo de código para chamar a API.
 
Chame a API com URL direta:

Exemplo 1:
```
Future<ClientResponse> clientRequest = new HttpClientBuilder()
                      
    //set direct url
   .setApiHost("https://localhost:8453")
   .setClientRequest(new ClientRequest().setPath("/v1/customers/1").setMethod(Methods.GET))
   .setLatch(new CountDownLatch(1))
   .disableHttp2()
   .addCCToken()
   .setConnectionRequestTimeout(new TimeoutDef(100, TimeUnit.SECONDS))
   .setRequestTimeout(new TimeoutDef(100, TimeUnit.SECONDS))
   .setRequestBody("")
   .setConnectionCacheTTLms(10000)
   .setMaxReqCount(5)
   .send();
ClientResponse clientResponse = clientRequest.get();

```
Exemplo 2:

```
            public static final String CUSTOMERS_URL = "https://localhost:8453";
            private static final String CUSTOMERS_PATH = "/v1/customers/{id}";
            
            HttpClientBuilder clientBuilder = new HttpClientBuilder();
            
            ClientRequest clientRequest = new ClientRequest().setMethod(Methods.GET)
            .setPath(CUSTOMERS_PATH.replace("{id}", id));
            
            clientBuilder.setClientRequest(clientRequest);
            clientBuilder.setLatch(new CountDownLatch(1));
            if (!Server.config.isEnableHttp2()) {
              clientBuilder.disableHttp2();
            }
            clientBuilder.setApiHost(CUSTOMERS_URL);
            clientBuilder.setConnectionCacheTTLms(20000);
            clientBuilder.setMaxReqCount(5)
            clientBuilder.setConnectionRequestTimeout(new TimeoutDef(10, TimeUnit.SECONDS));
            Future<ClientResponse> clientBuilderRequest = clientBuilder.send();
            ClientResponse clientResponse = clientBuilderRequest.get();
```

Chave a API via descoberta de serviço (Consul):

Exemplo 1:

```
Future<ClientResponse> clientRequest = new HttpClientBuilder()
                     
    //set protocol,service id , environment  and requestKey 
   .setServiceDef(new ServiceDef("https", "training.customers-1.00.00","training", null))
   .setClientRequest(new ClientRequest().setPath("/v1/customers/1").setMethod(Methods.GET))
   .setLatch(new CountDownLatch(1))
   .disableHttp2()
   .addCCToken()
   .setConnectionRequestTimeout(new TimeoutDef(100, TimeUnit.SECONDS))
   .setRequestTimeout(new TimeoutDef(100, TimeUnit.SECONDS))
   .setRequestBody("")
   .setConnectionCacheTTLms(10000)
   .setMaxReqCount(5)
   .send();
ClientResponse clientResponse = clientRequest.get();

```
Exemplo 2:

```  
    public static final String CUSTOMERS_SERVICE = "training.customers-1.00.00";
    private static final String CUSTOMERS_PATH = "/v1/customers/{id}";
    
    HttpClientBuilder clientBuilder = new HttpClientBuilder();
    
    ClientRequest clientRequest = new ClientRequest().setMethod(Methods.GET)
    .setPath(CUSTOMERS_PATH.replace("{id}", id));
    
    clientBuilder.setClientRequest(clientRequest);
    clientBuilder.setLatch(new CountDownLatch(1));
    if (!Server.config.isEnableHttp2()) {
        clientBuilder.disableHttp2();
    }
    clientBuilder.setServiceDef(new ServiceDef("https", CUSTOMERS_SERVICE, "training", null));
    clientBuilder.setConnectionCacheTTLms(20000);
    clientBuilder.setMaxReqCount(5);
    clientBuilder.setConnectionRequestTimeout(new TimeoutDef(10, TimeUnit.SECONDS));
    Future<ClientResponse> clientBuilderRequest = clientBuilder.send();
    ClientResponse clientResponse = clientBuilderRequest.get();
```
    
    

