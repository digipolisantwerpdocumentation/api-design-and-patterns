# Multi-tenancy API v1


## Context

Fragment uit de offerte-vraag document wat betreft het bouwen van een engine:
“Een engine is multi-tenant by design. 
Dit betekent dat één instantie van de software oplossing meerdere tenants kan voorzien. Elke tenant is een afzonderlijk gescheiden logische omgeving, elk met zijn eigen set gebruikers, autorisatie regels en data.”


## Conceptueel

Multi-tenancy kan op verscheidene manieren worden geïmplenteerd.
In wat volgt maken we een onderscheid tussen tussen de (dubbel)rol die betrokken componenten kunnen spelen : die van provider en/of consumer.

De voorgestelde oplossing is decentraal, vermijdt redundante componentenis API gebaseerd:
![multitenancy_conceptueel](/img/multitenancy_conceptueel_20180313.jpg)

## Assumpties

* Elke (nieuwe) (ACPaaS) component wordt verwacht rekening te houden met multi-tenancy, hetzij als provider, hetzij als consumer. 
Meer specifiek: elke toekomstige engine is multi-tenant (cfr. offerte vraag), maw elke nieuwe engine is een provider.

* Er is geen centraal tenant beheersysteem.
De provider componenten zijn zelf verantwoordelijk voor tenant beheer en persistentie. In de praktijk heeft men altijd 1 of meerdere tenants. Elke tenant heeft  een uniek tenant id. 

* Een eindgebruiker kan in theorie 1 of meerdere hoedanigheden hebben, per toepassing of over de toepassingen heen. Dit impliceert dat een gebruiker is gekoppeld aan een applicatie, maar ook dat de rollen en rechten tenant specifiek te configureren zijn.

## Terminologie

| Term | Definitie |
| :--- | :--- |
| Multi-tenant | Een component is multi-tenant indien één instantie van de software oplossing meerdere tenants kan voorzien. Elke tenant is een afzonderlijk gescheiden logische omgeving, elk met zijn eigen set gebruikers, autorisatie regels en data. Een service is multi-tenant indien deze de multi-tenancy API implementeert. |
| Multi-instance| Een uitbreiding van een multi-tenant component (zie boven) waarbij men gebruik maakt van afzonderlijk gescheiden processen. |
| Tenant-aware | Een client component is tenant-aware indien deze expliciet een ‘tenant id’ gebruikt bij het consumeren van een andere component. |
| Tenant catalog | Een component draagt zelf de verantwoordelijkheid voor persistentie van de tenant id’s, dit geldt voor de provider of consumer kant, of beide. De persistentie gebeurt in de zogenaamde 'tenant catalog'. |
| Tenant provisioning | De Multi-tenant API voorziet beheer methoden om nieuwe tenant id’s en mapping te provisioneren. | 
| Tenant id | Dit is de unieke identificatie van een tenant binnen een bepaalde component.

## Multi-tenant API template

#### Template met 2 swagger definities: Business API & Admin API

* De [Business API](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/business.json) voorziet het opvangen van de tenant id in elke business methode*:

| Method | Details |
| :--- | :--- |
| GET /tenants/me | Get all dependent created tenants and id |
| POST /business-parties* | Creates a business party |
| GET /business-parties* | Get all the parties |

	
* De [Admin API](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/admin.json) 
bevat volgende methoden ifv tenant beheer:

| Method | Details |
| :--- | :--- |
| POST /tenants | Creates a new tenant |
| GET /tenants | Get all tenants |
| GET /tenants/{tenantid} | Get tenant details |
| DELETE /tenants/{tenantid} | Delete a tenant |
| PUT /tenants/{tenantid} | Update the tenant details |

#### Hoedanigheid

Componenten kunnen simultaan volgende hoedanigheden hebben: 
* hoedanigheid als provider: deze is multi-tenant by design; 
* hoedanigheid als consumer: client van een multi-tenant component.


| ACPaas component | Provider rol? Aanleveren multi-tenant API? | Consumer rol? Tenant-aware? |
| :--- | :--- | :--- |
| Engines | Ja, engine is multi-tenant by design. | Optioneel, wel indien afnemer van een ander multi-tenant component. |
| Engine cluster | Optioneel, wenselijk. | Verplicht. |
| Business services  | Optioneel, wenselijk. | Verplicht. |
| FE & BFF | Nvt. | Verplicht. |


#### Tenant id

* De tenant id is een unieke identificatie van een tenant binnen een component.

* Bij aanroep van een bedrijfsmethode geeft de consumer component de tenant id (dgp-tenant-id) mee in de HTTP header.  Deze tenant id moet gekend zijn in de tenant provider die wordt aangeroepen.

