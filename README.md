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

## Applicatiebeschrijving en mock-ups

### Team-app

* Veld teamnaam
![Mockup team-app 1](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/991.PNG)

1. Wanneer iemand wil meespelen met een quiz moet de persoon of groep mensen eerst een team naam en het wachtwoord invullen. Het wachtwoord van een quiz hoort de persoon of groep mensen te krijgen van de Quizmaster.

2. Als de persoon of groep alle velden heeft ingevuld en drukt op de knop “Verder” dan word er via AJAX `request.post(‘api/teams’)` gestuurd naar de server. De server handelt het request af met de express route `app.post(‘api/teams’)`, in deze route wordt er gecheckt in mongoDB of het wachtwoord overeenkomt met een quiz en tegelijk wordt er gekeken of er al een team bestaat met de zelfde naam.Als een van de twee al voorkomt stuurt de route het volgende JSON response terug `{ message: “Team naam bestaat al of er bestaat geen quiz met dat wachtwoord”}`.Zo niet dan wordt een nieuwe team document aangemaakt in mongoDB.

```
## Express route

app.post('api/teams', function (req, res) {
  // Checkt in mongoDB of het wachtwoord overeenkomst met een quiz

  if(// Als een van de twee al voorkomt) {
    res.send({ message: “Team naam bestaat al of er bestaat geen quiz met dat wachtwoord”});
  }
  else {
    // Team wordt toegevoegd aan het "Team" schema van MongoDB ( zie Mongoose data schema's )
  }
})
```


* Vraag invullen, antwoord geven
![Mockup team-app 2](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/992.PNG)

1. Het aantal vragen dat is geweest wordt opgeslagen in de session van een team. Als er weer een nieuwe vraag binnen komt van de Quizmaster dan wordt er weer 1 bijgeteld bij het totale aantal. Na een ronde wordt dit weer gereset.

2. De vraag en categorie krijgt een team binnen van de Quizmaster d.m.v het volgende de websocket message `websocket.send({"question": String, category: 'String'})`.

3. Uit het input veld word het antwoord op de vraag van een team gehaald, dit gebeurd waarschijnlijk d.m.v `document.getElementById(String).value`.Wanneer een team een antwoord heeft ingevuld in het input veld maar nog niet op "Verder" heeft geklikt word het volgende JSON websocket message naar het scoreboard gestuurd `websocket.send({teamName: String, answered: Boolean})`. Als er nog niks in ingevuld dan staat `answered` op `False` maar na het invullen van het input veld word het `True`.

4. Als een team een antwoordt heeft ingevuld in het input veld en op de knop “Verder” drukt dan wordt er het volgende JSON websocket message `websocket.send({“teamName”: String, “question”: String, answer: String})` verstuurd naar de Quizmaster.<br><br> De Quizmaster kan het antwoord zien van ieder team, als een team hun antwoord wil aanpassen dan hebben ze hiervoor de mogelijkheid totdat de Quizmaster de vraag sluit. Het laatst  verstuurde antwoord is dan het definitieve antwoord van een team. Het team kan dan ook niks meer submitten en krijgt een pop-up te zien met het volgende bericht `Vraag gesloten`.


### Quizmaster

* Start de Quizz Night en opent het voor teams.
![Mockup quizmaster 1](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/1.PNG)

1. Wanneer de Quiz master een wachtwoord heeft ingevuld en daarna op de knop "Start quizz" drukt wordt deze via AJAX `request.post('/api)` naar de server gestuurd. De server handeld de request af met de express route `app.post('/api')`, in de route wordt in mongoDB een nieuwe entry gemaakt voor een nieuwe quizz. Als er al een quizz bestaat met hetzelfde wachtwoord wordt een JSON reponse terug gestuurd met daarin het volgende bericht `{ message: 'Er bestaat al een quiz met het zelfde wachtwoord'}`, zoniet dan wordt het volgende scherm getoond.

```
## Express route

app.post('/api', function (req, res) {
  if (// als het wachtwoord al bestaat) { 
    req.post('/api').send({message: 'Er bestaat al een quiz met het zelfde wachtwoord'})
  }
  else {
    // Maakt een nieuwe mongoDB entry aan voor een nieuwe quizz.
  }
})
```

* Kan een team keuren (goedkeuren / rejecten)
![Mockup quizmaster 2](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/2.PNG)

