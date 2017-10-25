# Quizzer Design

## Korte uitleg over Quizzer

The Quizzer is a een web applicatie die gebruikt kan worden in cafe's, sportkantines of andere plekken om quizzes te spelen als team.Een typisch gebruik in een bar ziet er als volgt uit:

* Spelers worden in teams gestopt. Een typisch team zou moeten bestaan uit drie tot en met acht spelers, maar dit is niet kritisch voor de applicatie.
* Een Quizz Night heeft meestal twee tot zes teams die deelnemen.
* Een Quizz Night bestaat uit meerdere rondes, elke ronde bevat twaalf vragen, gekozen uit drie            verschillende categorieën.
* Naast de teams is er ook een Quizz Master die vragen selecteert, antwoorden goed of fout keurt en        houdt de stemming erin met behulp van humor en enthousiasme.
* Elk team deelt een tafel en gebruikt een smartphone van een teamlid om vragen te beantwoorden.
* De Quizz Master gebruikt een tablet om het spel te hosten.
* Ten slotte wordt een groot scherm (bijvoorbeeld een beamer) gebruikt om de score van iedereen te tonen,  de huidige vraag en de teams die deze vraag hebben beantwoord.
* De vragen van Quizzer moeten in het kort beantwoord worden. Het zijn geen meerkeuze vragen maar dat      betekend niet dat er lange zinnen nodig zijn
* De Quizz Master bepaalt wanneer een Quizz Night over is.En als het dan zo ver is, selecteert de          applicatie de winnaars op basis van de resultaten

## Mock-ups

### Team-app

* Veld teamnaam
![Mockup team-app 1](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/991.PNG)

* Vraag invullen, antwoord geven
![Mockup team-app 2](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/992.PNG)

### Quizmaster

* Start de Quizz Night en opent het voor teams.
![Mockup quizmaster 1](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/1.PNG)

* Kan een team keuren (goedkeuren / rejecten)
![Mockup quizmaster 2](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/2.PNG)

* 3 categoriën selecteren, worden twaalf vragen uit gekozen
![Mockup quizmaster 3](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/3.PNG)

* “Start” knop dat de vraag start en zichtbaar maakt op het scoreboard en de team app
![Mockup quizmaster 4](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/4.PNG)

* Kan de volgende vraag kiezen ( nieuwe vraag kiezen )
![Mockup quizmaster 5](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/5.PNG)

* Antwoord lezen en goedkeuren, afkeuren
![Mockup quizmaster 6](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/6.PNG)

* Nog een keer spelen knop
![Mockup quizmaster 7](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/7.PNG)

### Scoreboard app

* laat zien hoeveel rondes zijn gespeeld en hoeveel vragen er in de ronde zaten
* De team namen met hun scores in "Round points" en de hoeveelheid vragen ze correct hebben beantwoord.
* Round points worden toegewezen op deze manier: Na iedere ronden van twaalf vragen krijgt het team dat de meeste vragen goed beantwoord heeft, 4 Round Points. Het volgende beste team krijgt 2 Round points en het derde team krijgt 1 Round point. Alle andere teams krijgen 0.1 round points
* Wanneer een vraag nog bezig(nog wordt beantwoord door de teams) is wordt de volgende informatie getoond: De vraag, de categorie van de vraag en welke teams die het antwoord op de vraag hebben ingevuld maar nog niet hebben gesubmit.

![Mockup scoreboard 1](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/Scoreboard%20part%201.PNG)

* Zodra een antwoord is goedgekeurd of afgewezen, worden alle vragen getoond van de teams en wanneer goedgekeurd of afgewezen word de score bijgewerkt.

![Mockup scoreboard 2](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/Scoreboard%20part%202.PNG)

## Technieken

### Mongoose Data Schema's

Als team hadden we het besluit genomen om twee schema's te gebruiken, namelijk een voor de Teams en de ander voor de Quiz zelf.

