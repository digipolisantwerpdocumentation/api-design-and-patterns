# Circuit Breaker Implementatie

Een circuit breaker wordt meestal gebouwd als een state machine met 3 statussen :  
- gesloten
- open
- half-open  

Bij normale werking staat de circuit breaker in de **gesloten** stand. Alle requests naar de service worden gewoon doorgelaten.  
  
![circuit-breaker](/img/circuit-breaker-1.png)  
  
De circuit breaker registreert de communicatiefouten naar de service. Wanneer een vooraf bepaalde threshold van aantal mislukte communicaties binnen een tijdspanne bereikt is, zet de circuit breaker zichzelf in **open** stand waardoor er geen requests meer worden uitgestuurd naar de service. In plaats van onnodig resources te gebruiken of  te wachten op timeouts wordt er direct een foutcode terug gegeven.  
Na een tijdje wordt de circuit breaker in **half-open** stand gezet waarbij hij een minimum van requests zal doorlaten terwijl voor het gros van de requests nog altijd direct een foutcode wordt terug gegeven. Als blijkt dat de enkele doorgelaten requests lukken en de service dus opnieuw in een goede staat is, wordt de circuit breaker **gesloten** en kunnen alle requests opnieuw doorgelaten worden. Lukken deze nog altijd niet voldoende, dan wordt de circuit breaker opnieuw in stand **open** gezet en wordt de cyclus herhaald.  
  
Afhankelijk van de noden kunnen verschillende thresholds ingesteld worden voor verschillende soorten communicatiefouten (bv time-outs tov connection refused).  
  
De standaard implementatie van dit pattern geeft direct een foutcode terug aan de client. De HTTP standaard voorziet om hiervoor status code 503 (Service Unavailable) te gebruiken (ipv een generieke timeout foutcode) zodat de client weet wat er gebeurt en hier gepast op kan reageren.   
Er kan ook verder gebouwd worden op het circuit breaking principe door extra mogelijkheden te voorzien wanneer het systeem in de open stand staat. In plaats van een foutcode terug te geven kan er ook een standaard response, de laatste goede response of een response uit cache terug gegeven worden.   
Indien een nog responsiever systeem gewenst is, kan er ook aan load-balancing gedaan worden waarbij de request wordt gererout naar een andere instantie van de service.