1. Als een team zich heeft aangemeld en alles goed is verlopen, krijgt de Quiz master via een JSON websocket message de team naam binnen in het volgende formaat `message.teamName`.

2. De Quiz master heeft de keuze om een team te accepteren of te weigeren, door op een van de twee te knoppen wordt de keuze bevestigd. Door een AJAX put request in het volgende formaat `request.put('api/teams').send({ approvel: Boolean})` naar de express route `app.put('api/teams')` te sturen, word er in mongoDB eerst het bijhorende record van het team gevonden en daarna geupdate op basis van de gemaakte keuze. Als de Quiz master `true` heeft gegeven veranderd er niks bij het team(mischien response terug van dat je mag meedoen).Maar als het `false` is wordt er het volgende JSON reponse terug gestuurd `{message: Je mag niet mee doen}`. het objectID van het team word opgeslagen in het bijbehorende quiz document  wanneer `{approvel: True}` is. Naast dit worden alle goedgekeurde teams opgeslagen in de session van de Quizmaster.

3. Nadat alle teams zijn beoordeelt door de Quiz master dan pas wordt het volgende scherm geladen als de Quizmaster op de "Verder knop" drukt.

```
## Express route

app.put('/api/teams', function (req, res) {
  // route ontvangt een boolean bij het goedkeuren of afwijzen van een team

  if (// als de boolean (approvel) true is) { 
    // Voegt het team toe in MongoDB
  }
  else {
    // Verstuurd een message, dat het team niet mag mee doen
  }
})
```

* 3 categoriën selecteren, worden twaalf vragen uit gekozen
![Mockup quizmaster 3](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/3.PNG)

1. Het eerste wat op dit scherm wordt uitgevoerd is een AJAX get request in het volgende formaat `request.get('api/cats')`. Op de server handelt de express route `app.get('api/cats')` de request af, deze haalt alle categorieën vanuit mongoDB met mongoose. De categorieën worden dan als response teruggestuurd naar de cliënt en laat ze zien aan de Quizmaster.

```
## Express route

app.get('/api/cats', function (req, res) {
  // Haalt alle categoriën op vanuit MongoDB met mongooose

  res.send(// Verstuurd de categorieën als response terug naar de cliënt);
})
```

2. Uit alle categoriën kiest de Quizmaster er drie uit. wanneer de Quizmaster op de knop "Verder" drukt word er een AJAX get request in het volgende formaat `request.get('api/cats/:id')` naar de express route `app.get('api/cats/:id')` op de server gestuurd. Deze route haalt 4 vragen van een categorie op uit de mongoDB database d.m.v mongoose en dit word terugstuurd in json in het volgende formaat `{[{"question": String, answer:String ,category: String}]}`. De response van de server word opgeslagen in een session zodat we het meerdere keren kunnen gebruiken.

```
## Express route

app.get('api/cats/:id', function (req, res) {
  // Haalt alle categoriën op vanuit MongoDB met mongooose

  res.send({[{"question": String, answer:String ,category: String}]} // Verstuurd de 4 categoriën in JSON);
})
```

* “Start” knop dat de vraag start en zichtbaar maakt op het scoreboard en de team app
![Mockup quizmaster 4](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/4.PNG)

1. Er worden drie kolomen getoond met daarboven de gekozen categoriën, in iedere kolom bevinden zich 4 vragen van de categorie. Deze vragen waren opgehaald in het vorige scherm en bewaard in de session van de Quizmaster.

2. Als de Quizmaster op de knop "Verder" drukt, vestuurd hij of zij een JSON websocket message in  het volgende formaat naar de teams en het scoreboards `websocket.send({"question": String, category: 'String'})`. Nadat het JSON websocket message is verstuurd word de huidige vraag met het antwoord opgeslagen in de session van de Quizmaster. Als laaste word het volgende scherm getoond.

* Kan de volgende vraag kiezen ( nieuwe vraag kiezen )
![Mockup quizmaster 5](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/5.PNG)

1. Het eerste wat wordt getoond zijn de gestelde vraag en het antwoord, in het vorige scherm waren deze opgeslagen in de session van de Quizmaster. De vraag en het antwoord worden uit de session gehaald en getoond op het scherm van de Quizmaster.

2. Alle teams worden uit de session gehaald van goedgekeurde team. en getoond op het scherm