<table>
<tr>

<td>

#### Team

```
{
_id: Objectid,
teamName: {Type:String,required: true},
teamAproved: {Type:Boolean,required: true},
teamPoints: {Type:Number,required: true}
}
```
</td>

<td>

#### Quiz

```
{
_id: Objectid:
quizPass: {Type:String,required: true},
Teams: {Type:[Schema.Types.ObjectId],required: true},
quizRounds: {Type:Number,required: true}
}
```

</td>
</tr>
</table>

### Bootstrap

We hebben bootstrap toegepast om de front-end van onze web applicatie eenvoudiger op te zetten dmv een grid-systeem, dit heeft ons geholpen de webapplicatie snel responsive te maken.

### SCSS

Als variant op CSS hebben we SCSS toegepast, we vinden als team dat SCSS overzichtelijker is om mee te werken dan CSS. Daarnaast is het een stuk uitgebreider dan standaard CSS.

### Express

Wij hadden als team besloten om express te gebruiken om verschillende resource based routes aan te maken. Deze verschillende resource based routes worden hieronder in het kort beschreven:

`app.get('/')` toont de homepage en hier krijg je de keuze om als team te spelen of om een quiz te starten

##### Quizmaster

`app.get('/quizmasters')` toont het scherm voor de quizmaster

`app.get('/quizmasters/categories')` haalt alle categorien op uit de database

`app.post('/quizmaster/categories/questions')` op basis van de drie geselecteerde categorien worden er 12 vragen uit de database gehaald

`app.post('/quizmasters')` verstuurt het wachtwoord en maakt een nieuwe quiz aan

##### Teams

`app.get('/teams')` toont het scherm voor de teams

`app.post('/teams')` voegt een team toe aan de mongoDB database als deze nog niet bestaat

`app.put('/teams/approval')` wordt door de quizmaster gebruikt om een team goed of fout te keuren


##### Scoreboard

`app.put('/scoreboards/rounds')` post het aantal gespeelde rondes nadat de quiz afgelopen is.

`app.put('/scoreboards/scores')` post de scores die teams hebben behaald wanneer de quiz stopt


### Node.js

### SuperAgent

Superagent word door de client gebruikt om data te versturen of op te halen van de server, de server handeld deze requests af.

##### Quizmaster

`request.get('quizmasters/categories')` haalt alle categorien en toont deze op het scherm van de quizmaster

`request.post('/quizmasters')` vertuurd een wachtwoord voor de quiz naar de server en start daarna de quiz

`request.post('/quizmaster/categories/questions')`  vestuurd de drie gekozen categorein op naar de server en krijgt 12  vragen terug

##### Teams

`request.post('/teams')`  verstuurd de naam van het team naar de server en daar wordt het opgeslagen

`request.put('/teams/approval')` update de record in mongoose op met aprove of false, dit wordt gedaan door de quizmaster

##### Scoreboard

`request.put('/scoreboards/rounds')` update het aantal gespeelde rondes

`request.put('/scoreboards/scores')` update de scores van alle teams die hebben meegedaan met de quiz

### Websockets ( ws )

##### Quizmaster



### React.js

We hebben React toegepast om de applicatie zo vloeiend mogelijk te laten lopen, zonder dat de pagina vaak refreshed moet worden.

### Routing

@@@@@@@@@@

### Sessions

##### Quizmaster

De quizmaster houd bij in de session hoeveel teams er al een vraag hebben beantwoord.!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

##### Teams

in de session van een team wordt de team naam opgeslagen en of een vraag al is beantwoord(mischien nog niet zeker)

##### Scoreboard

Als eerste zijn wij van plan om in de session op te slaan, hoeveel rondes er al zijn gespeeld en het aantal beantwoorde vragen in de ronde\. Verder wordt het aantal punter per team nog opgeslagen in een session en hoeveel vragen een team correct heeft beantwoord in een ronde.