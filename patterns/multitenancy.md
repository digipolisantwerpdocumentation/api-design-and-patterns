# Multi-tenancy API v1
## Context & Richtlijnen
Een toelichting over Multi-tenancy, de belangrijkste begrippen en ACPaaS Richtlijnen zijn terug te vinden op het [ACPaaS portaal](https://acpaas.digipolis.be/nl/docs/multitenancy-requirements).  

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

## Ivm persistentie: hoe maken we technisch het onderscheid tussen tenant specifieke databronnen?

We werken met een unieke connectie string per tenant:

* Eén instantie:
  - PostgreSQL: `jdbc:postgresql://localhost:5432/mydatabase?currentSchema=myschema`
  - MongoDB: cfr. https://docs.mongodb.com/manual/reference/connection-string/mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]

* Meerdere instanties:
  - PostgreSQL: per instantie een specifieke connectie string
  - MongoDB: per instantie een specifieke connectie string

## Mag de tenant id als extra kolom gepersisteerd worden?

We maken GEEN gebruik van een extra kolom met daarin een (indirecte) verwijzing naar de tenant id.

![multitenancy_no_tenant_id_columns](/img/multitenancy_no_tenant_id_comlumns_20180313.jpg)

####  Eigenaarschap: wie of welke component neemt het toevoegen/wijzigen/verwijderen van tenants op zich?

* De partij die de component oplevert is verantwoordelijk.
* Een provider component implementeert de ‘Multi-tenant API’. Het tenant beheer kan als een volledig- dan wel semi-automatisch proces geimplementeerd worden.
* Een consumer component identificeert expliciet de tenant id tijdens API calls. 
* Documentatie voor een MTA/MTP is noodzakelijk.
