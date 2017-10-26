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

1. Wanneer de Quiz master een wachtwoord heeft ingevuld en daarna op de knop "Start quizz" drukt wordt deze via AJAX `request.post('/api')` naar de server gestuurd. De server handeld de request af met de express route `app.post('/api')`, in de route wordt in mongoDB een nieuwe entry gemaakt voor een nieuwe quizz. Als er al een quizz bestaat met hetzelfde wachtwoord wordt een JSON reponse terug gestuurd met daarin het volgende bericht `{ message: 'Er bestaat al een quiz met het zelfde wachtwoord'}`, zoniet dan wordt het volgende scherm getoond.

* Kan een team keuren (goedkeuren / rejecten)
![Mockup quizmaster 2](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/2.PNG)

1. Als een team zich heeft aangemeld en alles goed is verlopen, krijgt de Quiz master via een JSON websocket message de team naam binnen in het volgende formaat `message.teamName`.

2. De Quiz master heeft de keuze om een team te accepteren of te weigeren, door op een van de twee te knoppen wordt de keuze bevestigd. Door een AJAX put request in het volgende formaat `request.put('api/teams').send({ approvel: Boolean})` naar de express route `app.put('api/teams')` te sturen, word er in mongoDB eerst het bijhorende record van het team gevonden en daarna geupdate op basis van de gemaakte keuze. Als de Quiz master `true` heeft gegeven veranderd er niks bij het team(mischien response terug van dat je mag meedoen).Maar als het `false` is wordt er het volgende JSON reponse terug gestuurd `{message: Je mag niet mee doen}`.

3. Nadat alle teams zijn beoordeelt door de Quiz master dan pas wordt het volgende scherm geladen als de Quizmaster op de "Verder knop" drukt.

* 3 categoriën selecteren, worden twaalf vragen uit gekozen
![Mockup quizmaster 3](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/3.PNG)

1. Het eerste wat op dit scherm wordt uitgevoerd is een AJAX get request in het volgende formaat `request.get('api/cats')`. Op de server handelt de express route `app.get('api/cats')` de request af, deze haalt alle categorieën vanuit mongoDB met mongoose. De categorieën worden dan als response teruggestuurd naar de cliënt en laat ze zien aan de Quizmaster.

2. Uit alle categoriën kiest de Quizmaster er drie uit. wanneer de Quizmaster op de knop "Verder" drukt word er een AJAX get request in het volgende formaat `request.get('api/cats/:id')` naar de express route `app.get('api/cats/:id')` op de server gestuurd. Deze route haalt 4 vragen van een categorie op uit de mongoDB database d.m.v mongoose en dit word terugstuurd in json in het volgende formaat `{[{"question": String, answer:String ,category: String}]}`. De response van de server word opgeslagen in een session zodat we het meerdere keren kunnen gebruiken.


* “Start” knop dat de vraag start en zichtbaar maakt op het scoreboard en de team app
![Mockup quizmaster 4](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/4.PNG)

1. Er worden drie kolomen getoond met daarboven de gekozen categoriën, in iedere kolom bevinden zich 4 vragen van de categorie. Deze vragen waren opgehaald in het vorige scherm en bewaard in de session van de Quizmaster.

2. Als de Quizmaster op de knop "Verder" drukt, vestuurd hij of zij een JSON websocket message in  het volgende formaat naar de teams en het scoreboards `websocket.send({"question": String})`. Nadat het JSON websocket message is verstuurd word de huidige vraag met het antwoord opgeslagen in de session van de Quizmaster. Als laaste word het volgende scherm getoond.

* Kan de volgende vraag kiezen ( nieuwe vraag kiezen )
![Mockup quizmaster 5](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/5.PNG)

1. Het eerste wat wordt getoond zijn de gestelde vraag en het antwoord, in het vorige scherm waren deze opgeslagen in de session van de Quizmaster. De vraag en het antwoord worden uit de session gehaald en getoond op het scherm van de Quizmaster.

2. nog iets invullen over de team namen

3. Als een team een antwoord heeft ingevuld krijgt de Quizmaster deze binnen via een websocket message in het 
JSON formaat,dit bericht ziet er dan uit zo uit `websocket.send({teamName: String, answer: String})`. Todat de quizmaster op de knop "Vraag sluiten" drukt hebben de teams de mogelijkheid om hun antwoord te wijzigen.

