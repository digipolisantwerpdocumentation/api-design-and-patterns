# API design and patterns

Deze documentatie geeft een overzicht van de API design patterns en concepten die wij gebruiken. Zij kunnen een developer of architect helpen in het designen en implementeren van enterprise applicaties.

## Inhoudstafel

[Circuit Breaking](/patterns/circuit-breaking.md)
> Circuit breaking vermijdt onnodig gebruik van resources door requests die niet gaan lukken onmiddellijk te doen falen ipv ze uit te voeren.
  
[Rate Limiting](/patterns/rate-limiting.md)  
> Rate limiting beschermt een service tegen overload door het aantal toegelaten requests binnen een tijdspanne te limiteren.

[BFF Session Storage/Caching](/patterns/session-storage.md)  
> BFF Session storage is nodig om de state van een aangemelde gebruiker bij te houden.  
> BFF Caching kan gebruikt worden om data tijdelijk te cachen om de performantie te verbeteren.
