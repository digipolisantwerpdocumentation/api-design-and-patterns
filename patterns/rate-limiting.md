# Rate Limiting

![rate-limiting-kraan](/img/rate-limiting.jpg)

Rate limiting beschermt een service tegen extreme pieken in het aantal binnenkomende requests. Deze gebeuren meestal met slechte bedoelingen zoals bij denial-of-service attacks of brute-force login pogingen maar kunnen ook te wijten zijn aan een slechte implementatie van communicatie flows. Zo is het niet ondenkbaar dat in bepaalde toepassingen piekmomenten in gebruik zijn zoals bvb bij ticketreservatie systemen. 


Van zulke toepassingen wordt verwacht dat ze intelligent om gaan met het versturen van requests naar een API zodat deze niet overbelast wordt. Dit houdt in dat de client software zodanig moet geïmplementeerd worden dat deze de pieken kan opvangen. Dit kan bvb door een queueing mechanisme te voorzien voor het bufferen van de requests naar de API. 


Gebeurt dit niet dan zal bij dreiging van overbelasting de rate limiting van de service geactiveerd worden waardoor elke request die over de limiet gaat, geweigerd wordt. De client zal onmiddellijk een HTTP status code **429 (Too Many Requests)** terug krijgen. Client toepassingen zijn er best op voorzien om deze status code te herkennen en de request later opnieuw te proberen, gecombineerd met een eigen throttling die er voor zorgt dat er minder requests in een korte tijdspanne worden gestuurd. 


Zulke rate limiting kan in de service zelf voorzien worden maar wordt in de meeste gevallen opgevangen door een infrastructurele component zoals een _API Gateway_.
