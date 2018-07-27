# Correlation in Services (API's)

![correlation](/img/correlation.jpg)


<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

## Inhoudstabel
- [Doel](#doel)
- [Gebruik](#gebruik)
  - [Schema](#schema)
  - [HTTP Header](#http-header)
  - [Ontbrekende header bij binnenkomend request](#ontbrekende-header-bij-binnenkomend-request)
  - [UserId en IP adres](#userid-en-ip-adres)
  - [Policy op API Manager](#policy-op-api-manager)
- [SDK's](#sdks)
  - [ASP.NET Core](#aspnet-core)
  - [.NET 4.x](#net-4x)
  - [AngularJs](#angularjs)
  - [Node.Js](#nodejs)
  - [ESB](#esb)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## Doel

Dit document beschrijft het concept **_correlation_** bij het aanroepen van services en API's.

In de meeste moderne, gedistribueerde systemen zijn de functionaliteiten en gegevens verspreid over meerdere services of API's. Een handeling die een gebruiker uitvoert, heeft meestal tot gevolg dat er een keten van services aangesproken wordt, al dan niet over API gateways en ESB's. Om zo 1 handeling (of **_flow_**)  te kunnen traceren over alle systemen heen, wordt van in het begin van de flow een correlation toegevoegd. Deze correlation is uniek per flow die vertrekt bij de gebruiker, en wordt telkens overgenomen tussen elke node die tijdens de flow wordt aangesproken.

De correlation wordt mee gelogd zodat hierop kan gezocht of gefilterd worden tussen de logs van de verschillende nodes. Het gebruik van de logging engine om al die logs samen te brengen (of m.a.w. door al de nodes te doen loggen naar de logging engine) kan dit proces vergemakkelijken doordat op 1 plaats kan gezocht worden naar alle log entries voor een bepaalde correlation.


## Gebruik

De correlation wordt doorgegeven via een HTTP header met naam '**_Dgp-Correlation_**'.


### Schema
De correlation is een json object met 7 string velden volgens volgend schema :


```json
{
  "id": "d80db7ea-fe4c-4df5-afe1-1b675e19921f",
  "sourceId": "e27ce2ff-4cf1-40e8-8d70-fe6b105e6490",
  "sourceName": "appName",
  "instanceId": "8d0e2382-0540-4229-bc6d-8cdc9c2294ab",
  "instanceName": "appName-instanceName",
  "userId": "userid",
  "ipAddress": "194.25.76.122"
}
```

Veld | Betekenis | Aanwezigheid
---- | --------- | ------------
id   | Uniek id per flow. | Verplicht
sourceId | Unieke identificatie van de component of toepassing die de flow opstart (maw waar de correlation wordt gecreëerd).<br/> Info komt uit het applicatie configuratie systeem. | Verplicht
sourceName | Display-friendly naam van de component of toepassing die de flow opstart (maw waar de correlation wordt gecreëerd).<br/> Info komt uit het applicatie configuratie systeem. | Verplicht
instanceId | Unieke identificatie van de instance van de component of toepassing die de flow opstart. Deze info is nuttig in load-balanced situaties. | Verplicht
instanceName | Display-friendly naam van de instance van de component of toepassing die de flow opstart. | Verplicht
userId | User Id van de gebruiker die de flow opstart. | Optioneel
ipAddress | IP adres van de gebruiker die de flow opstart. | Optioneel



### HTTP Header

Het correlation object wordt aangemaakt en toegevoegd aan het request door de component die een flow opstart. Meestal zal dit de de user interface zijn.

Het object wordt als **_base64 encoded string_** doorgegeven in een HTTP header met key "**_Dgp-Correlation_**".

vb :

hogervermelde JSON wordt omgezet naar base64 :


```text
eyAiaWQiOiAiYjg2MjVjIiwgInNvdXJjZSI6ICJhcHBOYW1lIiwgInVzZXJJZCI6ICJ1c2VyaWQiLCAiaXBBZGRyZXNzIjogIjE5NC4yNS43Ni4xMjIiIH0=
```


en doorgegeven als volgt :


![correlation-id](/img/correlation-1.png)


Dit geldt voor alle requests (zowel voor GET's, POST's, PUT's, … als alle andere).


### Ontbrekende header bij binnenkomend request 

Als bij een binnenkomend request de header ontbreekt, zal de ontvanger van het request zelf een correlation creëren volgens bovenstaande regels en vanaf dan deze correlation doorgeven bij volgende requests naar andere nodes binnen dezelfde flow.

Als er wel een correlation header bestaat op een binnenkomend request :

1.  als de verplichte velden zijn ingevuld dan moet dit as-is doorgegeven worden bij de volgende request in dezelfde flow.
2.  als 1 van de verplichte velden ontbreekt, dan worden **ALLE** verplichte velden (id en source) geherinitialiseerd met info van het op dat moment in uitvoering zijnde systeem (ingevulde optionele velden blijven ongewijzigd).


### UserId en IP adres

Alhoewel userId en ipAddres voor de meeste flows niet nodig zijn, is het aangeraden deze altijd in te vullen. 

Sommige services zullen deze velden wel als vereiste hebben. Bij aanroepen van services die wettelijk verplicht zijn opgevraagde privacy gegevens te loggen, moeten deze wel ingevuld zijn. Deze services zullen requests waar deze velden niet zijn ingevuld, weigeren.

Ook voor logging doeleinden is het altijd aangeraden deze velden in te vullen. Dit geeft extra opzoekmogelijkheden in de logs wanneer een gebruiker een probleem inseint.


### Policy op API Manager

Het toevoegen van de correlation header aan een request zou ook via een (nog te voorziene) custom policy op de API Manager kunnen worden gedaan. 


## SDK's

Voor onze DAAS stacks, zijn SDK's of libraries beschikbaar die deze functionaliteit aanbieden.


### ASP.NET Core

Voor ASP.NET Core is er een Correlation toolbox die bovenstaande logica uitvoert: de correlation header wordt ingelezen en in de code is er een ICorrelationContext beschikbaar die kan geïnjecteerd worden en de correlation velden bevat. Indien er geen header binnen komt, wordt een nieuwe correlation aangemaakt.

[https://github.com/digipolisantwerp/correlation_aspnetcore](https://github.com/digipolisantwerp/correlation_aspnetcore) 

In samenwerking met de Logging toolboxen wordt deze correlation mee gelogd naar de logging engine.

[https://github.com/digipolisantwerp/serilog-correlation_aspnetcore](https://github.com/digipolisantwerp/serilog-correlation_aspnetcore) 

Bij gebruik van de ServiceAgents toolbox, wordt de binnengekomen correlation toegevoegd aan elke uitgaande request. 

[https://github.com/digipolisantwerp/serviceagents_aspnetcore](https://github.com/digipolisantwerp/serviceagents_aspnetcore) 


### .NET 4.x

nog uit te werken indien nodig...


### AngularJs

nog uit te werken...


### Node.Js

nog uit te werken...


### ESB

Voor de ESB is een sequence voorzien die de correlation zal beheren.

Sequence : UseCorrelation.seq

Assumpties : 

*   Als de Dgp-Correlation header niet aanwezig is, dan wordt deze gecreëerd door de ESB (met een unieke ID en als Source "ESB")
*   Als de Dgp-Correlation header aanwezig is :
    *    dan wordt er gecontroleerd op de verplichte velden "id" ,"sourceId", "sourceName", "instanceId"en "instanceName"; <br/>
indien 1 hiervan ontbreekt, dan worden alle verplichte velden opnieuw aangemaakt door de ESB.
    *   dan worden optionele velden doorgegeven indien deze niet leeg zijn.

Er wordt ook een OM object ter beschikking gesteld met de Correlation info zodat je zelf geen decodering moet doen indien je Correlation info wil gebruiken : property=Dgp-Correlation-Header (type=OM)


Vb. van het OM object :

```xml
<correlation>
   	<id>75cda136-571d-e0a1-7b83-4992f7848176</id>
   	<sourceId>123-PEK</sourceId>
   	<sourceName>PekeApp</sourceName>
   	<instanceId>123-PEK-ABC</instanceId>
   	<instanceName>AlfaBeta</instanceName>
   	<userId>itsme</userId>
   	<ipAddress>123.123.123.123</ipAddress>
</correlation>
```


