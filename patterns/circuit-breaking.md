# Circuit Breaking

![circuit-breaker-zekering](/img/circuit-breaker.png)

Toepassingen in een enterprise architectuur zijn opgebouwd uit een netwerk van services. 
Een handeling van een gebruiker leidt in de meeste gevallen tot een keten van requests tussen services. Bij deze communicatie tussen services kan er heel wat mislopen. Een request kan mislukken doordat de service niet beschikbaar is of er kan een time out optreden omdat de response langer op zich laat wachten dan verwacht. Wanneer dit niet opgevangen wordt, kan de hele keten geïmpacteerd zijn. Bovendien is er een groter risico op overload van het systeem doordat er resources gebruikt worden voor requests die niet lukken. In de meeste gevallen zal een client een request uitvoeren en op de response wachten. Als deze response niet onmiddellijk komt, wordt er gewacht tot de time out periode verlopen is. In deze wacht periode zal de client belangrijke resources zoals database connecties, threads en memory bezet houden. Als er dan bovendien in parallel veel van deze requests worden opgestart, stijgt het aantal geblokkeerde resources exponentieel. 

Het is daarom aangeraden dat een client zo veel mogelijk vermijdt requests te doen voor iets dat zeer waarschijnlijk zal falen en zodoende niet onnodig resources en CPU cycles gebruikt.

Bij het toepassen van een circuit breaking pattern wordt er voor gezorgd dat een service die onbeschikbaar of te traag is geworden, tijdelijk niet meer wordt aangesproken. In plaats van de requests naar die service te blijven proberen wordt de gebruiker of downstream client direct verwittigd dat er problemen zijn. Dit zorgt er voor dat de clients veel sneller op de hoogte zijn van de problemen (er wordt niet op timeouts gewacht) en er worden minder resources gebruikt om requests uit te voeren die niet lukken. 

![circuit-breaking](/img/circuit-breaking-pattern.png)

Wanneer de circuit breaker geopend is, zal deze regelmatig checken of de service opnieuw beschikbaar is. Van zodra de service opnieuw requests op een normale manier kan afhandelen, sluit de circuit breaker het circuit en kunnen de requests opnieuw doorgestuurd worden naar de service.

Een voorbeeld van een implementatie kan je [hier](/implementaties/circuit-breaking-implementatie.md) vinden.