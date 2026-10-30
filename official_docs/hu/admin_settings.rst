:author: Miklós Bán
:date: 2026-08-08

Adminisztrációs beállítások
***************************

A projektadminisztrációs felület eszközöket biztosít egy OpenBioMaps-projekt
konfigurálásához, felhasználóinak és adatstruktúráinak kezeléséhez, valamint
a projekthez kapcsolódó szolgáltatások felügyeletéhez. Az adminisztrátor
számára elérhető oldalak az adminisztrátor jogosultságaitól, a projekt
konfigurációjától, a telepített moduloktól és a szerverkörnyezettől függnek.

Ez az oldal áttekintést nyújt az adminisztrációs beállításokról és
eszközökről. Egyes beállítások befolyásolják a projektadatokhoz való
hozzáférést, vagy módosítják az alapul szolgáló adatbázist. Az
adminisztrátoroknak ezért körültekintően át kell tekinteniük a
változtatásokat, és tesztelniük kell azokat, mielőtt éles projektben
alkalmaznák őket.

A projektadminisztrációs dokumentáció áttekintését lásd:
:doc:`Projektadminisztráció <../admin_pages>`.


.. _administrative-access:

Adminisztrációs hozzáférés
==========================

Az **Adminisztrációs hozzáférés** szakasz lehetővé teszi a
projektadminisztrátorok számára, hogy az egyes adminisztrációs funkciókat
felhasználói csoportokra ruházzák át. A projektadminisztrációs felületen
elérhető minden funkció egy vagy több csoporthoz rendelhető.

Ez részletes szabályozást biztosít afelett, hogy ki végezhet
adminisztrációs feladatokat. Egy projekt például a következő csoportokat
határozhatja meg:

* **Felhasználókezelők**, akik hozzáférnek a felhasználók és csoportok
  kezeléséhez;
* **Adatkurátorok**, akik hozzáférnek a fajnevekhez, a csatolmányokhoz és
  az adatkezelési eszközökhöz; valamint
* **Feltöltésiűrlap-szerkesztők**, akik hozzáférnek a feltöltési űrlapok
  kezeléséhez.

Minden adminisztrációs szerepkörnek csak a szükséges jogosultságokat adja
meg. Az adatbázis-struktúrákat módosító, SQL-t végrehajtó, hozzáférési
szabályokat kezelő vagy végrehajtható kódot szerkesztő funkciókat csak
megbízható adminisztrátorok számára szabad engedélyezni.

.. TODO: Dokumentálni kell minden hozzárendelhető adminisztrációs funkciót
   és az általuk biztosított jogosultságokat. Azt is tisztázni kell, hogy a
   beágyazott csoportokon keresztül örökölt jogosultságok rekurzívan
   kerülnek-e kiértékelésre, és hogy a felhasználónak újra be kell-e
   jelentkeznie az adminisztrációs jogosultságai megváltozása után.


.. _admin-sidebar-pins:

Adminisztrációs oldalak kitűzése az oldalsávra
==============================================

Az adminisztrációs oldal (a gépház) minden kártyáján van egy kitűzés gomb. Egy oldal kitűzése felveszi azt a profil oldalsávjába, ugyanazon kategóriák alá, amelyekben az adminisztrációs oldalon megjelenik. A több kategóriába tartozó oldalnak egyetlen kitűzési állapota van, és minden kategóriában megjelenik. A kitűzött oldal nélküli kategóriák nem jelennek meg. Az adminisztrációs oldalon a kártya továbbra is elérhető marad.

A kitűzések személyesek: minden adminisztrátornak saját listája van, amelyet a projektbeállításokkal együtt mentünk. A kitűzés soha nem ad hozzáférést az oldalhoz, és azok az oldalak, amelyeket a felhasználó nem nyithat meg, akkor is rejtve maradnak, ha korábban kitűzték őket. Ha ugyanaz a felhasználó két munkamenetben egyszerre módosítja a kitűzéseket, az utolsó mentés érvényesül.

.. _database-columns:

Adatbázistáblák és -oszlopok
============================

Az **Adatbázistáblák és -oszlopok** szakasz a projekthez kapcsolódó SQL
táblák, nézetek és oszlopok létrehozására és kezelésére szolgál. Az ezen a
felületen regisztrált objektumok bekerülnek az OpenBioMaps metaadatai közé,
így elérhetővé tehetők a feltöltési űrlapok, lekérdezések, modulok és más
OpenBioMaps-felületek számára.

A szabványos SQL-klienssel közvetlenül létrehozott táblák és oszlopok
regisztrációja nem történik meg automatikusan. Ezeket is hozzá kell adni a
megfelelő OpenBioMaps-metaadatokhoz, mielőtt a webalkalmazáson keresztül
használhatók lennének.

.. TODO: Ismertetni kell, hogyan regisztrálható egy meglévő SQL-tábla vagy
   -nézet annak újbóli létrehozása nélkül. Dokumentálni kell, hogy a felület
   mely metaadattáblákat módosítja, és hogy a felületen kívül létrehozott
   adatbázis-objektumok biztonságosan importálhatók-e.


Táblák és oszlopok elnevezése
-----------------------------

A tábla- és oszlopnevekhez kisbetűket, számokat és aláhúzásjeleket
használjon. Kerülje a szóközöket, az ékezetes karaktereket, az idézőjeles
azonosítókat és más különleges karaktereket. A nevek legyenek leíró
jellegűek, és maradjanak változatlanok, miután űrlapok, lekérdezések vagy
modulok használni kezdik őket.

Tábla vagy oszlop létrehozásakor mindig ajánlott leírást megadni. Ezek a
leírások a projekt metaadatainak részét képezik, és segítenek a
felhasználóknak megérteni az adatok jelentését és tervezett felhasználását.

.. TODO: Dokumentálni kell a felület által kikényszerített összes
   elnevezési szabályt, beleértve a maximális hosszúságot, a fenntartott
   neveket, a sémák kezelését, valamint azt, hogy kezdődhet-e egy név
   számmal.


Az elérhető oszlopok regisztrálása
----------------------------------

Az adminisztrátorok kiválaszthatják, hogy mely oszlopok legyenek elérhetők
feltöltési űrlapok és lekérdezési felületek létrehozásakor. A PostgreSQL
rendszerben létező, de elérhetőként nem regisztrált oszlop nem jelenik meg
automatikusan ezeken a felületeken.

Az oszlopokhoz szemantikai szerepek is rendelhetők. Ezek a szerepek
lehetővé teszik az OpenBioMaps és moduljai számára, hogy a projektspecifikus
oszlopnevektől függetlenül azonosítsák a fontos mezőket. A projekttől és a
telepített moduloktól függően a szerepek többek között a következő
tartalmú mezőket azonosíthatják:

* tudományos név;
* alternatív taxonnév;
* megfigyelési dátum;
* adatgyűjtő;
* helyszín vagy geometria;
* egyedek száma;
* szélességi és hosszúsági értékek;
* hivatkozás; vagy
* csatolmány.

