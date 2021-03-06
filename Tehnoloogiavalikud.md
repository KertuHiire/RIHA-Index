---
title: Tehnoloogiavalikute lähtekohad
permalink: Tehnoloogiavalikud
---

# Tehnoloogiavalikute lähtekohad
{:.no_toc}

Käesolev dokument:

- esitab lähtekohad ja põhimõtted tehnoloogiate valikuks
- fikseerib juba tehtud valikud
- annab viiteid tehnoloogiate kirjeldustele.

<div class='rem'>Dokumenti võiks nimetada ka RIHA tehnoloogiliseks profiiliks või tehnoloogiliseks plaaniks. Profiil &mdash; ülesande täitmiseks kehtestatud standardite kogum.
</div>

Tehnoloogiaportfelli täiendatakse arenduse käigus.

* TOC
{:toc}

## 2 Tehnoloogiate valiku lähtekohad ja põhimõtted

Tehnoloogiate valimisel tuleb arvestada eriti järgmist.

2.1 Üldiselt on eesmärk kasutada tänapäevaseid, efektiivseid arendusvahendeid ja tehnoloogiaid, üritades üle saada avalikult sektorile iseloomulikust inertsist.

2.2 Tehnoloogiate teisejärgulisus äriülesande täitmise suhtes. See, kui uut ja "ägedat" tehnoloogiat kasutatakse, ei ole alati nii oluline. Esmatähtis on, et ülesanne saab täidetud.

2.3 Tehnoloogiliste ummikteede ja sundseisude vältimine, niipalju, kui see on võimalik, unustamata, et üleliigne paindlikkus on investeering, mis sageli jääb kasutamata. 

2.4 Lokaalse optimeerimise vältimine, niipalju, kui see on võimalik.

2.5 Tehnoloogiad ja töövahendid peavad olema vabalt, litsentsitasudeta kasutatavad.

2.6 RIHA ärinõudeks on lihtsus. Eesmärgiks ei ole teha suurt, keerulist süsteemi. Seetõttu tuleb tehnoloogiliste valikute tegemisel küsida, kas ärilise eesmärgi võiks saavutada lihtsama tehnoloogiaga. Teiste sõnadega, kas võimsam, aga samas keerukam tehnoloogia toob konkreetsel juhul väärtust.

2.7 RIHA rajatakse komponentidena. RIHA komponenti iseloomustab:

- selge funktsioon, soovitavalt ühe (või mitme üksteisega seotud) funktsiooni täitmine.
- eraldipaigaldatavus, sh erinevatesse keskkondadesse
- andmete pakkumine masinliidese abil
- moodulid liidestatakse üksteisega ja avatakse liidestusteks välistele süsteemidele REST API-de abil.
- moodulite kogum peab olema laiendatav. RIHA strateegias on sõnastike, projektide, finantssjuhtimise moodulite perspektiivne lisamine.

<div class='block__note'>
  <p class='block__note--heading'>Väljavahetatavad moodulid</p>
  <p>Moodulprintsiibi tähtis aspekt on moodulite väljavahetatavus. Moodulid tuleb projekteerida nii väikesteks, et vajadusel saab mooduli välja vahetada, teostades selle teises programmeerimiskeeles vm teisel tehnoloogia alusel. Ideaaljuhul nagu To-Do rakendus, mida saab igas programmeerimiskeeles teostada.</p>
</div>

2.8 Komponentide kokkusobivus ja tervikuna toimimine tagatakse muuhulgas:

- kasutajaliideste ühtlustatud kujunduspõhimõtetega
- liideste (API-de) täieliku ja täpse dokumenteerimisega.
  
2.9 Komponentide paigaldatavus erinevates keskkondades   

- RIHA on hajussüsteem. Komponentrakendused paigaldatakse erinevate asutuste kontrolli all olevatesse keskkondadesse
- osa komponente paigaldatakse RIA taristusse (RIHA kesksüsteem)
- seetõttu komponendid peavad vastama erinevate haldajate taristute poolt seatavatele nõuetele
- kuna need nõuded ei ole ette teada, on eelistatud laialt levinud tehnoloogiad ja standardsed lahendused.

## 3 Tehnoloogilised nõuded ja valikud