3. Als een team een antwoord heeft ingevuld krijgt de Quizmaster deze binnen via een websocket message in het 
JSON formaat,dit bericht ziet er dan uit zo uit `websocket.send({teamName: String, answer: String})`. Todat de quizmaster op de knop "Vraag sluiten" drukt hebben de teams de mogelijkheid om hun antwoord te wijzigen.

4. De knop "Vraag sluiten" zorgt ervoor dat de teams hun vragen niet meer kunnen wijzigen, dit gebeurt door een JSON websocket message te sturen naar alle teams met daarin `websocket.send({open: Boolean})`.

5. Hier heeft de Quizmaster de keuze om een vraag van een team goed of fout te keuren. Als de Quizmaster een vraag heeft fout of goed heeft gekeurd, dan wordt er een JSON websocket message gestuurd naar de scoreboard in het volgende formaat `websocket.send({teamName: String, answer: String, correct: Boolean})`. Als de boolean `True` is wordt er een 1 bijgeschreven bij de correcte antwoorden van een team. Bij `False` wordt er niks gedaan met de JSON websocket message.

6. Wanneer de Quizmaster alle vragen heeft gekeurd hoort hij of zij op de knop "Verder" te drukken, hierdoor wordt de volgende vraag geselecteerd in de lijst en begint het gehele proces weer overnieuw. In de session van de Quizmaster wordt er bijgehouden hoeveel vragen er al zijn geweest. Na vraag 12 wordt het volgende scherm getoond aan de Quizmaster. het volgende JSON websocket message wordt verstuurd `websocket.send({nexRound: Boolean})`;

* Nog een keer spelen knop
![Mockup quizmaster 7](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/7.PNG)

1. Na de 12 vragen is hier te zien welk team op dat moment de winnaar is van de quiz. Deze informatie wordt verkregen doordat de Scoreboard iedere ronde het team met het hoogste aantal punten opstuurt naar de Quizmaster via een JSON websocket message. 

2. De knop "Stop" kan gebruikt worden door de Quizmaster om de quizz te beindigen, er wordt dan via een AJAX put request gestuurd naar de server. De call wordt gestuurd in het volgende formaat `request.put('api/:id').send({Isfinished: True})` naar expess route `app.put('api/:id')`. Het JSON response is als volgt `{Isfinished: True}` en dit wordt door gestuurd naar de teams en het scoreboard via het volgende JSON websocket message `websocket.send({Isfinished: True})`.

```
## Express route

// Wanneer de quizmaster de quizz stopt
app.put('api/:id') {
  res.send({Isfinished: True});
})
```

3. Als de Quizmaster besluit om nog een ronde te spelen dan kan hij of zij drukken op de knop "Speel opnieuw", dit vestuurd een JSON websocket message naar de teams en het scoreboard. HIER NOG UITWERKEN NA BESCHRIJVEN SCOREBOARD TEAM

### Scoreboard app

* laat zien hoeveel rondes zijn gespeeld en hoeveel vragen er in de ronde zaten
* De team namen met hun scores in "Round points" en de hoeveelheid vragen ze correct hebben beantwoord.
* Round points worden toegewezen op deze manier: Na iedere ronden van twaalf vragen krijgt het team dat de meeste vragen goed beantwoord heeft, 4 Round Points. Het volgende beste team krijgt 2 Round points en het derde team krijgt 1 Round point. Alle andere teams krijgen 0.1 round points
* Wanneer een vraag nog bezig(nog wordt beantwoord door de teams) is wordt de volgende informatie getoond: De vraag, de categorie van de vraag en welke teams die het antwoord op de vraag hebben ingevuld maar nog niet hebben gesubmit.

![Mockup scoreboard 1](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/Scoreboard%20part%201.PNG)

1. Om erachter te komen hoeveel rondes er zijn gespeeld wordt er in de session van het scoreboard bewaard hoeveel rondes er al zijn gespeeld. Als de Quizmaster de keuze maakt om nog een ronde te spelen krijgt het scoreboard een JSON websocket message binnen, dit bericht ziet er zo uit `{nexRound: Boolean}`. Als het `True` is, wordt er een 1 opgeteld in de session en bij `false` niet.

2. Dit veld houdt bij hoeveel vragen er al in een ronde zijn beantwoord, de hoeveelheid gestelde vragen wordt in de session van het scoreboard bewaard. Bij iedere nieuwe vraag stuurt de Quizmaster een JSON websocket message, als deze binnen komt wordt erin de session 1 opgeteld. Zo krijgt het scoreboard een actueel beeld van hoeveel vragen er al zijn beantwoord.