.. TODO: Teljes listát kell készíteni a szemantikai szerepekről, és meg
   kell határozni, hogy mely alapfunkciók vagy modulok használják az egyes
   szerepeket. Tisztázni kell, hogy több oszlop rendelkezhet-e ugyanazzal a
   szereppel, illetve egy oszlophoz több szerep is rendelhető-e.


Oszloptípusok
-------------

Az adminisztrációs felület a következő dokumentált oszloptípusokat vagy
szemantikai szerepeket biztosítja:

``Data``
   Általános célú adatoszlop.

``Spatial Geometry``
   Térképekhez és térbeli műveletekhez használt geometriaoszlop.

``Scientific Species Name``
   A taxonkezelési funkciók által használt tudományosnév-oszlop.

``Alternative Names``
   A taxonkezelési funkciók által használt alternatívnév-oszlop.

``Date``
   A dátumszűrők által használt dátum- vagy dátum-idő oszlop.

``Number of Individuals``
   Az összesítő funkciók által használt numerikus oszlop.

``Latitude/Longitude``
   Térbeli geometria létrehozásához használt koordinátaoszlop.

``Citing``
   Az összesítő funkciók által használt, hivatkozással kapcsolatos oszlop.

``Attachment``
   Feltöltött fájlcsatolmányokra hivatkozó oszlop.

``UTM Zone``
   Koordinátákból létrehozott térbeli geometria esetén használt
   UTM-zóna-oszlop.

.. TODO: Meg kell erősíteni, hogy ezek a nevek megfelelnek az
   adminisztrációs felület jelenlegi címkéinek. Ismertetni kell, hogyan
   kapcsolódnak ezek a szemantikai típusok a PostgreSQL adattípusaihoz, és
   dokumentálni kell az egyes beállításokhoz szükséges PostgreSQL-típust.

.. TODO: Tisztázni kell, hogyan történik a szélességi és hosszúsági
   oszlopok párosítása, valamint hogyan határozza meg a rendszer az UTM
   zónát, a koordináta-referenciarendszert és a féltekét a geometria
   létrehozása során.


Oszlopleírások és parancsok
---------------------------

A **Comment** mező az oszlop tartalmának leírását tartalmazza. Érdemes
jelentést hordozó leírást megadni, mert ez hozzájárul a projekt
metaadataihoz.

A **Command** mező meghatározott műveletek végrehajtására vagy beállítások
oszlophoz rendelésére használható. A dokumentált parancsok a következők:

``SET srid:4326``
   Az ``obm_geometry`` oszlophoz rendeli a 4326-os SRID-t. A ``4326``
   helyére a projekt által megkövetelt térbeli referencia-azonosítót kell
   írni.

``SET use_rules:1``
   Engedélyezi a hozzáférési szabályok kezelését az ``obm_id`` oszlophoz.

``RENAME:new_name``
   Átnevezi az oszlopot ``new_name`` névre.

``DROP``
   Törli az oszlopot.

Egy oszlop átnevezése vagy törlése érvénytelenné teheti az arra hivatkozó
feltöltési űrlapokat, lekérdezési sablonokat, modulokat, nézeteket,
triggereket és külső alkalmazásokat. Bármelyik művelet elvégzése előtt
frissítse az összes függő konfigurációt, és szükség esetén készítsen
biztonsági másolatot az adatbázisról.

.. TODO: Meg kell erősíteni minden parancs pontos szintaxisát,
   kis- és nagybetű-érzékenységét és támogatott célobjektumait. Ismertetni
   kell, hogy a parancsok végrehajtása azonnal megtörténik-e, valamint hogy
   a felület ellenőrzi-e az adatbázis függőségeit egy oszlop átnevezése
   vagy törlése előtt.

.. TODO: Tisztázni kell, hogy a ``SET srid`` csak a metaadatokat
   változtatja-e meg, vagy a meglévő koordinátákat is átalakítja. Ha egy
   SRID úgy változik meg, hogy a koordinátaértékeket nem alakítják át, az
   érvénytelen térbeli adatokat eredményezhet.

.. TODO: Ismertetni kell, mit változtat meg a ``SET use_rules:1``, és hogy
   létrehozza, engedélyezi vagy csak regisztrálja-e a projekt sorszintű
   hozzáférési szabályait.


SQL-konzol
----------

A rendszeradminisztrátorok számára SQL-konzol is elérhető. Az SQL-konzol
a projektadatok és adatbázis-struktúrák módosítására vagy törlésére
használható. Emiatt az adatbázistáblák felületéhez csak olyan megbízható
felhasználóknak szabad hozzáférést adni, akik megfelelő PostgreSQL- és
OpenBioMaps-rendszeradminisztrációs tapasztalattal rendelkeznek.

Az SQL-konzolban végrehajtott lekérdezések elmenthetők és újból
futtathatók.

A konzol dinamikus táblában jeleníti meg a lekérdezési eredményeket. A
lekérdezési tábla eredményei CSV-fájlként exportálhatók. Ha a lekérdezési
eredmény több mint 1 000 sort tartalmaz, a tábla már nem jelenik meg;
helyette automatikusan CSV-export készül.


Nézetek kezelése
----------------

Egy adattábla nézettel helyettesíthető az adatok testreszabott
megjelenítése vagy egy meghatározott munkafolyamat javítása érdekében. A
dokumentált eljárás az eredeti táblával azonos nevű sémát hoz létre, az
eredeti táblát ebbe a sémába helyezi át, majd annak korábbi helyén nézetet
hoz létre. A megfelelő ``INSERT``, ``UPDATE`` és ``DELETE`` szabályok
biztosítják az írási műveleteket, ahol azok konfigurálva vannak.

Ez a megközelítés hasznos lehet költséges munkafolyamatok vagy triggerek
által érintett nagy táblák esetében. Jelentősen megváltoztatja az
adatbázis szerkezetét, és hatással lehet az űrlapokra, lekérdezésekre,
modulokra, idegen kulcsokra, triggerekre, biztonsági mentésekre és külső
kliensekre.

.. TODO: Dokumentálni kell a tábla nézettel történő helyettesítésekor
   végrehajtott pontos átalakítást, beleértve az objektumneveket, a
   tulajdonjogot, a jogosultságokat, a sorozatokat, indexeket,
   megszorításokat, idegen kulcsokat és a létrehozott írási szabályokat.
   Támogatott visszaállítási eljárást is biztosítani kell.

.. TODO: Ismertetni kell, milyen teljesítményproblémák megoldására szolgál
   ez a funkció. Egy tábla nézettel történő helyettesítése önmagában nem
   javítja a teljesítményt, ezért le kell írni a várt nézetdefiníciót és
   optimalizálási stratégiát.


.. _data-access-check:

Adathozzáférés
==============

Az **Adathozzáférés** szakasz összefoglalja a projekt
hozzáférés-konfigurációját és a sorszintű hozzáférési szabályok aktuális
állapotát. Az adminisztrátorok megvizsgálhatják a projektre és kezelt
adattábláira alkalmazott olvasási és módosítási szinteket.

A felület a következőket tartalmazza:

* az adatok olvasásához és módosításához beállított szintek;
* az egyes adattáblák hozzáférési korlátozásainak állapota;
* a konfigurált korlátozások engedélyezésére vagy letiltására szolgáló
  vezérlők;
