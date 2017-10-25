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
teamName: String,
teamAproved: Boolean,
teamPoints: Number
}
```
</td>

<td>

#### Quiz

```
{
_id: Objectid:
quizPass: String,
Teams: [Schema.Types.ObjectId],
quizRounds: number
}
```

</td>
</tr>
</table>

### Bootstrap

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

`app.post('/scoreboards/rounds')` post het aantal gespeelde rondes nadat de quiz afgelopen is.

`app.post('/scoreboards/scores')` post de scores die teams hebben behaald wanneer de quiz stopt


### Node.js

### SuperAgent

### Websockets ( ws )

### React.js

### Routing

### Sessions