* Decentraal : de unieke tenant id’s worden aangemaakt en bewaard door de provider component zelf, dus niet via een centrale entiteit. Dit kan bijvoorbeeld in een gedeelde tabel in de ‘tenant catalog’ databank van de component.

* Vorm: sleutel, waarde paar.

* Formaat: UUID met 36 tekens,  waarvan 32 hexadecimale karakters en vier streepjes: 8-4-4-4-12 (cfr. https://tools.ietf.org/html/rfc4122) 

```
Sleutel:  dgp-tenant-id
Waarde: 198b64a5-9a48-4887-9b18-78344946dcc2
```


## FAQ

#####  Eigenaarschap: wie of welke component neemt het toevoegen/wijzigen/verwijderen van tenants op zich?

* De partij die de component oplevert is verantwoordelijk.
* Een provider component implementeert de ‘Multi-tenant API’. Het tenant beheer kan als een volledig- dan wel semi-automatisch proces geimplementeerd worden.
* Een consumer component identificeert expliciet de tenant id tijdens API calls. 
* Documentatie voor een MTA/MTP is noodzakelijk.

#####  Hoe implementeer ik de Multi-tenant API in een ACPaas provider component?

* De [Business API](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/business.json) voorziet het opvangen van de tenant id in elke business methode*:

| Method | Details |
| :--- | :--- |
| GET /tenants/me | Get all dependent created tenants and id |
| POST /business-parties* | Creates a business party |
| GET /business-parties* | Get all the parties |

	
* De [Admin API](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/admin.json) 
bevat volgende methoden ifv tenant beheer:

| Method | Details |
| :--- | :--- |
| POST /tenants | Creates a new tenant |
| GET /tenants | Get all tenants |
| GET /tenants/{tenantid} | Get tenant details |
| DELETE /tenants/{tenantid} | Delete a tenant |
| PUT /tenants/{tenantid} | Update the tenant details |
    
##### Wat als een consumer van een business service geen expliciete dgp-tenant-id meegeeft?

In het Swagger contract is deze HTTP header verplicht mee te geven, de verantwoordelijkheid ligt bij de consumer component, deze dient de correcte tenant-id mee te geven.

##### Hoe wordt de tenant van een gebruiker bepaald? Eenzelfde gebruiker kan immers toegang hebben tot meerdere tenants voor eenzelfde applicatie.

* Indien de applicatie meerdere tenants voorziet: het is de verantwoordelijkheid van de FE/BFF component om de keuzelijst aan te bieden tijdens de inlogprocedure.
* De component kan op basis van de rollen/permissies de verschillende tenants van een geauthenticeerde gebruiker bepalen.
* De gebruiker maakt een keuze en start feitelijk een sessie voor één specifieke tenant.
* De toepassing linkt de gebruiker dus aan specifieke tenant om vb. de correcte datastore aan te spreken. 

##### Wat met gemeenschappelijke data in de datastore?
Vb. fysiek dupliceren of in een gemeenschappelijk schema, leggen we een restrictie op?

Een component kan intern gebruik maken van een data die niet tenant specifiek is
Dit is interne keuken van de component en staat los van de Multi-tenant API.

##### Is er een centrale infrastructuur component die de tenant id’s beheert?

Neen, de unieke tenant id’s worden bewaard door de provider component zelf, niet centraal. 
vb. via persitentie in een adminstratieve tabel in de tenant catalog van de component.

##### Wat is de tenant catalog?

Iedere tenant heeft zijn eigen databank(schema). De tenant catalog bevat data nodig voor het tenant beheer van de component. De tenant catalog bevindt zich op zijn beurt in een eigen datastore(schema).

Ter illustratie:

![multitenancy_tenant_catalog](/img/multitenancy_tenant_catalog_20180313.jpg)


##### Ivm persistentie: hoe maken we technisch het onderscheid tussen tenant specifieke databronnen?

We werken met een unieke connectie string per tenant:

* Eén instantie:
  - PostgreSQL: jdbc:postgresql://localhost:5432/mydatabase?currentSchema=myschema
  - MongoDB: cfr. https://docs.mongodb.com/manual/reference/connection-string/mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]

* Meerdere instanties:
  - PostgreSQL: per instantie een specifieke connectie string
  - MongoDB: per instantie een specifieke connectie string

##### Mag de tenant id als extra kolom gepersisteerd worden?

We maken GEEN gebruik van een extra kolom met daarin een (indirecte) verwijzing naar de tenant id.

![multitenancy_no_tenant_id_columns](/img/multitenancy_no_tenant_id_comlumns_20180313.jpg)
