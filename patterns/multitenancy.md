##  API & multi-tenancy

#### 2-ledig: impact op de  Business API & een extra Tenant Admin API

* De [Business API](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/business.json) voorziet het opvangen van de tenant id in elke business methode*:

| Method | Details |
| :--- | :--- |
| `GET /tenants/me` | Get all dependent created tenants and id |
| `POST /business-parties*` | Creates a business party |
| `GET /business-parties*` | Get all the parties |

	
* De [Admin API](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/admin.json) 
bevat volgende methoden ifv tenant beheer:

| Method | Details |
| :--- | :--- |
| `POST /tenants` | Creates a new tenant |
| `GET /tenants` | Get all tenants |
| `GET /tenants/{tenantid}` | Get tenant details |
| `DELETE /tenants/{tenantid}` | Delete a tenant |
| `PUT /tenants/{tenantid}` | Update the tenant details |

## FAQ

#### Wat is een tenant id?

* De tenant id is een unieke identificatie van een tenant binnen een component.

* Bij aanroep van een bedrijfsmethode geeft de consumer component de HTTP header `dgp-tenant-id` mee.  Deze tenant id moet gekend zijn in de tenant provider die wordt aangeroepen.

* Decentraal : de unieke tenant id’s worden aangemaakt en bewaard door de provider component zelf, dus niet via een centrale entiteit. Dit kan bijvoorbeeld in een gedeelde tabel in de ‘tenant catalog’ databank van de component.

* Vorm: sleutel, waarde paar.

* Formaat: UUID met 36 tekens,  waarvan 32 hexadecimale karakters en vier streepjes: 8-4-4-4-12 (cfr. https://tools.ietf.org/html/rfc4122) 

* Voorbeeld:
   ```
   Sleutel:  dgp-tenant-id
   Waarde: 198b64a5-9a48-4887-9b18-78344946dcc2
   ```
   
#### Wat is de tenant catalog?

De tenant catalog bevat data nodig voor het tenant beheer van de component. De tenant catalog bevindt zich in een eigen datastore(schema).<br/>
Iedere tenant heeft zijn eigen databank(schema).

Ter illustratie:

![multitenancy_tenant_catalog](/img/multitenancy_tenant_catalog_20180313.jpg)

####  Hoe implementeer ik de Multi-tenant API in een ACPaas provider component?

Zie de paragraaf hierboven: API & multi-tenancy. 
    
#### Wat als een consumer van een business service geen expliciete dgp-tenant-id meegeeft?

In het Swagger contract is deze HTTP header verplicht mee te geven, de verantwoordelijkheid ligt bij de consumer component, deze dient de correcte tenant-id mee te geven.

#### Hoe wordt de tenant van een gebruiker bepaald? Eenzelfde gebruiker kan immers toegang hebben tot meerdere tenants voor eenzelfde applicatie.

* Indien de applicatie meerdere tenants voorziet: het is de verantwoordelijkheid van de FE/BFF component om de keuzelijst aan te bieden tijdens de inlogprocedure.
* De component kan vb. op basis van de rollen/permissies de verschillende tenants van een geauthenticeerde gebruiker bepalen.
* De gebruiker maakt een keuze en start feitelijk een sessie voor één specifieke tenant.
* De toepassing linkt de gebruiker dus aan specifieke tenant om vb. de correcte datastore aan te spreken. 

#### Wat met gemeenschappelijke data in de datastore?
Vb. fysiek dupliceren of in een gemeenschappelijk schema, leggen we een restrictie op?

Een component kan intern gebruik maken van een data die niet tenant specifiek is
Dit is interne keuken van de component en staat los van de Multi-tenant API.

#### Is er een centrale infrastructuur component die de tenant id’s beheert?

Neen, de unieke tenant id’s worden bewaard door de provider component zelf, niet centraal. 
vb. via persitentie in een adminstratieve tabel in de tenant catalog van de component.

#### Ivm persistentie: hoe maken we technisch het onderscheid tussen tenant specifieke databronnen?

We werken met een unieke connectie string per tenant:

* Eén instantie:
  - PostgreSQL: `jdbc:postgresql://localhost:5432/mydatabase?currentSchema=myschema`
  - MongoDB: cfr. https://docs.mongodb.com/manual/reference/connection-string/mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]

* Meerdere instanties:
  - PostgreSQL: per instantie een specifieke connectie string
  - MongoDB: per instantie een specifieke connectie string

#### Mag de tenant id als extra kolom gepersisteerd worden?

We maken GEEN gebruik van een extra kolom met daarin een (indirecte) verwijzing naar de tenant id.

![multitenancy_no_tenant_id_columns](/img/multitenancy_no_tenant_id_comlumns_20180313.jpg)

####  Eigenaarschap: wie of welke component neemt het toevoegen/wijzigen/verwijderen van tenants op zich?

* De partij die de component oplevert is verantwoordelijk.
* Een provider component implementeert de ‘Multi-tenant API’. Het tenant beheer kan als een volledig- dan wel semi-automatisch proces geimplementeerd worden.
* Een consumer component identificeert expliciet de tenant id tijdens API calls. 
* Documentatie voor een MTA/MTP is noodzakelijk.