3. Om de team namen te kunnen tonen heeft de Quizmaster een JSON websocket bericht gestuurd naar het scoreboard in het volgende formaat `websocket.send({teamNames: [String]})`. Het scoreboard haalt alle team namen uit dit bericht en slaat ze op in de session van het scoreboard. Hierna worden ze getoond op het scherm.

4. Na iedere vraag heeft de Quizmaster het volgende JSON websocket message gestuurd naar het scoreboard `websocket.send({teamName: String, answer: String, correct: Boolean})`. Als er in het bericht `correct: True` staat wordt het nummer van "Correcte antwoorden" van een team verhoogt met 1. Dit wordt bewaard in de session van het scoreboard en na iedere ronde wordt het gereset.

5. Netzo als bij stap 1 word het aantal punten van een team berekend als het scoreboard het volgende JSON websocket message binnen krijgt `{nexRound: True}`. Het team met het meeste aantal goede antwoorden in een ronde krijgt 4 round points bij het totaal, de nummer twee krijgt 2 round points en het derde team krijgt 1 round points. Alle andere teams krijgen 0.1 round points. Al deze punten worden bijgehouden in de session van het scoreboard.

6. De huidige vraag en categorie krijgt het scoreboard van de Quizmaster, ze zijn verstuurd via het volgende JSON websocket message `websocket.send({"question": String, category: 'String'})`. De informatie uit de websocket message is opgeslagen in een session en word getoond op scherm. Als de volgende vraag binnenkomt word de inhoud van de session vervangen.

7. Als een team een antwoord heeft ingevuld in het input veld dan krijgt het scoreboard het volgende JSON websocket message binnen `websocket.send({teamName: String, answered: Boolean})`. Wanneer `answered: True` is dan word de teamnaam getoond en als het `answered: False` is niet.

* Zodra een antwoord is goedgekeurd of afgewezen, worden alle vragen getoond van de teams en wanneer goedgekeurd of afgewezen word de score bijgewerkt.

![Mockup scoreboard 2](https://raw.githubusercontent.com/TimFrak/quizzer-temp/master/quizz_master_app/Scoreboard%20part%202.PNG)

1. Wanneer de Quizmaster een vraag heeft gesloten veranderd het scherm. Zo is er te zien welke antwoord ieder team heeft gegeven en of deze goed of fout zijn gekeurd. Dit haalt het scherm op door zijn eigen session te raadplagen want hierin wordt dit bewaard in de vorm van een array. Deze array wordt dan uitgelezen en op basis daarvan krijgt een team een bepaald aantal punten.

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
<td>

#### Vragen

```
{
_id: getNextSequenceValue(value), //Auto-Increment
question: {Type:String,required: true},
answer: {Type:String,required: true},
category: {Type:String,required: true}
}
```

</td>
</tr>
</table>

### Bootstrap

We hebben bootstrap toegepast om de front-end van onze web applicatie eenvoudiger op te zetten dmv een grid-systeem, dit heeft ons geholpen de webapplicatie snel responsive te maken.

### SCSS

Als variant op CSS hebben we SCSS toegepast, we vinden als team dat SCSS overzichtelijker is om mee te werken dan CSS. Daarnaast is het een stuk uitgebreider dan standaard CSS.

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

### Sessions

##### Quizmaster

`session.questionsRound = Array;` // bewaard alle twaalf de vragen van de drie verschillende categorieën in een array.

`session.approvedTeams = Array;` // bewaard alle teams die zijn goedgekeurd door de Quizmaster.

`session.currentQuestion = Object;` // bewaard de huidige vraag en categorie van een ronde.

##### Teams

`session.currentQuestion = Object;` // bewaard de huidige vraag en categorie van een ronde.

##### Scoreboard

`session.roundPlayed = Numbers;` //+1 iedere ronde.

`session.roundQuestions = Numbers;` //+1 na iedere gestelde vraag.

`session.teamNames = Array;` //bewaard alle namen van de teams die meedoen in een quiz.

`session.teamCorrectAnswers` = Array; //bewaard het aantal correcte antwoorden van alle teams in een ronde.

`session.teamPoints = Array;` //bewaard het totaal aantal punten van alle teams in een quiz.

`session.roundCurQuestion = Object;` //bewaard de huidige vraag en categorie in een object, word iedere ronde overschreven.