* a hozzáférési szabályokat karbantartó triggerek állapota; valamint
* kapcsolódó dokumentációra mutató hivatkozások.

A dokumentált hozzáférési szintek a következők:

``everybody``
   A hozzáférés nincs hitelesített felhasználókra korlátozva.

``logged-in users``
   A hozzáférés hitelesítést igényel.

``specified group members``
   A hozzáférést projektcsoportok és részletesebb szabályok szabályozzák.

Egy rekord tényleges hozzáférését projekt-, sor- és oszlopszintű szabályok
is befolyásolhatják. A részletes áttekintést lásd:
:doc:`Adathozzáférés <../data_access>`.

A felület a **Profile > Project administration > Data access** útvonalon
érhető el. Egyes alapértelmezett értékeket a projekt
``local_vars.php.inc`` konfigurációs fájlja is meghatározhat.

.. TODO: Meg kell erősíteni az aktuális navigációs címkéket, és hozzá kell
   rendelni az ``everybody``, ``logged-in users`` és
   ``specified group members`` felületi címkéket a megfelelő konfigurációs
   értékekhez.

.. TODO: Dokumentálni kell, mely változtatások végezhetők el közvetlenül
   ezen az oldalon, és melyekhez szükséges továbbra is a
   ``local_vars.php.inc`` szerkesztése. Ismertetni kell, hogyan oldódnak
   fel a felület beállításai és a konfigurációs fájl értékei közötti
   ütközések.


.. _group-settings:

Csoportok
=========

A **Csoportok** szakasz lehetővé teszi a projektfelhasználói csoportok
létrehozását és kezelését. A csoportok segítségével hozzáférés rendelhető
adatokhoz, feltöltési űrlapokhoz, modulokhoz és adminisztrációs funkciókhoz.

Az adminisztrátorok:

* létrehozhatnak egy csoportot;
* felhasználókat adhatnak hozzá egy csoporthoz, vagy eltávolíthatják őket
  onnan;
* csoportokat adhatnak más csoportokhoz, ahol a beágyazott csoportok
  támogatottak; valamint
* az így létrejövő csoportokat más hozzáférés-kezelési felületeken is
  használhatják.

A beágyazott csoportok újrafelhasználható és méretezhető
jogosultságstruktúrát biztosíthatnak. Ennek ellenére elég egyszerűnek kell
maradniuk ahhoz, hogy az adminisztrátorok meg tudják határozni az egyes
felhasználók tényleges jogosultságait.

.. TODO: Ismertetni kell a beágyazott csoportok pontos működését,
   beleértve a rekurzív tagságot, a körkörös hivatkozások megelőzését és a
   jogosultságok elsőbbségi sorrendjét. Dokumentálni kell, hogy egy csoport
   törlése eltávolítja-e annak hivatkozásait a feltöltési űrlapokból, a
   hozzáférési szabályokból, a modulokból és az adminisztrációs
   jogosultságokból.

.. TODO: Tisztázni kell, hogy a csoportnevek módosíthatók-e, miután a
   csoportot hozzáférési szabályokban használták, valamint hogy a
   hozzáférési szabályok csoportazonosítót vagy csoportnevet tárolnak-e.


.. _upload-forms:

Feltöltési űrlapok
==================

A feltöltési űrlapok határozzák meg, hogyan lehet adatokat rögzíteni vagy
importálni a projekt tábláiba. Meghatározzák az elérhető mezőket, a
beviteli vezérlőket, a validálási szabályokat és az adatgyűjtési
munkafolyamat hozzáférési beállításait.

A részletes útmutatót lásd:
:doc:`Feltöltési űrlapok kezelése <../upload_forms>`.


.. _trigger-functions:

Funkciók
========

A **Funkciók** szakasz eszközöket biztosít a projekt tábláihoz és
nézeteihez kapcsolódó SQL-szabályok és triggerek áttekintéséhez. Minden
táblához külön listázza a regisztrált szabályokat és triggereket, valamint
sablonokat biztosít egyes triggerfüggvényekhez.

A felület a következő dokumentált triggertípusokat képes létrehozni,
szerkeszteni, engedélyezni vagy letiltani:

* taxonlistatriggerek;
* előzménytriggerek; valamint
* hozzáférésiszabály-triggerek.

Ezenkívül egyéni triggerek és szabályok is létrehozhatók és
konfigurálhatók ezen a felületen.

Az adatbázis-triggerek automatikusan végrehajtódnak, amikor az adatok
megváltoznak. Egy hibás trigger elutasíthat érvényes módosításokat,
váratlanul megváltoztathat adatokat, vagy gyengítheti a
hozzáférés-szabályozást. Az egyénileg módosított triggerfüggvényeket éles
projektben történő engedélyezésük előtt tesztelni kell.


Taxonlistatrigger
-----------------

A taxonlistatrigger a projekt taxontáblájába illeszti a konfigurált
fajnévmezőben szereplő, korábban ismeretlen tudományos neveket. Ez segíthet
olyan projekt karbantartásában, amelynek fajlistája a megfigyelések
hozzáadásával bővül.

A taxontáblához hozzáadott fajnevek ezután a taxonnevek kezelésére szolgáló
felületen tarthatók karban.

:ref:`Adminisztrációs beállítások: fajnevek <species-names>`


Előzménytrigger
---------------

Az előzménytrigger rögzíti a céltábla rekordjain végzett módosításokat. Az
így létrejövő előzmények a rekord adatelőzmény-felületén jeleníthetők meg.

.. TODO: Dokumentálni kell az előzménytrigger által rögzített műveleteket
   és értékeket. Tisztázni kell, hogy tárolja-e a mezők korábbi és új
   értékeit, az időbélyegeket, a szerkesztők személyazonosságát, a
   tranzakcióazonosítókat, vagy csak a módosítások számát. Ismertetni kell
   a megőrzési, hozzáférési, visszaállítási és tárhelyigényeket is.


Hozzáférésiszabály-trigger
--------------------------

A hozzáférésiszabály-trigger a projekttábla rekordjainak sorszintű
hozzáférési szabályait tartja karban. A korlátozásokat egy konfigurált
érzékenységi mezőből származtathatja, továbbá átviheti a rekord
létrehozásához használt feltöltési űrlap olvasási és írási jogosultságait.

Ha például egy feltöltési űrlap olvasási hozzáférést biztosít az A és B
csoportnak, írási hozzáférést pedig a C csoportnak, a trigger ezeket a
hozzárendeléseket hozzáadhatja az adott űrlappal létrehozott minden
rekordhoz tartozó szabálytábla-bejegyzéshez.

Ez a trigger a csoport- vagy sorszintű hozzáférési korlátozásokat használó
projektek számára fontos. Konfigurációjának összhangban kell lennie a
projekt általános hozzáférési beállításaival és a szabálytábla sémájával.

További információért lásd:
:doc:`Adathozzáférés <../data_access>`.

.. TODO: Ismertetni kell, hogyan kezeli a trigger az SQL, az API vagy más,
   társított feltöltési űrlappal nem rendelkező folyamat által létrehozott
   rekordokat. Dokumentálni kell a működését akkor is, ha egy rekordot
   frissítenek, másik feltöltéshez helyeznek vagy törölnek.

