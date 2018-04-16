# Multi-tenancy API


#Leeg document in te vullen vanuit google docs

### Hyperlink example met swagger
[admin api](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/admin.json) 

[business api](https://editor.swagger.io/?url=https://raw.githubusercontent.com/digipolisantwerpdocumentation/api-design-and-patterns/multitenancy/swaggers/multitenancy/business.json) 

# Context

Fragment uit de offerte vraag wat betreft het bouwen van een engine:
“Een engine is multi-tenant by design. Dit betekent dat één instantie van de software oplossing meerdere tenants kan voorzien. Elke tenant is een afzonderlijk gescheiden logische omgeving, elk met zijn eigen set gebruikers, autorisatie regels en data.”


# Conceptueel

![multitenancy_conceptueel](/img/multitenancy_conceptueel_20180313.jpg)


# Assumpties

Elke (nieuwe) (ACPaaS) component wordt verwacht rekening te houden met multi-tenancy, hetzij in de hoedanigheid van provider of als consumer. 
Meer specifiek: elke toekomstige engine is multi-tenant (cfr. offerte), maw elke nieuwe engine is een provider.

Er is geen centraal tenant beheersysteem.
De provider componenten zelf zijn verantwoordelijk voor tenant beheer en persistentie. In de praktijk 1 of meerdere tenants. Elke tenant heeft  een uniek tenant id. 

Een eindgebruiker kan in theorie 1 of meerdere hoedanigheden hebben, per toepassing of over de toepassingen heen. Dit impliceert dat een gebruiker gekoppeld aan een applicatie, maar ook dat de rollen en rechten tenant specifiek te configureren zijn (vb. In UME).