### Andmesalvestus
- RIHAs ei moodustata tsentraalset superandmebaasi, vaid andmed paigutatakse hajusalt
- [JSON](http://www.json.org/), RIHA andmete põhivorming. Andmete iseloomu arvestades ei ole fookuses relatsiooniline andmetehnoloogia, vaid suuremat paindlikkust võimaldav JSON
- Seejuures võib relatsioonilist andmebaasi kasutada JSON-tekstide hoidmiseks
- Kontseptsioonis on hajusalt tekkivate ja uuenevate andmete kokkukogumine (ingl _harvesting_) kesksesse andmehoidlasse (seda võib nimetada andmelaoks). 

### Masinliidesed (API-d)
- [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer). Masinliidesed teostatakse reeglina REST liidestena
- [X-tee](https://www.ria.ee/ee/x-tee.html) on kasutusel liidestes, kus turvakaalutlused, REST API-de puudumine või muud põhjused ei võimalda REST liideseid kasutada
- [Open API Initiative (Swagger)](https://www.openapis.org/) abil spetsifitseeritakse kõik API-d.

### Autentimine

#### Inimkasutajate autentimine
- teostatakse RIHA-välise teenuse või lahendusena
- kavas on kasutada 2017. a kevadel valmiva eesti.ee eIDAS-autentimisteenust
  - võimalikud on ka täiendavad variandid

#### Masinkasutajate autentimine
- [HTTP Basic Authentication](https://tools.ietf.org/html/rfc2617)

### Sessioonihaldus
- [JSON Web Token](https://jwt.io/), vt [Sessioonihaldus](Sessioonihaldus)

### Pääsuhaldus
 - [LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)
 - [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control) (rollipõhine pääsuhaldus)

### Kasutajaliides (UI)
- [HTML5](https://www.w3.org/TR/html5/)
- [Javascript](http://www.2ality.com/)
- [CSS](https://www.w3.org/Style/CSS/Overview.en.html)
- [React](https://facebook.github.io/react), kasutatud senitehtus
- [Bootstrap 4](https://v4-alpha.getbootstrap.com/) (agiilselt arendatud komponentides)
- millist veebiraamistikku kasutada, ei ole põhiküsimus. Veebiraamistikud arenevad kiiresti. Tähtis on teha komponentide kasutajaliidesed lihtsad ja äriloogika selge, nii, et vajadusel (tehnoloogia vananemine) saaks komponendi ümber kirjutada.
- [Sass](http://sass-lang.com/)

### Äriloogika
- järgides tänapäeva trende, teostatakse kasutajaliidesed peamiselt üleherakendustena (ingl _single page application_)
- paindlikkuse tagamiseks kasutatakse mallipõhisust jms tehnikaid.
  
### Serveripoolsed komponendid
- [Java](https://www.java.com/en/), on eelistatud ja seni kasutatud
- [Go](https://golang.org/) ei ole välistatud
- [Node.js](https://nodejs.org/en/) jms ei ole välistatud.

### Dokumentatsioon

#### Põhilised
- [Markdown](https://guides.github.com/features/mastering-markdown/) (GitHubi dialekt) (avaldamisvorming)
- dokumentatsiooni valmistamisel võib kasutada ka muid vahendeid (lepingupartneri soovil ka Microsoft Word), kuid tulemus teisendatakse alati avaldamisvormingusse

#### Abistavad
- [Jekyll](https://jekyllrb.com/docs/home/) (avaldamistehnoloogia kõrgemate kasutatavusnõuetega dokumentatsioonile), kasutatav  GitHubis või eraldi
- [Liquid](http://shopify.github.io/liquid/) templiidikeel, kasutusel Jekyllis
- [Kramdown](https://kramdown.gettalong.org/quickref.html) - Markdown -> HTML teisendaja, kasutusel GitHub Jekyllis

### Andmebaas
- [PostgreSQL](https://www.postgresql.org/)
  - sh JSON-i töötlemise võimalused
  
### Tarkvararepositoorium
- [GitHub](https://github.com/), jooksvaks arenduseks
  - organisatsioon [e-gov](https://github.com/e-gov) (sisaldab ka muid reposid)
- [BitBucket](https://bitbucket.org/), tulemuste fikseerimiseks
- töövoog on commit-ne GitHub-i, sealt automatiseeritud peegeldamine RIA taristu BitBucket-sse.

### Modelleerimine
- [Enterprise Architect](http://www.sparxsystems.com/products/ea/) 
- [asciiFlow](http://asciiflow.com/), abistavas rollis, lihtsamates joonistes.

### Logimine
- [SLFJ4](http://www.slf4j.org/), Simple Logging Facade for Java
- syslog protokoll (RFC 5424, RFC 3164). 

### Mallid ja konfiguratsioonifailid
- [JSON](http://www.json.org/)
- [YAML](http://yaml.org/).
 
### Üldiselt kasutavav
- [UTF-8](https://en.wikipedia.org/wiki/UTF-8)
- [Ubuntu](https://www.ubuntu.com/) - paigalduskeskkonna op-süsteem.
  
### Visualiseerimine
- tasuta raamistike või teekidega, mis tarbivad RIHA API-sid
- nt [Google Charts](https://developers.google.com/chart/) teegiga on tehtud eksperimentaalne visualiseerimismoodul.

### Sõnastikud
- [Schema.org](https://github.com/schemaorg/schemaorg) on olnud kaalumisel, kuid sobivus RIHA vajadusteks pole selge
- [JSON-LD](http://json-ld.org/), "a lightweight format for linked data", on olnud kaalumisel; sobivus RIHA vajaduste ja ressurssidega on ebaselge.