.. TODO: Tisztázni kell, hogy a trigger engedélyezése létrehoz-e
   szabályokat a meglévő rekordokhoz, vagy csak a későbbi változtatásokat
   érinti. Támogatott eljárást kell dokumentálni az összes szabály
   újragenerálásához és ellenőrzéséhez.


.. _species-names:

Fajnevek
========

A **Fajnevek** szakasz a projekt taxontábláját kezeli. A fajnevek a
következő dokumentált kategóriákhoz rendelhetők:

* elfogadott név;
* szinonima;
* köznyelvi név; valamint
* hibásan írt név.

A taxontáblában tárolt neveket a taxonokhoz kapcsolódó keresési felületek,
valamint a taxonneveket felismerő vagy javító háttérfolyamatok használják.

.. TODO: Meg kell erősíteni a kategóriák jelenlegi neveit, és szükség
   esetén javítani kell a ``misspelled`` szó forrásfelületen szereplő
   helyesírását. Dokumentálni kell az elfogadott nevek, szinonimák,
   köznyelvi nevek és hibásan írt változatok között engedélyezett
   kapcsolatokat.

.. TODO: Ismertetni kell, mely taxonómiai mezőket tárolja a rendszer, hogyan
   importálhatók és exportálhatók a nevek, valamint hogyan előzi meg a
   felület a duplikált vagy körkörös szinonimakapcsolatokat.

.. TODO: Azonosítani kell a ``taxon-name-repair-background-jobs`` funkció
   aktuális nevét és működését, és hivatkozni kell a konfigurációs
   útmutatójára.


.. _localisation:

Fordítások
==========

Az OpenBioMaps globális és projektspecifikus fordításokat használ.


Globális fordítások
-------------------

A globális fordítások az
`OpenBioMaps fordítási platformján
<https://translate.openbiomaps.org/>`_ adhatók hozzá és fejleszthetők. A
platform a webalkalmazás, a mobilalkalmazások és más OpenBioMaps-összetevők
fordításait tartalmazza. A közreműködők új nyelv hozzáadását is
javasolhatják.

.. TODO: Dokumentálni kell a fordítási platform fiókkezelési,
   felülvizsgálati és kiadási munkafolyamatát. Ismertetni kell, hogy egy
   elfogadott globális fordítás mikor válik elérhetővé egy
   OpenBioMaps-szerveren.


Helyi fordítások
----------------

A helyi fordítások lehetővé teszik, hogy egy projekt projektspecifikus
felületi szövegeket határozzon meg. A fordítási kulcsok a ``str_``
előtagból és egy leíró angol azonosítóból állnak. Egy projekt például
meghatározhatja a ``str_observations`` kulcsot, és megadhatja annak
fordítását minden aktív nyelven.

Nyilvános példa itt érhető el:

https://openbiomaps.org/projects/checkitout/upload/?form=426&type=web

.. TODO: Dokumentálni kell, hol hozhatók létre a helyi fordítások, hogyan
   választhatók ki az aktív nyelvek, mely összetevők ismerik fel a helyi
   kulcsokat, és mi történik hiányzó fordítás esetén. Tisztázni kell, hogy
   a helyi fordítások felülírják-e az azonos kulcsot használó globális
   szövegeket.

.. TODO: A nyilvános példát stabil képernyőképpel vagy leírással kell
   helyettesíteni vagy kiegészíteni, mert a hivatkozott projekt és
   űrlapazonosító megváltozhat.


.. _module-settings:

Modulok
=======

A modulok kibővítik az OpenBioMaps-projektben elérhető funkciókat.
Konfigurációjuk és hozzáférési követelményeik az egyes moduloktól függenek.

A modulok gyakran alapvető funkciókat biztosítanak, például szöveges
keresési felületeket a térképoldalon; más esetekben meghatározott
feladatokhoz készült eszközöket nyújtanak. A modulok működése gyakran
testreszabható.

További információért lásd:
:doc:`Modulok <../modules>`.


.. _interrupted-uploads:

Megszakított feltöltések
========================

A **Megszakított feltöltések** szakasz a mentett vagy befejezetlen
fájlfeltöltéseket és webes űrlapos adatrögzítési munkameneteket sorolja fel.
Állapotától függően egy megszakított feltöltés visszaállítható vagy
elvethető.

Az adminisztrátoroknak törlés előtt meg kell győződniük arról, hogy a
feltöltésre már nincs szükség. Egy megszakított feltöltés olyan munkát
tartalmazhat, amelyet a tulajdonosa később folytatni kíván.

.. TODO: Dokumentálni kell, ki tekintheti meg, folytathatja vagy törölheti
   más felhasználó megszakított feltöltését. Ismertetni kell a manuálisan
   mentett feltöltés, az automatikus biztonsági másolat és a megszakított
   feltöltés közötti különbséget.

.. TODO: Meg kell határozni a megőrzési időket, a tárhelykorlátokat, az
   automatikus törlési szabályokat, valamint azt, hogy egy megszakított
   feltöltés törlése a hozzá tartozó ideiglenesen feltöltött fájlokat is
   törli-e.


.. _file-manager:

Fájlkezelő
==========

A **Fájlkezelő** szakasz eszközöket biztosít a projekthez feltöltött
csatolmányok kezeléséhez. Segítségével böngészhetők a csatolmányok,
áttekinthetők az adatbázisrekordokkal fennálló kapcsolataik, és exportok
hozhatók létre.

A dokumentált funkciók a következők:

* a feltöltött csatolmányok listázása;
* a csatolmányok szűrése és rendezése;
* a fájlokhoz tartozó megjegyzések szerkesztése;
* csatolmányok összekapcsolása adatrekordokkal;
* a meglévő fájlkapcsolatok kezelése; valamint
* egy adattáblához kapcsolódó csatolmányok exportálása.

A tömeges export feldolgozása háttérfolyamatként történik. A feldolgozás
befejeződése után a rendszer hivatkozást biztosít az elkészült archívum
letöltéséhez.

A csatolmánykezelési és exportálási funkciókhoz való hozzáférést az arra
jogosult felhasználókra kell korlátozni. Az exportált fájlokra továbbra is
vonatkoznak a projekt adathozzáférési és adatvédelmi követelményei.

.. TODO: Meg kell erősíteni az elérhető szűrőket és szerkeszthető
   metaadatokat. Dokumentálni kell, mely csatolmányformátumok előnézete
   jeleníthető meg, és hogy egy fájlkapcsolat módosítása frissíti-e a
   megfelelő rekordot is.

.. TODO: Ismertetni kell, hogyan alkalmazzák a csatolmányexportok a sor- és
   oszlopszintű hozzáférési szabályokat, hol tárolja a rendszer a
   létrehozott archívumokat, meddig maradnak érvényesek a letöltési
   hivatkozások, és kik használhatják azokat.

.. TODO: Tisztázni kell, hogy támogatott-e fájl törlése ezen a felületen,
   és mi történik a törölt fájlra hivatkozó rekordokkal, metaadatokkal,
   bélyegképekkel és biztonsági mentésekkel.


