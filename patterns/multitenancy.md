# Multi-tenancy API


## Context

Fragment uit de offerte vraag wat betreft het bouwen van een engine:
“Een engine is multi-tenant by design. Dit betekent dat één instantie van de software oplossing meerdere tenants kan voorzien. Elke tenant is een afzonderlijk gescheiden logische omgeving, elk met zijn eigen set gebruikers, autorisatie regels en data.”


## Conceptueel

Multi-tenancy via twee hoedanigheden : providers en consumers.

![multitenancy_conceptueel](/img/multitenancy_conceptueel_20180313.jpg)


## Assumpties

* Elke (nieuwe) (ACPaaS) component wordt verwacht rekening te houden met multi-tenancy, hetzij in de hoedanigheid van provider of als consumer. 
Meer specifiek: elke toekomstige engine is multi-tenant (cfr. offerte), maw elke nieuwe engine is een provider.

* Er is geen centraal tenant beheersysteem.
De provider componenten zelf zijn verantwoordelijk voor tenant beheer en persistentie. In de praktijk 1 of meerdere tenants. Elke tenant heeft  een uniek tenant id. 

* Een eindgebruiker kan in theorie 1 of meerdere hoedanigheden hebben, per toepassing of over de toepassingen heen. Dit impliceert dat een gebruiker gekoppeld aan een applicatie, maar ook dat de rollen en rechten tenant specifiek te configureren zijn (vb. In UME).

## Multi-tenant API template

### Hyperlink example met swagger
[admin api](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/admin.json) 

[business api](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/business.json) 


## FAQ

*  Eigenaarschap: wie of welke component neemt het toevoegen/wijzigen/verwijderen van tenants op zich?

De partij die de component oplevert is verantwoordelijk.
Een provider component implementeert de ‘Multi-tenant API’. De implementatie kan een volledig-  dan wel semi-automatisch proces zijn.
Een consumer component geeft identificeert expliciet de tenant id tijdens API calls. 
Documentatie voor een MTA/MTP is noodzakelijk.

*  Hoe implementeer ik de Multi-tenant API in een ACPaas provider component?

Voorzie een admin service met volgende methoden:

GET /tenants: oplijsten van alle tenants voor component.

POST /tenants: aanmaken van een nieuwe tenant en specifieke dependencies, mogelijks is er een goedkeuringsproces nodig.

DELETE /tenants/{tenantId} : verwijderen van de tenant (by default een soft delete, tenzij parameter ‘isHard’ aan staat).

GET  /tenants/{tenantId} : geeft tenant details terug

PUT  /tenants/{tenantId} : updates de tenant details
    
Voorzie het opvangen  van de tenant id in elke business service: 

Request header: dgp-tenant-id,198b64a5-9a48-4887-9b18-78344946dcc2

Wat als een consumer van een business service geen expliciete dgp-tenant-id meegeeft?

In het Swagger contract is deze HTTP header verplicht mee te geven, de verantwoordelijkheid ligt bij de consumer component.

* Hoe wordt de hoedanigheid van een gebruiker bepaald? 
Vb. gebruiker x logt in de FE/BFF met een specifieke hoedanigheid.

verantwoordelijkheid van de FE/BFF component
tijdens de inlogprocedure definieert de toepassing de hoedanigheid, technisch vertaalt zich dit naar de gdp-tenant-id.

* Wat met gemeenschappelijke data in de databanken?
Vb. fysiek dupliceren of in een gemeenschappelijk schema, leggen we een restrictie op?

Een component kan intern gebruik maken van een data die niet tenant specifiek is
Dit is interne keuken van de component en staat los van de Multi-tenant API.

* Is er een centrale infrastructuur component die de tenant id’s beheert?
Vb. via API Gateway,  AppConfig,  satelliet

Neen, de unieke tenant id’s worden bewaard door de provider component zelf, niet centraal. Dit kan bijvoorbeeld in een gedeelde tabel in de databank.

* Ivm persistentie: hoe maken we een onderscheid tussen de tenant specifiek databronnen?

We werken met een unieke connectie string per tenant:

Eén instantie:
PostgreSQL: jdbc:postgresql://localhost:5432/mydatabase?currentSchema=myschema
MongoDB: cfr. https://docs.mongodb.com/manual/reference/connection-string/
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]

Meerdere instanties:
PostgreSQL: per instantie een specifieke connectie string
MongoDB: per instantie een specifieke connectie string

* Mag de tenant id als extra kolom gepersisteerd worden?

We maken GEEN gebruik van een extra kolom met daarin een (indirecte) verwijzing naar de tenant id.

![multitenancy_no_tenant_id_columns](/img/multitenancy_no_tenant_id_comlumns_20180313.jpg)
