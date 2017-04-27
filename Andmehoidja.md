---
title: Andmehoidja
permalink: Andmehoidja
---

# Andmehoidja
{:.no_toc}

* TOC
{:toc}

Vt ka: [Kesksüsteem](Kesk).

## Ülevaade

Andmehoidja (RIHA-Storage) on serveriteenus (e RIHA _backend_ komponent), mis korraldab andmete püsihoidmist. Andmehoidla teenindab oma API kaudu RIHA teisi serveriteenuseid, olles vahendajaks PostgreSQL andmebaasi ja HTTPS päringute vahel.

```
                             Serveriteenused

+------------+ +------------+ +------------+ +------------+ +------------+
|            | |            | |            | |            | |            |
| KIRJELDAJA | |   KOGUJA   | |   HINDAJA  | |  SIRVIJA   | | TEAVITAJA  |
|            | |            | |            | |            | |            |
+-----+------+ +----+-------+ +------+-----+ +-------+----+ +------+-----+
      ^             ^                ^               ^             ^
      |             |    HTTPS       |               |             |
      |             |                v               |             |
      |             |         +------+-----+         |             |
      |             +-------> +            | <-------+             |
      +---------------------> + ANDMEHOIDJA| <---------------------+
                              |            |
                              +---+-----+--+
                                  |     ^
                                  |     |
                                  v     |

                         Andmehoidla (PostgreSQL)


```
Andmehoidja kood asub [RIHA-Storage](https://github.com/e-gov/RIHA-Storage).

<p class='rem'>Vajadusel vt vanem versioon "RIHA andmebaasi kontseptuaalne mudel" v2.0, 29.04.2016 RIA dokumentatsioonikeskkonnas</p>

## Põhimõtted

Andmebaasimootor. Kasutatakse uusimat Ubuntu LTS-s toetatud PostgreSQL versiooni. Alates versioonist 9.4 on PostgreSQL-l väga hea tugi vaba struktuuriga JSON andmete efektiivseks hoidmiseks ja töötlemiseks (`jsonb` tüüpi väljal).

Senised erinevad tabelid infosüsteemi, teenuse, klassifikaatori, valdkonna, valdkonna sõnastiku ja XML vara jaoks asendatakse ühise universaalse tabeliga `main_resource`.

Kõik uute tabelite väljad esitatakse PostgreSQL `jsonb` tüüpi nimi-väärtus paaridest koosneva objektina, kus väärtused võivad olla nii lihtväärtused kui omakorda sisemise struktuuriga objektid või massiivid. Selline mahukas `jsonb` objekt on klassikalise SQL tabeli mõttes väljal `json_content`.

Igal uuel tabelil on lisaks `json_content` väljale veel mitmeid klassikalisi SQL välju, mille põhieesmärk on võimaldada kiiremat otsingut, ehk niiöelda puhverdatud väärtused.

Väliselt kirjeldatavad ressursid (infosüsteemid, teenused, tabelid jne) identifitseeritakse stabiilse, versioonist sõltumatu tekstilise URI väljaga, mida mh kasutatakse selle ressursi kirjeldamisel väliste kirjeldusfailide abil.

Primaarvõti `id` identifitseerib ressursi konkreetset versiooni.

Ressursi spetsiifilised abitabelid, mis on ette nähtud loendite jms lihtsate struktuuride kodeerimiseks (näiteks, eri keeltes olevad nimed jms) kaotatakse ja asendatakse kas JSON massiivi või JSON nimi-väärtus objektiga otse põhitabeli vastaval `jsonb`-väljal.

Kommentaaride ja dokumentide universaalsed abitabelid jäetakse alles ja restruktureeritakse uute põhimõtete järgi.

## Tabelid

Andmebaasi struktuur toetub kahele peamisele tabelile: ülemise taseme tabel `main_resource` ja tema komponentide tabel `data_object` ning neile lisainfot andvatele tabelitele `document` ja `comment`.

Lisaks on kasutusel objektitüüpide tabel `kind`.

<p status='rem'>Tabeli "role_right" vajadus tuleb üle vaadata.</p>

````
                +---------------+
                |               |
                |    kind       |
                |               |
                +-------+-------+
                        |
                        |
                +-------+-------+
                |               |
                | main_resource |
       +--------+               +---------+
       |        +---------+-----+         |
       |                  |               |
       |                  |               |
+------+------+ +---------+-----+  +------+-----+
|             | |               |  |            |
|data_object  | |    document   |  |   comment  |
|             | |               |  |            |
+-------------+ +---------------+  +------------+

````

Mõlemad tabelid võivad moodustada hierarhia oma `parent_id` välja kaudu. Seejuures näeme ette, et:

- `main_resource` tabeli rida sisaldab suurel hulgal informatsiooni, on versioneeritav ja ei moodusta üldjuhul sügavaid hierarhiaid. Tabel ei sisalda nö väga suurtes kogustes ridu.
- `data_object` tabeli rida on seotud konkreetse `main_resource` tabeli reaga, sisaldab vähem informatsiooni, ei ole omaette versioneeritav ja võib moodustada sügavaid hierarhiaid. Tegemist on kogu andmebaasi kõige mahukama tabeliga.

Peamised täiendavat informatsiooni sisaldavad tabelid, mis võivad viidata mistahes `main_resource` või `data_object` reale, on:

`document` - sisaldab üldjuhul ametliku dokumendi (määruse vms) tervet sisu või tema viita kas URLi või failisüsteemi viidana, samuti selle metainformatsiooni: nimi, kehtivusperiood jms.
- `comment` sisaldab mõne süsteemi kasutaja tekstilist kommentaari või küsimust konkreetse süsteemi või tema osa kohta, koos metainfoga: millal lisati jms.

Uue andmebaasi struktuuri füüsilise andmemudeli põhiosad

Esitame siin füüsilise andmemudeli põhiosad, jättes täpsustamata igas tabelis oleva json_content välja sisemise struktuuri, kus hoitakse tabeli ridade põhi-infot.

json_content välja struktuur tuuakse välja eraldi dokumendis „RIHA+ füüsiline mudel".

## Infosüsteemi kirjeldamine

Infosüsteem esitatakse `main_resource` tabelis. Välja `kind` väärtus on `infosystem`. Kirjelduse põhiosad esitakse `json_content` väljal, mis sisaldab mh ka kõiki ülaltoodud SQL-lauses antud klassikalisi välju.

Infosüsteemi püsiv, versioonist sõltumatu identifikaator on `uri`, kuhu sisestatakse üldjuhul `riha:infosystem:lühinimi` . Hierarhilise põhiobjekti korral viidatakse ülemobjektile `parent_uri` välja kaudu, millele lisandub sekundaarse võimalusena versioonitundlik `parent_id` juhuks, kui hierarhia versioneerimise käigus muutub.

Infosüsteemi, tema andmebaaside ja tabelite versioneerimise põhimõtted on detailselt kirjas selle dokumendi hilisemas peatükis „versioneerimine".

## Infosüsteemi funktsioonide kirjeldamine

<p class='staatus'>Funktsioonide kirjeldamise ärivajadus on praeguse seisuga mitteaktuaalne. Seetõttu käesolev jaotis on 'deprecated'.</p>

Uue võimalusena on võimalik infosüsteemi kirjelduse koosseisus kirjeldada infosüsteemi funktsioonide/eesmärkide loetelu. Funktsioonid/eesmärgid tuleb kirjeldada andmete säilitustähtaegade täpsusega - erineva säilitustähtajaga säilitatavate andmete kohta tuleb kirjeldada erinev funktsioon/eesmärk.

Funktsioonid/eesmärgid esitatakse `data_object` tabelis hierarhiliselt, kasutades `parent_id` välja. Tasemete arv pole piiratud. Olemi tüüp on kirjas tekstiväljal `kind` ning see on alati väärtusega `function`. Iga selline olem on `main_resource_id` kaudu alati ühe infosüsteemi konkreetse versiooniga seotud.

Kirjelduse põhiosad esitakse `json_content` väljal.

## Infosüsteemi andmekoosseisu kirjeldamine

Infosüsteemi andmekoosseis esitatakse `data_object` tabelis hierarhiliselt, kasutades `parent_id` välja. Tasemete arv pole piiratud. Olemi tüüp on kirjas tekstiväljal `kind` ning see on alati väärtusega `entity`. Iga selline olem on `main_resource_id` kaudu alati ühe infosüsteemi konkreetse versiooniga seotud.

Kirjelduse põhiosad esitakse `json_content` väljal.

## Andmebaaside, tabelite ja väljade kirjeldamine

Infosüsteemi andmebaasid, tabelid ja väljad esitatakse `data_object` tabelis hierarhiliselt, kasutades `parent_id` välja. Esimene tase on alati andmebaas. Olemi (andmebaas, tabel, ...) tüübi määrab tekstiväli `kind`. Iga selline olem on `main_resource_id` kaudu alati ühe infosüsteemi konkreetse versiooniga seotud.

Olemi püsiv, versioonist sõltumatu identifikaator on `uri`, kuhu sisestatakse üldjuhul `riha:infosystem:lühinimi:andmebaas:tabel:väli` ehk koolonitega eraldatud hierarhia infosüsteemi, andmebaasi, tabeli jne väljast kuni antud olemi nimeni. Olemi hierarhia võib olla ka sügavam, näiteks, kui kasutatakse mitte-SQL-andmebaase või sisemise struktuuriga JSON välju.

Kirjelduse põhiosad esitakse `json_content` väljal, mis sisaldab mh ka kõiki ülaltoodud SQL-lauses antud klassikalisi välju.

## Teenuste kirjeldamine

Teenused esitatakse `main_resource` tabelis, ning nad on üldjuhul konkreetse infosüsteemiga `parent_id` kaudu seotud. Teenustel võivad olla versioonid, sõltumatult infosüsteemist kui terviku versioonidest.

Teenuse WSDL jms info esitatakse `json_content` väljal, mis sisaldab mh ka kõiki ülaltoodud SQL-lauses antud klassikalisi välju.

Teenuse sisendid ja väljundid kirjeldatakse lisaks WSDL-s toodud struktuurile ka nende klassifikaatorite, sõnastike ja muu semantika jaoks eraldi igaüks ühe `data_object` reaga, mille `kind` on `input` või `output` ja mille `main_resource_id` on teenuse id ja mille `uri` on kujul `riha:service:teenusenimi:_input:_sisendinimi`.

Sisendite ja väljundite järjekord esitatud ei ole, konkreetne struktuur on üldjuhul loetav WSDL failist.

## Valdkondade, sõnastike ja XML varade kirjeldamine

<p class='staatus'>Käesolev jaotis on ettevaatav. Ei teostata 2017 arenduses.</p>

Nende kolme põhiressursi edasine konkreetne kasutus- ja haldamisviis vajab detailset ärianalüüsi. Põhiküsimused seejuures on:

-  millises kirjelduskeeles jätkata valdkonnasõnastike esitamist: kas minna OWL-lt üle lihtsamale RDFS-le või mõnele muule kirjelduskeelele?
- kas asuda valdkonnasõnastike juures baashierarhiana kasutama olemasolevat http://schema.org hierarhiat ja kui, siis mis moel integreerida sinna eesti hierarhiad?
- kes ja kuidas valdkonnasõnastikke haldab: administratiivselt ja tehniliselt?
- kas ja kuidas nö lahti-struktureerida klassifikaatorite senised Excel-failid konkreetseteks, töödeldavateks väärtuseloenditeks?
- kuidas esitada klassifikaatoreid lahtistruktureeritud kujul?
- kes ja kuidas klassifikaatoreid haldab: administratiivselt ja tehniliselt?
- kas edaspidi on vaja XML-varade haldamist, või võib selle põhiressursi aktiivsest RIHA-st eemaldada?

Seetõttu ei esita me praeguses etapis veel põhimõtteliselt uut viisi nende olemite kodeerimiseks. Küll aga esitame nende kodeerimise nö ajutisel, lihtsustatud moel `main_resource` tabelisse, mis toob samas kaasa minimaalse hulga muutusi andmebaasi struktuuris.

Valdkonnasõnastike ja klassifikaatorite praktiline kasutamine andmetabelite, väljade ja teenuste kirjeldamisel edaspidi valitud kodeerimisviisist ei sõltu: praktiline kasutamine tähendab alati tekstiliste tagide lisamist kirjeldustele, mis ei sõltu sõnastiku või klassifikaatori esitamisest RIHA andmebaasi struktuuris. Tekstiliste tagide kasutamise põhimõtted on esitatud eraldi dokumendis, mis kirjeldab RIHA objektide välist esitust JSON-kujul.

Ajutine, lihtsustatud kirjeldus uue RIHA andmebaasis:

- valdkonnasõnastikud, teenused ja XML varad lisatakse `main_resource` tabelisse, tekitades igaühele uue `uri` kujul a la `RIHA:classifier:classifierid`, kus `classifierid` on senise klassifikaatori id senises RIHAs.
- kogu senine põhiobjekti kirjeldus lisatakse `json_content` väljale JSON nimi-väärtus paaride ojektina täpselt sel viisil ja selliste väljanimedega, nagu ta on olemas senises RIHAs, pluss `main_resource` tabelist tulenevad väljad. Kui mõni `main_resource` väljanimi kattub senise väljanimega, siis nimetatakse senine ümber prefiksiga `legacy_...`.
- põhiobjekti kirjelduses olevad senised väljasisud jäävad muutmata.
- põhiobjekti abitabelid ja nende sisu jäävad muus osas muutmata, kui nad aga sisaldavad välisvõtit senisele põhiobjektile viitamiseks, siis lisatakse neile täiendavalt väli `main_resource_uri` , mis täidetakse vastava senise põhiobjekti uue `uri` välja sisuga kujul a la `RIHA:classifier:classifierid`, kus `classifierid` on senise põhiobjekti id senises RIHAs. See võimaldab kasutada senist objekti versioneeritud kujul.

## Versioneerimine

<p class='staatus'>Versioneerimise käsitlus vajab täpsustamist</p>

Kirjeldame järgnevas infosüsteemi versioneerimist, kuid samad põhimõtted kehtivad ka teiste ressursside (teenused, ..., XML varad) kohta.

Infosüsteemi kirjeldusi versioneeritakse eeskätt selleks, et võimaldada infosüsteemi kirjeldust jooksvalt uuendada, tuvastades seejuures tekkinud erinevused viimasest kooskõlastatud seisus olevast kirjeldusest.

### Versioneerimise stsenaariumid

Oletame, et kuupäeval `X` saab infosüsteem kooskõlastuse, olles näiteks versiooniga `V`. Pärast seda infosüsteem areneb ja tema kirjeldust uuendatakse. Uuendamine võib olla küllalt sage, kui seda tehakse automaatselt API kaudu.

Iga kirjelduse uuendus ei too üldjuhul kaasa olukorda, et kooskõlastus enam ei kehti. Samas peab olema võimalik suhteliselt lihtsalt tuvastada, milliseid muutusi on infosüsteemi kirjelduses toimunud peale kooskõlastamiskuupäeva `X`.

Kui muutuste hulk osutub aja jooksul väga suureks, lisanduvad näiteks uued isikuandmete ja aadressandmete tabelid, siis võib kas infosüsteemi eest vastutaja ise või mõni kooskõlastaja leida, et süsteem vajab uuesti kooskõlastamist. Seejuures võib olla mõistlik realiseerida automaatsed kontrollreeglid, mis osutavad, et infosüsteemi muutused on sedavõrd suured/olulised, et tasuks kaaluda uue versiooni loomist ja uut kooskõlastamist.

Uue kooskõlastamise vajaduse tuvastamise puhul oleks mõistlik hakata infosüsteemi uuesti kooskõlastama. Seejuures on abiks, kui saab endiselt kergesti näha erinevusi viimase kooskõlastatud versiooni ja hetkeseisu vahel.

Kui kooskõlastamine võtab pikemalt aega, siis infosüsteemi kirjeldused muutuvad tõenäoliselt ka uue kooskõlastamise käigus, näiteks parandatakse ja täiustatakse infosüsteemi ja tema kirjeldusi vastavalt kooskõlastaja nõuetele. Seega tasub uus versioon `V+1` üldjuhul luua alles siis, kui kooskõlastamine on edukalt lõppenud, mis tähendab, et kooskõlastatakse jooksvalt arenevat viimast hetkeseisu.

Samuti on võimalik stsenaarium, kus infosüsteemi eest vastutaja ise leiab, et on toimunud piisavalt suured muutused selleks, et luua kohe uus „vaheversioon" `V+1`, mis salvestaks hetkeseisu. Kooskõlastust hakatakse siis ikkagi küsima selle vaheversiooni jooksvalt arenevale kirjeldusele, ning kooskõlastuse saamise järel fikseeritakse seis järgmise versiooniga `V+2`.

Kokkuvõttes, kriitiline versioneerimise eesmärk on kergesti leida, mis on erinevused viimase kooskõlastatud kirjelduse ja hetkekirjelduse vahel.

Lisaeesmärk on see, et vaheseisu saab infosüsteemi eest vastutaja soovi korral säilitada, luues ise uue versiooni. Kõigi vaheseisude jooksev säilitamine ei ole eesmärk. Automaatselt uusi versioone ei tekitata. Kui hiljutine kirjelduse muutus uuesti muudetakse, siis vaheseis kirjutatakse üle ja ei ole taastatav, tingimusel, et pole spetsiaalselt loodud vaheversiooni.

### Versioneerimise tehnoloogia

Versioneeritavad põhitabelid infosüsteemi kirjelduses on:

- `main_resource` sisaldab infosüsteemi kirjelduse üldisel kõrgtasemel: üks rida on üks infosüsteem
- `data_object` sisaldab erinevate detail-andmeobjektide (andmebaasid, tabelid, tulbad, teenuste sisendid-väljundid) kirjeldusi: üks rida on üks andmeobjekt. Iga andmeobjekt viitab infosüsteemi id-le.

Kummagi tabeli juures tuleb ette näha kahte võimalust muutusteks:

- inimkasutaja uuendab mõnda välja
- API kaudu laetakse terve infosüsteemi kirjeldus automaatselt uuesti, seejuures võib mõni väli muutuda/mõni tabel lisanduda, kuid kogu sisu võib ka muutumatuks jääda

Automaatse kirjelduse laadimise juures tuleb ette näha infosüsteemi unikaalne nimi , mille API ette annab. Meie soovituseks on kasutada URI, milleks võib kasutada ka infosüsteemi URLi, kuid ei pruugi seda kasutada.

`main_resource` tabeli versioneerimiseks kasutame järgmisi põhimõtteid:

- infosüsteemi identifikaatoriks on tabeli `id` väli, lisaidentifikaatoriks on tabeli `uri` väli.
    Infosüsteemi esialgselt loodud `id` püsib alati edaspidigi, loodavad versioonid on nö mitte-muudetavad vahekoopiad, mis saavad uue `id`.
- sama `uri`-ga ja sama `id`-ga (sisuliselt samal) infosüsteemil võib infosüsteemide tabelis olla hulk ridu: ajaliselt esimesena loodud rida on alati aktiivne hetkeseis, kõik uuemad read on mitteaktiivsed vaheversioonid
- vaheversioonide ridu ei muudeta
- infosüsteemi mõne välja muutumise korral (inimese või API poolt) muudetakse lihtsalt ära aktiivse hetkeseisu mõni väli; muutuste ahelad lähevad seejuures kaduma, kui spetsiaalselt ei looda vaheversiooni.
- infosüsteemi real on alati kehtivuse alguse ajatempli ja lõppemise ajatempli väli
- infosüsteemi uue versiooni tekitab ainult inimkasutaja (kas siis kooskõlastaja või infosüsteemi eest vastutaja), mille käigus:
  - luuakse kehtivast reast koopia, mis saab uue `id` , kuid säilib senine `uri` väli: uus koopia vastab viimasele kehtinud versioonile, ning ei kuulu muutumisele, senine `id` vastab endiselt hetkeseisule.
  - muudetakse uue loodud koopia-rea kehtivuse lõpukuupäev hetkekuupäevaks.
  - hetkeseisu rea kehtivuse alguskuupäev muudetakse hetkekuupäevaks
  - luuakse koopiad (arvestades optimeeringuid, mis kirjas järgnevas) kõigist antud infosüsteemi senikehtinud reale viitavatest `data_object` tabeli ridadest: iga koopia saab uue `id` ja hakkab viitama infosüsteemi äsjaloodud vaheversiooni `id`-le, samuti muudetakse uueks vanem-andmeobjekt-`id` viidad (andmeobjektid moodustavad hierarhilise süsteemi).

Kokkuvõttes paneme tähele, et see infosüsteemi rida, mis on loodud esimesena ja millel on kõige väiksem `id`, on alati hetkel kehtiv ja muudetav seis. Uuemad read suuremate `id`-dega vastavad vaheversioonidele, mis salvestavad mingi hetkeseisu. Versioonide kehtivusajad on määratud kehtivusaja alguse ja lõpu timestamp väljadega.

Andmeobjekte sisaldava `data_object` tabeli uuendamisel kasutame järgmisi põhimõtteid:

- tabeli ridadest luuakse uued versioonid juhul, kui luuakse uus infosüsteemi vaheversioon (vt ülal) kuid mitte juhul, kui lihtsalt uuendatakse andmeobjektide kirjeldust või struktuuri: viimasel juhul lihtsalt muudetakse viimast hetkeseisu.
- andmeobjekti struktuuri uuendamisel API kaudu lähtutakse põhimõttest, et objekti identifitseerib temani viivate nimede ahel (üldjuhul infosüsteemi `uri` -> andmebaasi nimi -> tabeli nimi -> välja nimi) . Seda nimede ahelat käsitletakse kui andmeobjekti URI ja hoitakse andmeobjekti tabeli vastaval `uri` väljal.
- andmeobjekti väljal on alati kehtivuse alguse ajatempli ja lõpu ajatempli väli.
- infosüsteemist (`main_resource` tabel) uue versiooni loomisel kasutatakse järgmist optimeeringut ridade arvu kokkuhoiuks: Olgu äsjaloodud infosüsteemi eelmine versioon `V`, hetkel loodav vaheversioon `V+1` ja uus hetkeversioon `V+2`. Vaheversiooni `V+1` jaoks tuleks luua koopiad kõigist hetkeversiooni ridadest. Kui aga eksisteerib eelmine versioon `V` ja `V-l` on olemas hetkeseisuga identse sisuga (v.a. id-d ja timestampid) andmeobjekti rida `R`, siis uuendatakse `R`-i kehtivuse lõpu timestampi . See optimeering tekitab küll olukorra, kus andmeobjekti rida ei pruugi viidata temale vastavale infosüsteemi versioonile, vaid varasemale, ning kehtivus tuleb tuvastada timestampide vahemiku kaudu.

Analoogiliselt `main_resource` tabelile paneme tähele, et see andmeobjekti (`data_object`) tabeli rida, mis on loodud esimesena ja millel on kõige väiksem `id`, on alati hetkel kehtiv ja muudetav seis. Uuemad read suuremate `id`-dga vastavad vaheversioonidele, mis salvestavad mingi fikseeritud ja mittemuudetava seisu. Versioonide kehtivusajad on määratud kehtivusaja alguse ja lõpu ajatempli väljadega. Kõikidele infosüsteemi (`main_resource` tabel) vaheversioonidele vastavaid andmeobjekti-välju ei pruugi seejuures füüsiliselt eksisteerida: pikkade ahelate puhul taaskasutatakse varasemaid andmeobjekti-ridu, nihutades nende kehtivuse lõpu ajatempleid vastavalt edasi.