.. _sql-query-settings:

SQL-lekérdezési beállítások
===========================

Az **SQL-lekérdezési beállítások** szakasz határozza meg a MapServer
rétegeihez és a webalkalmazás szöveges lekérdezési eredményeihez használt
lekérdezések összeállítására szolgáló sablonokat. Ezek a sablonok az
SQL-hez hasonlítanak, de olyan OpenBioMaps-helyőrzőket tartalmaznak,
amelyeket a lekérdezésértelmező dinamikusan helyettesít.

Minden lekérdezési sablont egy webes térképréteghez kell kapcsolni. A
MapServer mapfile fájljában a dinamikusan létrehozott lekérdezést használó
WMS-rétegnek ``%query%`` helyőrzőt tartalmazó ``DATA`` definícióval kell
rendelkeznie.

A lekérdezési sablonok százalékjelekkel határolt helyőrzőket
tartalmazhatnak. Az alapfunkciók és a telepített modulok futásidőben
SQL-részletekkel helyettesíthetik ezeket a helyőrzőket.

.. TODO: Teljes referenciát kell készíteni minden támogatott helyőrzőről,
   beleértve az érvényes helyüket, helyettesítési értékeiket,
   függőségeiket és biztonsági korlátozásaikat. A forrásszöveg a
   ``%morefilter%`` és a ``%morefilters%`` alakra egyaránt hivatkozik;
   meg kell erősíteni, melyik az érvényes.


Alapvető lekérdezési sablon
---------------------------

A lekérdezési sablon használhat például ``%qstr%`` helyőrzőt a
lekérdezési feltételekhez és ``%morefilter%`` helyőrzőt a további
szűrőkhöz:

.. code-block:: sql

   SELECT obm_id, %grid_geometry% AS obm_geometry
       %selected%
   FROM %F%checkitout c%F%
       %uploading_join%
       %rules_join%
       %taxon_join%
       %grid_join%
       %search_join%
       %morefilter%
   WHERE %geometry_type% %envelope% %qstr%

A ``%F%`` jelölők az elsődleges ``FROM`` relációt és annak aliasát
azonosítják, hogy az értelmező felbonthassa és kibővíthesse a sablont.

.. TODO: Ismertetni kell, miért kell az elsődleges relációt ``%F%``
   jelölők közé helyezni, támogatottak-e a sémával minősített és
   idézőjeles nevek, és mely aliasok érhetők el a létrehozott
   SQL-részletek számára.


Összekapcsolások hozzáadása
---------------------------

További összekapcsolások ``%J%`` jelölők közé helyezhetők:

.. code-block:: sql

   SELECT
       n.obm_geometry,
       n.obm_id,
       -2 AS date_part,
       nestbox_type,
       project_id,
       beinaction
       %selected%
   FROM %F%public_nestbox_data n%F%
       %J%LEFT JOIN public_nestbox_data_observations o
           ON o.nestbox_id = n.obm_id%J%
       %taxon_join%
       %morefilter%
   WHERE %envelope% %qstr%

.. TODO: Ismertetni kell, hogyan dolgozza fel az értelmező a több ``%J%``
   blokkot, valamint eltávolíthat-e egy összekapcsolást, ha arra egyik
   kiválasztott mezőhöz vagy szűrőhöz sincs szükség.


Összetett lekérdezési sablonok
------------------------------

A sablonok közös táblakifejezéseket és más SQL-konstrukciókat is
használhatnak:

.. code-block:: sql

   WITH aall AS (
       SELECT
           o.obm_id,
           n.obm_geometry,
           nestbox_type,
           project_id,
           beinaction,
           COALESCE(
               EXTRACT(DAY FROM (CURRENT_DATE - datum)::interval),
               '-1'
           ) AS date_part
           %selected%
       FROM %F%public_nestbox_data_observations o%F%
           %J%LEFT JOIN public_nestbox_data n
               ON nestbox_id = n.obm_id%J%
           %taxon_join%
           %morefilter%
       WHERE 1 = 1 %envelope% %qstr%
   )
   SELECT *
   FROM aall
   ORDER BY date_part DESC

Egy jellemző egyszerű sablon a következő formájú:

.. code-block:: sql

   SELECT obm_id, obm_geometry %selected%
   FROM %F%checkitout c%F%
       %uploading_join%
       %rules_join%
       %taxon_join%
       %morefilter%
   WHERE %geometry_type% %envelope% %qstr%

A lekérdezési sablonok a helyes működést és az adathozzáférést egyaránt
befolyásolják. Egy hibás összekapcsolás vagy hiányzó hozzáférésiszabály-
helyőrző olyan rekordokat vagy mezőket tehet hozzáférhetővé, amelyekhez a
hozzáférést korlátozni kellene. Minden lekérdezést nyilvános,
hitelesített és meghatározott csoporthoz tartozó felhasználóként is
tesztelni kell, mielőtt elérhetővé válik.

.. TODO: Dokumentálni kell, mely hozzáférés-szabályozási helyőrzők
   kötelezők, és hogy az alkalmazás elutasítja-e az ezeket kihagyó
   sablonokat. Ismertetni kell, hogyan történik a paraméterértékek
   escape-elése vagy kötése az SQL-injektálás megelőzése érdekében.

.. TODO: Eljárást kell hozzáadni egy sablon teszteléséhez, a létrehozott
   SQL megvizsgálásához, a helyőrzőhibák diagnosztizálásához és az előző
   verzió visszaállításához.


.. _map-settings:

Térképbeállítások
=================

A **Térképbeállítások** szakasz a webes térkép térbeli rétegeit és a
hozzájuk tartozó MapServer-definíciókat konfigurálja. A webes térkép és a
MapServer beállításainak összhangban kell maradniuk, hogy a rétegek a
kívánt adatforrást, vetületet, kiterjedést és stílust használják.


Webes térképrétegek
-------------------

A webes térkép beállításai az OpenLayers-alapú térképi felületet
konfigurálják. Az adminisztrátorok többek között a következő beállításokat
határozhatják meg:

* a térkép kezdeti középpontja és nagyítási szintje;
* az elérhető alaptérképek és átfedő rétegek;
* az alapértelmezetten látható rétegek;
* a rétegek, projekttáblák és lekérdezési sablonok közötti kapcsolat;
  valamint
* a rétegek megjelenésének és működésének egyes jellemzői.

.. TODO: Dokumentálni kell minden szerkeszthető OpenLayers-beállítást,
   annak elvárt formátumát, alapértelmezett értékét és támogatott
   koordináta-referenciarendszerét. Ismertetni kell, hogyan konfigurálható
   a rétegek sorrendje, láthatósági tartománya, átlátszósága, jelmagyarázata
   és lekérdezhetősége.


MapServer-beállítások
---------------------

A haladó adminisztrátorok szerkeszthetik a projekt nyers MapServer
mapfile fájlját. A mapfile határozza meg a rétegek adatforrásait, térbeli
referenciarendszereit, kiterjedését, stílusát és megjelenítési beállításait.

