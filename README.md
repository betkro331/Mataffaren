## Mataffaren API

### Projektbeskrivning:
Detta projekt innehåller en liten matbutik där tester skrivits och körts i Postman, Newman, Github Actions och Hoppscotch.

### Postman, Newman och GithubActions
Projektet har använt Postman för att genomföra manuella och automatiska tester. Data är organiserad i en Postman Collection så att alla tester kan köras med "Run Collection". De flesta tester kan också köras var för sig. Från Postman är collection exporterad och testerna kan köras med Newman i terminalen. Git commit pull och push triggar igång testerna i Github Actions.

### Manuella tester och kartläggning av API:t 
Kartläggningen av API:t finns beskrivet i excelarket "Kartläggning API Mataffären" som en bilaga till detta repo. Där beskrivs kortfattat routes, vilken information som kan hämtas, vilket svar som förväntas vid en GET-request samt om GUI:t ger samma svar som Postman-requesten. Jag har inte tittat igenom alla kategorier utan koncenterat mig på vissa. Jag hittade inga uppenbart stora skillnader, men reagerade på att jämförpris blev konstigt eftersom Mataffären jämför pris/kg eller pris/liter med pris/styck inom samma produktkategori. Ibland fanns också något specialerbjudanden så som "Välj & blanda! 2 för" och då fanns två olika 'comparePrice' under samma produkt, vilket gjorde jämförelsen svårare. Gällande "popularitet" (ranking) fanns inte alla varor med. Många varor har '0' eller 'null' i ranking, vilket gjorde listan efter popularitet lite missvisande. Jag använde både Firefox och Chrome dev tool för att jämföra hur Network/XHR var att arbeta med. Svarstiderna var mellan 200-400 msek vid sidladdning, vilket jag anser är rimligt.

### Automatiserade tester i Postman
För varje testfall i Postman anges en kort beskrivning av vad testet syftar till (återges bara generellt här i detta dokument). Testernas enskilda svarstider var rimliga, mellan 130-400 ms. Den slutgiltiga collection i "runner" tog 10 s 960 ms vilket igenomsnitt blev 278 ms för 111 tester, även det en rimlig svarstid.

Jag gjorde totalt 11 GET-requests med mellan 3-7 tester i varje. Att statuskod "200 OK" returneras samt svarstider på under 500 respektive 1000 ms, har jag med i varje test. 
Andra testexempel är: 
 - att dataformatet är JSON
 - att minst 10 huvudkategorier finns
 - att alla huvudkategorier har minst en underkategori (förutom den sista, 'Lotter')
 - att några huvudkategorier har minst X antal underkategorier (childrens children)
 - att huvudkategorierna har vissa egenskaper såsom 'id', 'title' och 'url'
 - att en viss underkategori har X antal varor
 - att en underkategorier har minst X antal underkategorier
 - att de tre första varorna i en kategori (eller underkategori) har rätt namn
 - att en 'nyckel' som t ex categoryInfo inte är tom och/eller innehåller rätt värden
 - att verifiera olika datatyper som 'object', 'string' och 'number'
 - att några av produkterna i en lista har korrekt namn och kod
 - att en specifik produkt innehåller rätt namn, tillverkare och 'dietinformation'

 Jag gjorde lite olika tester och vissa av dem är hårdkodade. Inte den snyggaste koden kanske, men lärorikt för att se och lära hur det fungerar. 

*Tester på sortering*

Vid de olika sorteringsalternativen provade jag att göra på olika sätt i olika test. I de flesta test gjorde jag en kopia på responsdatan och sorterade den uppifrån/nerifrån beroende på, och jämförde sedan de två "listorna" för att se att de stämde överens. I ett test provade jag att istället använda mig av boolean genom att loopa och kolla att varans jämförpris är lägre eller samma som föregående vara. Om inte, så ändras variabeln isSorted till false. I slutet kontrolleras om isSorted är true, och i så fall blir testresultatet "pass".

Jag gjorde fem GET-requests på olika sortering:

#### Product list sorted by price (descending)
Innehåller ett test som verifierar att sortering på ”Pris (Dyrast-Billigast)” verkligen returnerar en sådan lista genom att jämföra responsen 'priceValue' och med en kopia sorterad i fallande ordning.

#### Product list sorted by popularity/ranking
Ett test verifierar att sortering på "Populärast" verkligen returnerar en rankad lista genom att jämföra responsen 'ranking' med en kopia sorterad från högsta till lägsta värde.

#### Product list sorted by compare price (ascending)
Ett test kontrollerar att sortering på "Jämförspris (Billigast-Dyrast)" returnerar en lista med jämförspris i rätt ordning genom att jämföra responsen 'comparePrice' med en kopia sorterad från lägsta till högsta värde.

#### Product list soda sorted by compare price (descending)
Testet loopar och kontrollerar att varje produkts 'comparePrice' är lägre eller lika med den förra produktens.

#### Product list sorted in alphabetic order (descending)
Testet verifierar att sortering på "Ö-A" returnerar en lista med produktnamnen sorterade från Ö till A. Bing Chat tipsade om att använda localeCompare('sv'), vilket jag gjorde, och sortering enligt svensk standard på ÅÄÖ fungerade bra.

### Slutsatser om testerna i Postman, Newman, Github Actions

Testerna gav förväntade resultat efter en del fixande och trixande. Jag använde mig av console.log väldigt mycket, och upptäckte på så sätt att vissa tester var "false positive" eftersom jag inte testade det jag trodde att jag testade eller att jag testade tomma strängar. Jag lade också in andra värden för att "testa testerna" så att de fallerade. 

Testerna gick igenom både enskilt och som "Run collection" i Postman. Det gick också bra att exportera och köra dem med Newman. I Github Actions kraschade testerna för att jag hade glömt att ta bort en massa console.log:ar. När jag utkommenterade dem i Postman och körde om hela proceduren fungerade det bra, förutom att ett test ibland fallerar i Github Actions eftersom tiden är satt till max 500 ms. Detta har jag inte ändrat eftersom jag ändå tycker att det är rimligt att ett test ska klara under 500 ms.


### Tester i Hoppscotch.io 
Jag testade att göra några enkla tester i Hoppscotch bara för att testa syntaxen och jämföra med Postman. Här är några test som gick igenom:

pw.test('The status code of the response should be 200', () => {
    pw.expect(pw.response.status).toBe(200);
})

pw.test("Response body is not null", () => {
  pw.expect(pw.response.body).not.toBe(null);
});

pw.test("Response body to be datatype object", () => {
  pw.expect(pw.response.body.results).toBeType("object");
});

pw.test('There should be at least 7 products in main category "frukt-och-gront"', () => {
    pw.expect(pw.response.body.results.length >= 7).toBe(true);
});

pw.test('There should be at least 5 products in the sub category "frukt"', () => 
{ pw.expect(pw.response.body.subCategories.length >=5).toBe(true); });

### Thomas test-motor baserat på Node.js + Chai.js
Jag hade tänkt att försöka lägga till några tester i detta projekt men jag kom inte åt Nodehills sajt under helgen 11-12/11 så jag bestämde mig för att hoppa över denna del och lämna in ändå.