4. De knop "Vraag sluiten" zorgt ervoor dat de teams hun vragen niet meer kunnen wijzigen, dit gebeurt door een JSON websocket message te sturen naar alle teams met daarin `websocket.send({open: Boolean})`.

5. Hier heeft de Quizmaster de keuze om een vraag van een team goed of fout te keuren. Als de Quizmaster een vraag heeft fout of goed heeft gekeurd, dan wordt er een JSON websocket message gestuurd naar de scoreboard in het volgende formaat `websocket.send({teamName: String, correct: Boolean})`. Als de boolean `True` is wordt er een 1 bijgeschreven bij de correcte antwoorden van een team. Bij `False` wordt er niks gedaan met de JSON websocket message.

6. Wanneer de Quizmaster alle vragen heeft gekeurd hoort hij of zij op de knop "Verder" te drukken, hierdoor wordt de volgende vraag geselecteerd in de lijst en begint het gehele proces weer overnieuw. In de session van de Quizmaster wordt er bijgehouden hoeveel vragen er al zijn geweest. Na vraag 12 wordt het volgende scherm getoond aan de Quizmaster.

* Nog een keer spelen knop
![Mockup quizmaster 7](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/7.PNG)

1. Na de 12 vragen is hier te zien welk team op dat moment de winnaar is van de quiz. Deze informatie wordt verkregen doordat de Scoreboard iedere ronde het team met het hoogste aantal punten opstuurt naar de Quizmaster via een JSON websocket message. 

2. De knop "Stop" kan gebruikt worden door de Quizmaster om de quizz te beindigen, er wordt dan via een AJAX put request gestuurd naar de server. De call wordt gestuurd in het volgende formaat `request.put('api/:id').send({Isfinished: True})` naar expess route `app.put('api/:id')`. Het JSON response is als volgt `{Isfinished: True}` en dit wordt door gestuurd naar de teams en het scoreboard via het volgende JSON websocket message `websocket.send({Isfinished: True})`.

3. Als de Quizmaster besluit om nog een ronde te spelen dan kan hij of zij drukken op de knop "Speel opnieuw", dit vestuurd een JSON websocket message naar de teams en het scoreboard. HIER NOG UITWERKEN NA BESCHRIJVEN SCOREBOARD TEAM

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
Isfinished: {Type:Boolean, required: true}
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

`app.get('/quizmaster/categories/questions')` op basis van de drie geselecteerde categorien worden er 12 vragen uit de database gehaald

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

`request.get('/quizmaster/categories/questions')`  vestuurd de drie gekozen categorein op naar de server en krijgt 12  vragen terug

##### Teams

`request.post('/teams')`  verstuurd de naam van het team naar de server en daar wordt het opgeslagen

`request.put('/teams/approval')` update de record in mongoose op met aprove of false, dit wordt gedaan door de quizmaster

##### Scoreboard

`request.put('/scoreboards/rounds')` update het aantal gespeelde rondes

`request.put('/scoreboards/scores')` update de scores van alle teams die hebben meegedaan met de quiz

### Websockets ( ws )

##### Quizmaster



### React.js

We hebben React toegepast om de applicatie zo vloeiend mogelijk te laten lopen, zonder dat de pagina vaak refreshed moet worden. Onze Quizzer app is onderverdeeld in drie verschillende single-page React applicaties. De benodigde informatie wordt opgehaald via onze ‘/api’ applicatie.

#### /qm
Het `/qm` scherm is  de react applicatie die verantwoordelijk is voor het tonen van de informatie die van belang is voor de quizmaster.

#### /team
Het `/team` scherm is de react applicatie die verantwoordelijk is voor het team.

#### /score
Het `/score` scherm is het scoreboard.


### Routing

@@@@@@@@@@

### Sessions

##### Quizmaster

De quizmaster houd bij welke antwoord een team heeft gegeven op een vraag, daarnaast huidige vraag en het antwoord(Iedere ronde worden deze gereset)!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

##### Teams

in de session van een team wordt de team naam opgeslagen en of een vraag al is beantwoord(mischien nog niet zeker)

##### Scoreboard

Als eerste zijn wij van plan om in de session op te slaan, hoeveel rondes er al zijn gespeeld en het aantal beantwoorde vragen in de ronde\. Verder wordt het aantal punter per team nog opgeslagen in een session en hoeveel vragen een team correct heeft beantwoord in een ronde.