A mapfile módosításai elérhetetlenné tehetik a projektrétegeket, vagy nem
szándékos adatforrást tehetnek hozzáférhetővé. Őrizzen meg egy működő
verziót, és üzembe helyezés előtt ellenőrizze a szerkesztett mapfile
érvényességét.

.. TODO: Dokumentálni kell, hol tárolja a rendszer a mapfile fájlt,
   verziókövetettek-e a módosítások, hogyan ellenőrizhető a szintaxis
   érvényessége, és hogyan állítható vissza egy korábbi konfiguráció.

.. TODO: Ismertetni kell, hogy a mapfile mely részeit hozza létre az
   OpenBioMaps, és mely részek szerkeszthetők biztonságosan anélkül, hogy
   felülíródnának.


Térbeli referenciarendszerek
----------------------------

A térképrétegeknek megfelelően meghatározott térbeli
referenciarendszereket kell használniuk. A konfigurált SRID határozza meg,
hogyan értelmezi és alakítja át a rendszer a koordinátákat, amikor
különböző forrásokból származó adatok együtt jelennek meg.

A térkép kiterjedés- és vetületbeállításai szabályozzák a webes térképen
megjelenített területet és koordináta-rendszert. Ezeknek kompatibilisnek
kell lenniük a rétegadatokkal, a MapServer konfigurációjával és az
OpenLayers beállításaival.

.. TODO: Azonosítani kell a webes térképhez szükséges vetületet, a
   támogatott forrásvetületeket és az átalakítás helyét. Útmutatást kell
   adni a kiterjedés kiválasztásához és a hibás helyen megjelenő rétegek
   diagnosztizálásához.


.. _member-settings:

Tagok
=====

A **Tagok** szakasz a projektben regisztrált felhasználókat sorolja fel. Az
adminisztrátorok kezelhetik a projekttagságot, az állapotot és a
csoporthozzárendeléseket.

A dokumentált tagi állapotok a következők:

``Normal``
   A felhasználó megkapja a projekt szokásos feltöltési és lekérdezési
   jogosultságait. A részletesebb csoporthozzárendelések és hozzáférési
   szabályok módosíthatják ezeket a jogosultságokat.

``Operator``
   A felhasználó hozzáfér a projekt minden funkciójához és adatához.

``Suspended``
   A felhasználó nem fér hozzá a projekt funkcióihoz és adataihoz. Egy
   felhasználó felfüggesztése hasonló a projekttagság letiltásához, de nem
   törli a profilját.

A projekt alapítója teljes hozzáféréssel rendelkezik a projekthez, és nem
szükséges operátori állapotot rendelni hozzá. A csoporthozzárendelések ezen
az oldalon is módosíthatók, bár több felhasználó kezeléséhez a
**Csoportok** felület kényelmesebb lehet.

A kapcsolódó beállításokat lásd:
:ref:`Csoportok <groups>` és
:ref:`Adminisztrációs hozzáférés <administrative-access>`.

.. TODO: Meg kell erősíteni a jelenlegi állapotneveket, és pontosan meg
   kell határozni az alapítók, tulajdonosok, gazdák, operátorok és normál
   felhasználók jogosultságait. Ezeket a szerepkörneveket egységesíteni
   kell a dokumentációban.

.. TODO: Ismertetni kell, hogy a felfüggesztés csak az aktuális projektet
   vagy a felhasználó teljes szerverfiókját érinti-e. Dokumentálni kell az
   API-tokenekre, aktív munkamenetekre, ütemezett feladatokra,
   rekordtulajdonra, meghívókra és üzenetekre gyakorolt hatását.


Másik felhasználó profiljának megtekintése
------------------------------------------

A tag neve a profiloldalára mutató hivatkozás. A szükséges jogosultsággal
rendelkező adminisztrátorok az oldal jobb felső részén egy titkos
felhasználót ábrázoló ikont láthatnak. Ez a funkció megnyitja egy másik
felhasználó profilját, miközben az adminisztrátor továbbra is a saját
fiókjával marad hitelesítve.

A felület által használt ikont a
`Fork Awesome
<https://forkaweso.me/Fork-Awesome/icon/user-secret/>`_ dokumentálja.

Ez a funkció személyes információkat és felhasználóspecifikus tartalmakat
tehet hozzáférhetővé. Hozzáférését korlátozni kell, használatának pedig meg
kell felelnie a projekt adatvédelmi és auditálási irányelveinek.

.. TODO: Tisztázni kell, hogy ez a funkció megszemélyesíti-e a
   felhasználót, vagy csak a profil adminisztrációs megtekintését teszi
   lehetővé. Dokumentálni kell az engedélyezett műveleteket, azt, hogy az
   érintett felhasználó kap-e értesítést, valamint hogy a hozzáférést
   rögzíti-e auditnapló.


.. _message-templates:

Üzenetsablonok
==============

Az üzenetsablon-szerkesztő jelenleg nem érhető el.

A rendszer vagy egy projekt által automatikusan küldött üzenetek
sablonokból készülnek. Az OpenBioMaps globális sablonokat biztosít a
megvalósított üzenettípusokhoz, a projektek pedig létrehozhatnak ezeket
felülíró helyi változatokat.

Egy globális sablon testreszabásához válassza ki, szerkessze a tartalmát,
majd mentse helyi változatként. A sablonok olyan változókat
tartalmazhatnak, amelyeket a rendszer az üzenet elküldésekor helyettesít.
Az egyes sablonok által támogatott változókat az üzenetet küldő funkció,
modul vagy háttérfolyamat határozza meg.

Egyéni modulokhoz és háttérfolyamatokhoz új sablonok is létrehozhatók.

.. TODO: Dokumentálni kell a sablonmezőket, a támogatott
   üzenetformátumokat, a nyelvkezelést, a tartalékértékek sorrendjét,
   valamint a helyi felülírás globális változatra történő
   visszaállításának eljárását.

.. TODO: Ismertetni kell, történik-e a sablontartalom escape-elése, és
   milyen HTML vagy jelölőnyelv engedélyezett. A sablonszerkesztést meg
   kell vizsgálni a nem biztonságos hivatkozások, a HTML-injektálás és a
   változók nem szándékos felfedésének kockázata szempontjából.


Változók és beillesztett sablonok
---------------------------------

A változókat százalékjelek közé kell írni, például ``%USER_NAME%``. A
dokumentált globális változók a következők:

``%PROJECT_TABLE%``
   A projekt adatbázis-azonosítója vagy táblaneve.

``%PROJECT_TITLE%``
   A projekt rövid leírása.

``%PROJECT_DESCRIPTION%``
   A projekt hosszú leírása.

``%USER_NAME%``
   A címzett vagy az érintett felhasználó neve.

``%URL%``
   Az üzenethez kapcsolódó URL.

``%OB_DOMAIN%``
   Az üzenethez kapcsolódó OpenBioMaps-tartomány.

``%DOMAIN%``
   A ``projects`` táblában meghatározott tartománynév.

``%PROTOCOL%``
   A ``projects`` táblában meghatározott protokoll.

Egy sablon másik sablont is beilleszthet. Például a ``@footer@``
hozzáfűzése beilleszti a ``footer`` nevű sablont.

