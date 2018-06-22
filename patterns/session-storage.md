# BFF Session Storage en Caching

![session-storage](/img/session-storage.jpg)

Een Backend-For-Frontend (BFF) heeft vaak nood aan persistent storage. Dit kan handig zijn om de (front-end) sessies van de ingelogde gebruikers in op te slaan of om een cache te implementeren. Het is echter niet de bedoeling om hier andere data in te bewaren. 

Om dit te realiseren mag een BFF een aparte PostgreSQL databank gebruiken voor zijn sessiebeheer en/of cache. 
Indien meerdere BFF's sessies moeten kunnen delen, dan mag dit één gedeelde PostgreSQL databank zijn. Let er wel op dat er een aparte service wordt gemaakt om updates of inserts te doen, en dat de andere services enkel leesacties (en updates van de TTL) doen.