.. TODO: Meg kell erősíteni minden globális változó pontos jelentését és
   elérhetőségét. Különösen meg kell különböztetni a
   ``%PROJECT_TABLE%``, ``%OB_DOMAIN%``, ``%DOMAIN%`` és ``%URL%``
   változókat.

.. TODO: Dokumentálni kell, hogy a beillesztett sablonok maguk is
   illeszthetnek-e be más sablonokat, hogyan kezeli a rendszer a hiányzó
   változókat vagy sablonokat, és megakadályozza-e a rekurzív
   beillesztést.


Előre meghatározott sablonok
----------------------------

A dokumentált felhasználókkal kapcsolatos sablonok:

``welcome_to``
   Üdvözli a felhasználót a projektben.

``change_email_address``
   Megerősítő hivatkozást küld a felhasználó e-mail-címének
   megváltoztatásához.

``dropmyaccount``
   Megerősíti egy fiók törlésére irányuló kérelmet.

``create_new_project``
   Megerősíti egy projekt létrehozását.

``invitation``
   Meghívót küld a projekthez való csatlakozásra.

``invitation_accomplished``
   Jelzi, hogy a meghívót elfogadták.

``invitation_request``
   Értesíti az adminisztrátorokat egy meghívási kérelemről.

``lostpw``
   Támogatja a jelszó helyreállítását.

A dokumentált általános célú sablonok:

``new_gitlab_issue``
   Egy beküldött hibajelentés másolatát tartalmazza.

``new_shared_polygon``
   Bejelent egy újonnan megosztott poligont.

``new_upload_news``
   Bejelent egy új feltöltést a projekt hírei között.

``new_upload_report``
   Értesíti az adminisztrátorokat egy új feltöltésről.

``footer``
   Általános üzenetláblécet biztosít.

``interconnect_request``
   Összekapcsolási kérelmet támogat.

A dokumentált értékelési értesítési sablonok:

``data_evaluation_commenters``
   Értesíti a korábbi hozzászólókat, amikor egy rekordhoz új megjegyzés
   érkezik.

``data_evaluation_owner``
   Értesíti a tulajdonost, amikor az általa feltöltött rekordhoz
   megjegyzés érkezik.

``upload_evaluation_commenters``
   Értesíti a korábbi hozzászólókat, amikor egy feltöltéshez új megjegyzés
   érkezik.

``upload_evaluation_owner``
   Értesíti a tulajdonost, amikor az általa feltöltött feltöltéshez
   megjegyzés érkezik.

``user_evaluation_commenters``
   Értesíti a korábbi hozzászólókat, amikor egy felhasználóhoz új
   megjegyzés érkezik.

``user_evaluation_owner``
   Értesíti a felhasználót, amikor megjegyzést kap.

A dokumentált modulokhoz kapcsolódó sablonok:

``dlr_new_request``
   Értesíti a projektadminisztrátorokat egy új letöltési kérelemről. A
   dokumentált változók: ``username``, ``requestid`` és
   ``request_message``.

``dlr_request_registered``
   Megerősíti a felhasználónak, hogy letöltési kérelmét regisztrálták.

``incomplete_list_processed``
   Jelzi, hogy egy hiányos lista feldolgozása megtörtént.

``incomplete_list_unprocessed``
   Jelzi, hogy egy hiányos listát nem sikerült feldolgozni.

.. TODO: Ellenőrizni kell, hogy minden sablonazonosító aktuális-e, és hozzá
   kell adni az egyes sablonok számára elérhető változókat. Az
   ``interconnect_request``, ``incomplete_list_processed`` és
   ``incomplete_list_unprocessed`` célja további magyarázatot igényel.

.. TODO: Tisztázni kell, hogy a ``dropmyaccount`` egy szerverszintű fiókot
   vagy csak a projekttagságot törli-e, valamint hogy a
   ``create_new_project`` létrehozás előtti megerősítő kérés vagy
   létrehozás utáni értesítés.


.. _server-info:

Szerverinformációk
==================

A **Szerverinformációk** szakasz az OpenBioMaps-szerverről és a projekt
által használt erőforrásokról jelenít meg kiválasztott információkat. A
szerver konfigurációjától függően ezek a következők lehetnek:

* a telepített OpenBioMaps-alkalmazás verziója;
* a projektfájlok, csatolmányok és feltöltések lemezhasználata;
* az előző 1, 5 és 15 perc terhelési átlaga;
* a processzormagok számával normalizált szerverterhelés;
* a rendelkezésre álló memória; valamint
* a Supervisor adminisztrációs felületére mutató hivatkozás.

Ezek az értékek segíthetnek az adminisztrátoroknak az erőforráskorlátok
azonosításában, és diagnosztikai információkat biztosíthatnak a
szerverüzemeltetőknek. A részletes szerverinformációkhoz való hozzáférést
korlátozni kell, mert a verzió- és infrastruktúra-információk biztonsági
szempontból érzékenyek lehetnek.

.. TODO: Meg kell erősíteni, mely értékek érhetők el a
   projektadminisztrátorok számára, és melyek igényelnek szerverszintű
   jogosultságot. Dokumentálni kell az egyes mérőszámok mértékegységét,
   frissítési időközét, adatforrását, figyelmeztetési küszöbértékeit és
   értelmezését.

.. TODO: Tisztázni kell, hogy a Supervisor-hivatkozás mindig elérhető-e,
   mely Supervisor-termékre utal, és hogyan történik a külső felülethez
   való hozzáférés hitelesítése.


.. _server-logs:

Szervernaplók
=============

A **Szervernaplók** szakasz hozzáférést biztosít a szerver konfigurációja
által elérhetővé tett naplókhoz. A dokumentált források a következők:

* alkalmazás- vagy rendszernaplók;
* MapServer-naplók;
* háttérfolyamat-események; valamint
* háttérfolyamat-hibák.

A felület szűrést és keresést biztosíthat. A naplók felhasználóneveket,
rekordazonosítókat, lekérdezési részleteket, fájlútvonalakat,
kérésparamétereket és más érzékeny információkat tartalmazhatnak.
Hozzáférésüknek és megőrzésüknek meg kell felelnie a szerver biztonsági és
adatvédelmi irányelveinek.

.. TODO: Meg kell erősíteni az elérhető naplóforrásokat és azt, hogy az
   élő frissítés jelenleg támogatott-e. Dokumentálni kell minden napló
   helyét, formátumát, időzónáját, rotációját, megőrzését és maximális
   eredményméretét.

.. TODO: Ismertetni kell, milyen személyes vagy bizalmas adatok
   jelenhetnek meg a naplókban, és hogyan tölthetik le, takarhatják ki vagy
   törölhetik az adminisztrátorok a naplótartalmat. Azt is rögzíteni kell,
   hogy a naplók megtekintését maga a rendszer naplózza-e.


.. _background-job-settings:

Háttérfolyamat-beállítások
==========================

A háttérfolyamatok lehetővé teszik, hogy egy projekt ütemezett vagy
manuálisan indított feladatokat hajtson végre folyamatos felhasználói
beavatkozás nélkül. Többek között a következő műveletekhez használhatók:

* fajnévadatok karbantartása;
* rekordok validálása;
* adatok importálása vagy exportálása;
* ideiglenes táblák tisztítása;
* elemzések futtatása; valamint
* materializált nézetek frissítése.

A háttérfolyamat önálló program. Az OpenBioMaps-feladatok gyakran PHP
nyelven készülnek, de a szerver Python, R, Bash vagy más telepített nyelven
írt programokat is támogathat.

Az adminisztrációs felület a következőkre használható:

* előre meghatározott feladatok telepítése központi Git-repository-ból;
* projektspecifikus feladat feltöltése;
* a telepített feladatok áttekintése;
* a feladatparaméterek és ütemezések konfigurálása;
* feladatok engedélyezése vagy letiltása;
* feladat manuális indítása;
* a legutóbbi kimenet és végrehajtási állapot megvizsgálása; valamint
* a feladat forráskódjának szerkesztése, ahol ez a funkció engedélyezett.

Részletes naplók a **Szervernaplók** szakaszban érhetők el.

Egy feladat szerkesztése vagy feltöltése végrehajtható kód szerverre
telepítésével egyenértékű. Ezeket a funkciókat megbízható
adminisztrátorokra kell korlátozni, az egyéni feladatokat pedig ellenőrizni
kell parancsinjektálás, nem biztonságos fájlhozzáférés, hitelesítési
adatok felfedése és túlzott erőforrás-használat szempontjából.

.. TODO: Dokumentálni kell egy feladat szükséges csomagszerkezetét,
   jegyzékét, belépési pontját, támogatott nyelveit, végrehajtó
   felhasználóját, munkakönyvtárát, környezeti változóit, függőségeit,
   időkorlátját és erőforrás-korlátait.

.. TODO: Ismertetni kell, hogyan történik a központi repository-ból
   származó feladatok hitelesítése, verziókezelése, frissítése és
   felülvizsgálata. Tisztázni kell, hogy egy frissítés felülírja-e a helyi
   módosításokat, és hogyan állítható vissza egy korábbi verzió.


További információért lásd:
:doc:`Háttérfolyamatok <../jobs>`.

Feladatok ütemezése
-------------------

Először a szerver rendszerszintű ütemezőjét kell konfigurálni. Docker-
telepítésben ez jellemzően a gazdagépen futó cron-folyamat. Ez rendszeresen
meghívja a projekt ütemezőjét, amely elindítja az esedékes feladatokat.

Újonnan telepített vagy módosított feladat ütemezése előtt:

#. tekintse át a konfigurációját és forráskódját;
#. a **Run** használatával futtassa manuálisan;
#. várja meg a végrehajtás befejeződését;
#. vizsgálja meg az eredményt és a naplókat; valamint
#. csak sikeres teszt után konfigurálja az ismétlődő ütemezést.

A projektütemező cronhoz hasonló perc-, óra- és napmezőket használ. A
csillag az adott mező minden érvényes értékét jelenti.

.. TODO: Dokumentálni kell minden ütemezőmezőt és az elfogadott
   cron-szintaxist, beleértve a tartományokat, listákat, lépéseket,
   hónapot, hétköznapot és időzónát. Tisztázni kell, hogy a rendszer
   megakadályozza-e ugyanazon feladat egymást átfedő végrehajtását.


Rendszerszintű Docker-példa
---------------------------

A következő példa a gazdagépről hívja meg egy projekt ütemezőjét:

.. code-block:: console

   */5 * * * * /usr/local/bin/docker-compose -f /srv/docker/openbiomaps/docker-compose.yml exec -u www-data -T app php /var/www/html/biomaps/root-site/projects/myproject/jobs.php

A Compose-fájlt, a szolgáltatást, a projektútvonalat és a végrehajtó
felhasználót a telepítésnek megfelelő értékekre kell cserélni.

.. TODO: Ellenőrizni kell ezt a parancsot a jelenleg támogatott
   Docker-telepítéssel. Az újabb telepítések a ``docker-compose`` helyett a
   ``docker compose`` parancsot használhatják.

.. TODO: Meg kell adni az ajánlott meghívási időközt, és ismertetni kell,
   hogy a parancs ötpercenkénti futtatása megakadályozza-e az
   egyperces ütemezésű feladatok tervezett időben történő futását. A
   gazdagépszintű cron-feladat naplózására és hibajelzésére vonatkozó
   ajánlásokat is hozzá kell adni.


.. _project-description:

Projektleírás
=============

A **Projektleírás** szakasz határozza meg az oldal fejlécében megjelenő
projektnevet és a hosszabb projektleírást. Minden aktív nyelvhez külön
értékek adhatók meg.

A rövid és hosszú leírás a projekt metaadataiban, üzenetsablonjaiban és
összefoglaló oldalain is használható. Ezért egyértelműen azonosítaniuk kell
a projektet, és szükség esetén aktuális kapcsolattartási vagy háttér-
információkat kell tartalmazniuk.

.. TODO: Dokumentálni kell a támogatott formázást, a maximális
   hosszúságokat, a tartaléknyelvet és minden olyan felületet, amelyen a
   rövid és hosszú leírás megjelenik. Tisztázni kell, hogy ezek az értékek
   pontosan megfelelnek-e az üzenetsablonok ``%PROJECT_TITLE%`` és
   ``%PROJECT_DESCRIPTION%`` változóinak.


.. _data-management-page:

Adatkezelés
===========

Az **Adatkezelés** szakasz feltöltések és megfigyelési listák
összefoglalóit biztosítja. Segíthet az adminisztrátoroknak a legutóbbi
beküldések áttekintésében, a közreműködők azonosításában, valamint a
kapcsolódó rekordok, feltöltések és útvonalnaplók közötti navigációban.

A dokumentált funkciók a következők:

* megfigyelési listák felsorolása feltöltő, dátum vagy útvonalnapló
  szerint;
* az egyes felhasználók és táblák által feltöltött rekordszám
  összesítése;
* az előző 90 nap során beküldött megfigyelési listák megjelenítése;
  valamint
* az előző 30 nap során beküldött útvonalnaplók megjelenítése.

Az interaktív táblák – ahol ez támogatott – szűrést és rendezést
biztosítanak.

.. TODO: Meg kell határozni a ``megfigyelési lista`` fogalmát, és
   ismertetni kell, hogyan kapcsolódik egy feltöltéshez, az egyes
   rekordokhoz és egy útvonalnaplóhoz. Dokumentálni kell az egyes
   összefoglalókból elérhető hivatkozásokat és műveleteket.

.. TODO: Meg kell erősíteni, hogy a 90 és 30 napos időközök rögzítettek,
   konfigurálhatók vagy csak alapértelmezettek-e. Ismertetni kell, mely
   időzónát és időbélyeget használja a rendszer a legutóbbi tevékenységek
   kiválasztásához.

.. TODO: Tisztázni kell, hogy az összefoglalók tartalmazzák-e a törölt,
   elutasított vagy részben befejezett feltöltéseket, és hogy a sorszintű
   hozzáférési korlátozások hogyan befolyásolják az adminisztrátor számára
   megjelenített értékeket.
