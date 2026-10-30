# Modulok

A modulok az OpenBioMaps webalkalmazás konfigurálható bővítményei. Hozzáadhatnak felhasználói felületi elemeket, adatfeldolgozási funkciókat, exportformátumokat, adminisztrációs eszközöket, API-kat vagy külső szolgáltatásokkal való integrációkat.

A moduloknak két fő hatóköre van:

- A **projektszintű modulok** a teljes projektre vonatkozó funkciókat biztosítanak, például térbeli alakzatok kezelését, csatolmányok támogatását vagy PostgreSQL-felhasználók létrehozását.
- A **táblaszintű modulok** egy adott adattáblára vonatkoznak, például a térképoldal szűrőire, az eredmények megjelenítésére, adatátalakításokra vagy exportformátumokra.

A modulok az alkalmazás hookjaihoz kapcsolódnak. A legtöbb felhasználói hook a térképoldalon és a profiloldalon található, de a modulok adminisztrációs oldalakat, API-kat, háttérfeladatokat és feltöltéshez kapcsolódó funkciókat is hozzáadhatnak.

A legtöbb modul JSON formátumú paramétereket fogad. Egyes modulok ehelyett külön adminisztrációs felületet biztosítanak, mások pedig JSON-paramétereket és további adatbázis- vagy MapServer-konfigurációt egyaránt igényelnek.

> **Verziókompatibilitás:** Az elérhető modulok és paramétereik változhatnak az OpenBioMaps kiadásai között. A telepített alkalmazás moduladminisztrációs oldala tartalmazza a projekt számára elérhető modulok irányadó listáját. Mielőtt egy másik telepítésből konfigurációt másolna át, ellenőrizze a modul forrását és a kiadási megjegyzéseket.

## Moduladminisztráció

A modulok a **Projektadminisztráció → Modulok** oldalon engedélyezhetők és konfigurálhatók.

Egy modul általában:

- hozzáadható a projekthez;
- felhasználókhoz vagy csoportokhoz rendelhető;
- JSON-paraméterekkel konfigurálható;
- engedélyezhető vagy letiltható; valamint
- megnyitható egy modulspecifikus adminisztrációs oldalon, ha van ilyen.

A modulnevek és a JSON-kulcsok megkülönböztetik a kis- és nagybetűket. Mentés előtt ellenőrizze a JSON érvényességét. A JSON nem engedélyez megjegyzéseket vagy záró vesszőket.

### Egyéni modul hozzáadása

Egyéni modulok tölthetők fel és adhatók hozzá a projekthez. A fejlesztőknek a `resources/includes/modules/examples/` könyvtárban található példamodulokat kell kiindulási alapként használniuk, és össze kell hasonlítaniuk azok megvalósítását a telepített OpenBioMaps-kiadásban szereplő modulokkal.

Az egyéni modulok kódját üzembe helyezés előtt felül kell vizsgálni. A modul az alkalmazás részeként fut, és hozzáférhet a projekt adataihoz, a hitelesített felhasználó munkamenetéhez és az adatbázis-kapcsolatokhoz.

#### A modulosztály szerződése

A modulok a `module` alaposztályt bővítik. Az alkalmazás a modult `new your_module($method, $stored_params, $runtime_arguments)` formában hozza létre. Az örökölt konstruktor a tárolt paramétereket a `$this->params` tulajdonságba normalizálja, ellenőrzi a hozzáférést, majd meghívja a kért metódust. A modul tartsa meg az örökölt konstruktort, és ne definiáljon sajátot, hogy ezek az ellenőrzések mindig lefussanak.

- Az automatikus horgok csak akkor futnak le, ha a nevük szerepel a modulmetaadatok `Methods` listájában. A horgok felderítése nem használ PHP-reflexiót.
- A `Module-type` értéke `table`, `project` vagy `both`. Válassza a legszűkebb megfelelő hatókört.
- A `$guarded_methods` listában szereplő metódusokhoz bejelentkezett felhasználó szükséges. A `$master_access_only` listában szereplő metódusokhoz mester-hozzáférés kell; az `adminPage` alapértelmezés szerint csak mesterek számára érhető el.
- Az `$ajax_allowed_actions` az örökölt AJAX-kezelést a felsorolt nyilvános műveletekre korlátozza, az `$ajax_deps` pedig az ahhoz betöltendő fájlokat sorolja fel.
- Az opcionális, védett `prepare_module()` és `finish_module()` metódus a sikeres meghívás előtt, illetve után fut le. A `prepare_module()` a hívást `false` visszaadásával megszakíthatja.
- Az `init()` a regisztráció után és egy engedélyezett regisztráció frissítése után fut, ezért idempotensnek kell lennie. Az `uninstall()` (vagy az örökölt `destroy()`) a letiltás előtt fut, és az `['error' => 'message']` visszaadásával megszakíthatja azt. A deklarált függőségek ellenőrzéséhez hívja meg a szülő metódusokat.
- A rendszermodulok a `resources/includes/modules/`, a projektlokális modulok és felülbírálások a `resources/local/includes/modules/` könyvtárba kerülnek.

A kommentezett referenciamegvalósítás a `resources/includes/modules/examples/official_module_template.php` fájl.

### Modul-hozzáférés

Ugyanaz a modul többször is hozzáadható eltérő hozzáférési beállításokkal vagy paraméterekkel. Ez lehetővé teszi az adminisztrátorok számára, hogy különböző konfigurációkat biztosítsanak különböző felhasználókhoz, csoportokhoz vagy táblákhoz.

Például:

- az `allowed_columns` különböző oszlopokat tehet elérhetővé különböző csoportok számára; és
- a `text_filter` táblaspecifikus szűrőoszlopokat biztosíthat egy több adattáblát tartalmazó projektben.

Az **Access** oszlop határozza meg egy modulpéldány általános célközönségét. Az elérhető lehetőségek közé tartozik a nyilvános hozzáférés és a bejelentkezett felhasználókra korlátozott hozzáférés.

A **Group access** oszlop tovább korlátozza a modulpéldányt a kiválasztott projektcsoportokra vagy egyéni felhasználókra.

Ha ugyanannak a modulnak több példánya is vonatkozik egy felhasználóra, tesztelje, hogy a telepített OpenBioMaps-verzió melyik konfigurációt választja ki, illetve mely konfigurációkat egyesíti. Kerülje az egymást átfedő hozzáférési szabályokat, hacsak nem ismert a működésük.

### Modulok engedélyezése és letiltása

Minden konfigurált modulpéldány engedélyezhető vagy letiltható. Egy modul letiltása megőrzi annak konfigurációját, de megakadályozza a használatát.

Egy modul állapotának módosítása után tesztelje az érintett oldalt minden érintett hozzáférési csoport felhasználóival. Egyes modulok adatbázis-objektumokat is létrehoznak, vagy letiltásuk után is megőrzik modulspecifikus beállításaikat.

### Modulfüggőségek

Egy modul más modulokat függőségként jelölhet meg. A függő modul csak akkor engedélyezhető, ha a szükséges modulok már engedélyezve vannak. Egy szükséges modul pedig nem tiltható le, amíg engedélyezett modul függ tőle.

Az alkalmazás ezeket a szabályokat a modulok telepítési és eltávolítási életciklus-hookjaiban ellenőrzi. Sikertelen ellenőrzés esetén a kért állapotváltozás nem történik meg, és a moduladminisztrációs oldal megjeleníti a hibát. A szükséges modul letiltása előtt tiltsa le a tőle függő modulokat.

A modulfejlesztők a modul metaadatainak `Dependencies` listájában deklarálják a függőségeket. A szükséges modulnak a függő modul által igényelt hatókörben elérhetőnek és engedélyezettnek kell lennie.

### Modulok eltávolítása

A moduladminisztrációs felület jelenleg nem biztosít általános lehetőséget egy telepített modul alkalmazásból való eltávolítására.

Egy konfigurált modulpéldány letiltható. Ne törölje kézzel a modul fájljait vagy adatbázis-objektumait, kivéve, ha ismert a modul eltávolítási eljárása, és rendelkezésre áll biztonsági mentés.

### Modulparaméterek

A legtöbb modul közvetlenül a moduladminisztrációs oldalon fogad JSON-paramétereket. Más modulok külön adminisztrációs lapot biztosítanak a modulspecifikus feladatokhoz. A `geometry_management` példa egy saját adminisztrációs felülettel rendelkező modulra.

A dokumentumban szereplő példák olyan helyőrzőket használnak, mint a `YOURTABLE`, a `column_name` és a `schema.table`. Cserélje le ezeket a helyőrzőket a projekt azonosítóira.

## Projektszintű modulok

.. _protocol-installer-guide:

### `protocol_installer`

A `protocol_installer` modul lehetővé teszi, hogy a projektgazdák adatgyűjtési
protokollokat telepítsenek egy beállított nyilvános katalógusból. A modul
adminisztrációs oldala végigvezeti őket a táblák és oszlopok megfeleltetésén,
a listák beállításán, a terv áttekintésén, a telepítésen és a támogatott
frissítéseken.

**Paraméterek:** Nincsenek. A katalógust a szerveradminisztrátor állítja be.

A telepítő egyetlen beállított nyilvános GitLab-tárolóból olvas deklaratív
YAML-csomagokat. Az 1. verzió kötelező előírásait a
:doc:`csomagformátum leírása <protocol_package_format>`
rögzíti. A közzétett protokollokat a
[share-your-protocol tárolóban](https://gitlab.com/openbiomaps/share-your-protocol)
tartják karban.

#### A protokollforrás beállítása

Állítsa be a következő szerverkonstansokat az
`/etc/openbiomaps/system_vars.php.inc` fájlban:

```php
define('PROTOCOL_INSTALLER_GITLAB_PROJECT', 'openbiomaps/share-your-protocol');
define('PROTOCOL_INSTALLER_GITLAB_REF', 'main');
define('PROTOCOL_INSTALLER_GITLAB_CATALOGUE_PATH', 'catalog.yml');
```

A projekt értéke GitLab `csoport/projekt` útvonal, nem URL. A Git-hivatkozás
és a katalógus útvonala opcionális; alapértékük `main`, illetve `catalog.yml`.
A tárolónak nyilvánosnak kell lennie; ez a kiadás nem használ GitLab-hitelesítést.
Engedélyezze a projektben a `protocol_installer` modult a moduladminisztrációban.

#### Protokoll telepítése

1. Projektgazdaként nyissa meg az **Adatgyűjtési protokollok**
   (**Data collection protocols**) oldalt.
   A katalógus jelzi, hogy az egyes protokollok telepítve vannak-e, melyik
   verziójuk van telepítve, és elérhető-e újabb verzió. Válassza a **Telepítés** (**Install**)
   vagy a legújabb verzióra történő **Frissítés** (**Upgrade to**) lehetőséget, majd tekintse át
   a követelményeket. A korábbi katalógusverziók az **Egyéb katalógusverziók**
   (**Other catalogue versions**) alatt érhetők el.
2. Minden logikai táblát rendeljen hozzá egy elérhető projekttáblához, vagy
   hozzon létre új táblát. A `project_main` tábla mindig a nyilvántartott
   főtáblának felel meg. Minden oszlophoz válasszon kompatibilis meglévő
   oszlopot, vagy hozzon létre újat. Ha a terv táblák vagy feltöltési űrlapok
   létrehozását tartalmazza, az ezekhez szükséges jogosultságokkal is
   rendelkeznie kell.
3. Állítsa be a listákat és az űrlapneveket. A rögzített listáknál válasszon
   beágyazott vagy a projekt kifejezéstábláját használó megoldást. Minden
   projektspecifikus listát állítson be `list_definition` JSON-nal vagy meglévő
   kifejezésforrással. Ellenőrizze a webes és API-űrlapok beállításait és
   hozzáférési értékeit.
4. Tekintse át az összes műveletet, ütközést és figyelmeztetést. Oldjon fel
   minden blokkoló ütközést, és a megfeleltetések módosítása után készítse újra
   a tervet. Külön erősítse meg a telepítést. Az eredmény hivatkozásokat ad a
   telepített űrlapokra, és megjeleníti a mentett megfeleltetéseket.

#### Protokoll frissítése

Egy protokoll projektenként egyszer telepíthető. Az ismételt beküldés a meglévő
telepítést adja vissza. Frissítéshez válassza ugyanannak a protokollnak egy újabb
verzióját. A varázsló megjeleníti a telepített verziót, zárolja a meglévő
megfeleltetéseket és helyi beállításokat, és csak új objektumok létrehozását
tervezi. Megerősítés előtt tekintse át az ütközéseket és a műveleteket. A
meglévő űrlapok, oszlopok, listák és rekordok megmaradnak.

A telepített űrlap új mezőkkel bővíthető, ha a korábbi mezői és definíciójának
egyéb tulajdonságai változatlanok maradnak. Az új mezők a meglévő helyi mezők
után kerülnek, megőrizve az űrlap azonosítóját, hozzáférési beállításait és a
mezők jelenlegi sorrendjét. Egy korábbi mező módosítása vagy eltávolítása,
illetve más meglévő csomagelem módosítása továbbra is blokkolja az automatikus
frissítést.

#### Frissítés kézi befejezése

Ütköző frissítésnél a terv felsorolja a kézi feladatokat, és ahol elérhető,
megjeleníti a régi és a célként megadott definíciót. Végezze el a módosításokat
a projektben, ellenőrizze az adatokat és az űrlapokat, rendelje a kézzel
létrehozott táblákat és oszlopokat a meglévő célobjektumokhoz, majd készítse
újra a tervet. Ezután a projektgazda bejelölheti a kézi befejezésről szóló
nyilatkozatot, és választhatja a **Frissítés megjelölése befejezettként**
(**Mark upgrade complete**) műveletet. A szerver ismét ellenőrzi az áttekintett tervet: megköveteli a
céltáblák és céloszlopok meglétét, ellenőrzi az új űrlapokat és az importált
kifejezéseket, és elutasítja a még végrehajtandó létrehozási műveleteket vagy
az egyéb blokkoló ütközéseket.

A nyilatkozat a `manual_upgrade_history` mezőben rögzíti az adminisztrátort,
az időpontot, a verziókat és a tudomásul vett ütközéseket. Frissíti a mentett
csomagverziót és megfeleltetéseket, de a kézzel létrehozott objektumokat nem
jelöli a telepítő által létrehozottként. Az objektumleltárból nem ellenőrizhető
definícióváltozásokért továbbra is az adminisztrátor felel.

#### Frissítés folytatása

Ha egy frissítés megszakad, válassza ki a protokollt, majd a **Rögzített
frissítés folytatása** (**Resume recorded upgrade**) lehetőséget. A helyreállítás a mentett csomagot és
tervet használja akkor is, ha később másik katalógusrevíziót választottak ki.
Fejezze be a rögzített frissítést, mielőtt újabbat indítana, vagy fejlesztői
módban eltávolítaná a protokollt.

#### A protokoll kompatibilitása és biztonsága

A meglévő oszlopok tárolási típusának és kapacitásának kompatibilisnek kell
lennie. A tervező nem alakítja át a meglévő adatokat, és nem módosítja a
meglévő oszlopdefiníciókat. Ellenőrzi a kezelt rendszeroszlopokat, a geometriai
követelményeket, a mezőtípusokat, a listaforrásokat, a metaadatokat és az
űrlapnevek ütközéseit. Más űrlapot vagy metaadatrekordot soha nem ír felül.
Az új fizikai nevek nem ütközhetnek meglévő relációk nevével.

Korlátozott hosszúságú, meglévő `varchar(n)` oszlophoz a csomagban
`data_type: varchar` típust adjon meg, legfeljebb `n` értékű `max_length`
beállítással. A csomag korlátlan hosszúságú `text` oszlopa nem rendelhető
korlátozott `varchar(n)` oszlophoz, mert a későbbi értékek meghaladhatnák a
céloszlop méretkorlátját.

Kifejezések importálásakor ütközést jelent, ha egy kódhoz eltérő címke vagy
tulajdonos tartozik; a megosztott kifejezéseket a telepítő nem módosítja
észrevétlenül. Tervezés előtt ellenőrzi a YAML-t és a hivatkozásokat, majd
végrehajtás előtt ismét ellenőrzi az aktuális objektumleltárt.

#### Telepítési napló és helyreállítás

A GIS- és a metaadat-adatbázis tranzakciói külön véglegesülnek. A
`public.protocol_installations` telepítési rekordja tárolja a protokoll
azonosítóját, verzióját, GitLab-revízióját, ellenőrzőösszegét, a csomag
pillanatképét, a megfeleltetéseket, a tervet, a fázisok állapotát, a létrehozott
objektumokat és az esetleges hibát. Az űrlapok csak a metaadatfázis befejezése
után válnak aktívvá.

Ha a telepítés megszakad, ellenőrizze a protokollhoz és projekthez tartozó
`status`, `gis_status`, `metadata_status` és `error` mezőket. Nyissa meg újra
ugyanazt a csomagot a varázslóban, és válassza a **Rögzített telepítés
folytatása** (**Resume recorded installation**) lehetőséget. A helyreállítás a mentett tervet és kizárólag a
telepítéshez tartozó objektumokat használja. Tartsa elérhetően az eredeti
csomagrevíziót a telepítés befejezéséig; a részlegesen létrehozott objektumokat
ne távolítsa el kézzel. Ha a csomag azonosítója vagy verziója eltér, válassza
a rögzített verziót.

#### Protokoll eltávolítása és az 1. verzió korlátai

Fejlesztői módban a rögzített telepítés eltávolítása előnézettel és
megerősítéssel végezhető el. A művelet eltávolítja a telepítés által
létrehozott táblákat, oszlopokat, űrlapokat, importált kifejezéseket, a
naplóbejegyzést és a GIS-adatbázis tulajdonjelölőit. Az újrahasznált
projektobjektumok megmaradnak. Más űrlapok függőségei blokkolják az eltávolítást.
Az éles környezetben történő eltávolítás és a nem kizárólag bővítést végző
automatikus protokollfrissítés későbbi fejlesztés tárgya. Az általános
adatimport, a csomagokhoz tartozó hookok, a függőségek és a szabványosított
exportok nem részei az 1. verziónak.

#### Protokollcsomag készítése és közzététele

1. Adjon a protokollnak stabil, fordított DNS-jelölésű `protocol_id`
   azonosítót és `protocol_version` verziót. Tartsa változatlanul a táblák,
   oszlopok, űrlapok és listák logikai azonosítóit. A javasolt fizikai nevek
   csak ajánlások; a hivatkozásokban logikai azonosítókat kell használni.
   A közzétett frissítéseknek meg kell őrizniük a meglévő definíciókat, az új
   elemeket pedig új logikai azonosítókkal kell hozzáadniuk. A meglévő űrlapok
   és listadefiníciók megmaradnak.
2. Minden táblát `project_main` vagy `create` típussal adjon meg a hordozható
   oszlopaival, listáival, valamint webes és API-űrlapjaival együtt. Az
   alkalmazás által kezelt mezőkhöz használja a `system.geometry` és
   `system.files` azonosítókat. Csak a
   :doc:`formátumleírásban <protocol_package_format>`
   felsorolt típusokat és beállításokat használja.
3. Tudatosan válassza meg a listák kezelését. A `fixed` lista beágyazva
   maradhat, vagy importálható a projekt kifejezéstáblájába. A tárolt érték
   a `code`, a webes és API-űrlapokon megjelenő szöveg a `label`. A `project`
   listát telepítéskor helyben kell beállítani. A `table` lista egy deklarált
   tábla oszlopaira hivatkozik. A csomag nem tartalmazhat PHP-t, SQL-t,
   hitelesítő adatokat vagy fizikai projektazonosítókat.
4. Adja hozzá a YAML útvonalát az ugyanabban a GitLab-tárolóban lévő
   `catalog.yml` fájlhoz. A katalógusbejegyzés azonosítójának és verziójának
   egyeznie kell a csomagéval. Az útvonalak a tároló gyökeréhez képest
   relatívak, és nem léphetnek ki abból. Véglegesítse mindkét fájlt egy
   commitban, és ellenőrizze, hogy a kiválasztott ág tartalmazza őket.
   A telepítő rögzíti ezt a commitot, és a telepítési naplóba menti a csomag
   ellenőrzőösszegét.

#### Fejlesztői regressziós ellenőrzések

Az adatbázistesztek előtt futtassa a
`tools/sync-sablon-from-resources.sh --apply` parancsot a web-app tárolóban a
tesztadatok frissítéséhez; a `projects/sablon` könyvtárat ne szerkessze kézzel.
Futtassa a csomagokat, tervezést, objektumleltárt és végrehajtást ellenőrző
célzott PHPUnit-teszteket a Docker-tesztkörnyezetben. A böngészős alaptesztek
a nyilvános és projektútvonalakat ellenőrzik. Teljes próbaüzemhez telepítsen
protokollokat egy új, erre kijelölt projektbe és egy meglévő rekordokat
tartalmazó projektbe, majd nyisson meg egy webes űrlapot, és kérjen le egy
API-űrlapot a mobilklienssel. Ellenőrizze, hogy a rögzített lista a kódját
tárolja mind a beágyazott, mind a kifejezéstáblát használó változatban.

### `classification`

A `classification` modul a projekt kontrollált szótárait (osztályozási sémáit) kezeli. Egy séma többnyelvű címkékkel és állandó azonosítókkal (UUID) rendelkező fogalmakat tartalmaz. A sémák lehetnek laposak vagy hierarchikusak. Egy fogalom több sémához is tartozhat, sémán belül lehetnek szélesebb (szülő) fogalmai, és kapcsolódhat rokon fogalmakhoz. A fogalmak és a tagságok archiválhatók és visszaállíthatók, a használaton kívüli fogalmak pedig törölhetők.

A modulnak saját telepítése és adminisztrációs oldala van, és nem függ a `geometry_management` modultól. A modul engedélyezése telepíti vagy ellenőrzi az osztályozási sémát a GIS-adatbázisban (a `system.classification_installation` táblában rögzítve), a meglévő adatok újralétrehozása vagy újraosztályozása nélkül. Az inicializálás idempotens: ha meghiúsul, javítsa a problémát, és engedélyezze újra a modult. A telepítés visszautasítja az inkompatibilis meglévő sémát ahelyett, hogy módosítaná.

A szótárkezelés projektmesterek számára korlátozott. Az adminisztrációs oldal lehetőségei:

- sémák létrehozása és szerkesztése;
- fogalmak létrehozása, címkéik szerkesztése, és meglévő fogalom hozzáadása egy másik sémához;
- szélesebb és rokon fogalmak hozzáadása és eltávolítása;
- fogalmak és tagságok archiválása, visszaállítása, valamint a használaton kívüliek törlése; és
- fogalmak keresése és az eredmények lapozása.

A modul letiltását akadályozza, ha egy aktív vagy közzétett webes, fájlos vagy API-űrlap osztályozási mezőt tartalmaz; a hibaüzenet felsorolja az érintett űrlapokat. A letiltás megőrzi az összes osztályozási adatot, az újbóli engedélyezés pedig újra elérhetővé teszi azokat. Automatikus eltávolító szkript nincs, a korábbi verzióra való visszalépéshez átvizsgált biztonsági mentés szükséges.

**Paraméterek:** Nincsenek.

**Kiadási állapot:** ez a kiadás a szótárkezelést tartalmazza. Az osztályozások hozzárendelése mintavételi egységekhez és megfigyelésekhez, a szótárak CSV-importja, az osztályozási mezők a feltöltési űrlapokban, valamint a szövegszűrők tervezettek, és még nem érhetők el.

### `geometry_management`

A `geometry_management` modul újrafelhasználható térbeli alakzatokat kezel. A felhasználók pontokat, vonalakat és poligonokat tölthetnek fel, megoszthatják őket más felhasználókkal, valamint webes vagy fájlból történő feltöltéskor rekordgeometriaként használhatják őket. Az elérhető feltöltési geometriákat a mobilalkalmazás is letöltheti és megjelenítheti.

Az újonnan feltöltött alakzatok alapértelmezés szerint nem láthatók más felhasználók számára. A projektadminisztrátorok jogosultságot adhatnak az egyes alakzatok lekérdezésekhez vagy adatfeltöltésekhez való használatára. A felhasználók az alakzatokat a profiljuk **Megosztott poligonok** oldalán kezelik. Az oldalon **Saját geometriák** és **Közösségi nyilvános geometriák** lap, keresőmező, a használat (lekérdezés vagy feltöltés) és a geometriatípus szerinti szűrők, GeoJSON-letöltés, kijelölés alapú törlés, valamint új térbeli fájlok feltöltésére szolgáló űrlap található. A projektadminisztrátorok a projekt összes geometriáját az **Összes poligon** adminisztrációs lapon látják, és az egyes alakzatokat kijelölt projektfelhasználókhoz rendelhetik.

A modul a `classification` modultól függetlenül működik.

Ha egy feltöltési űrlap `obm_geometry` mezőt használ, annak térképvezérlője felkínálhatja a **Geometria listából** lehetőséget. Egy elnevezett alakzat kiválasztása beilleszti annak WKT-geometriáját a feltöltési mezőbe.

**Paraméterek:** Nincsenek. A modul a saját adminisztrációs felületét használja.

### `photos`

A `photos` modul engedélyezi a fénykép- és egyéb csatolmán mezőket a feltöltési űrlapokon, és megjeleníti a csatolt képeket a rekordok adatlapjain.

A fájlméretkorlátokat, az engedélyezett fájltípusokat, a tárolást, a hozzáférés-vezérlést és a biztonsági mentési követelményeket az alkalmazás és a kiszolgáló szintjén is konfigurálni kell.

**Paraméterek:** Nincsenek.

### `create_pg_user`

A `create_pg_user` modul lehetővé teszi a jogosult felhasználók számára személyes PostgreSQL-fiókok létrehozását.

A modul engedélyezésekor:

- egy **PostgreSQL-felhasználó létrehozása** mező jelenik meg a jogosult felhasználók profiloldalán;
- a felhasználók létrehozhatják és megújíthatják saját adatbázis-fiókjukat;
- a létrehozott fiók hozzá lesz rendelve a projekt PostgreSQL-felhasználói csoportjához; valamint
- a fiók adatbázis-kliensekkel, például QGIS-szel használható.

Alapértelmezés szerint a létrehozott fiók:

- olvasási hozzáféréssel rendelkezik a projekt adatbázistábláihoz;
- egyidejűleg egy klienskapcsolatra korlátozott; és
- egy év után lejár.

A létrehozott fiók hozzá lesz adva a projektről elnevezett PostgreSQL-csoporthoz, amelynek formája általában `PROJECT_user`. Egy adatbázis-adminisztrátor további jogosultságokat adhat, például írási hozzáférést bizonyos táblákhoz, de a legkisebb jogosultság elvét kell követnie.

A felhasználók a lejárat előtt vagy után megújíthatják hozzáférésüket, a telepített modul szabályainak megfelelően.

A következő képernyőkép egy példa PostgreSQL/PostGIS-kapcsolatot mutat be QGIS-ben:

![OpenBioMaps PostGIS-kapcsolat hozzáadása QGIS-ben](images/qgis_connect.jpg)

Ne tegye elérhetővé a PostgreSQL-t a nyilvános interneten megfelelő tűzfal-, TLS-, hitelesítési és hozzáférés-vezérlési beállítások nélkül.

**Paraméterek:** Nincsenek. A jelenlegi kiadások külön adminisztrációs oldalt biztosíthatnak.

### `computation`

A `computation` modul projektspecifikus számítási funkciókat biztosít.

Pontos működése a telepített modulverziótól és a projekt konfigurációjától függ. Éles projektben való engedélyezés előtt vizsgálja felül a modul megvalósítását.

**Paraméterek:** Nincsenek dokumentálva.

### `custom_filetype`

A `custom_filetype` modul támogatja projektspecifikus egyéni letöltési formátumok, például Observado-stílusú CSV-fájlok előállítását.

A kimeneti formátum és minden szükséges egyéni megvalósítás a projekttől függ.

**Paraméterek:** Nincsenek dokumentálva.

### `taxon_meta`

A `taxon_meta` modul taxonokkal kapcsolatos metaadat-funkciókat biztosít.

Felhasználói felületét, szükséges adatbázis-struktúráját és konfigurációját a telepített modulverzió alapján kell ellenőrizni.

**Paraméterek:** Nincsenek dokumentálva.

### `turnstile`

A `turnstile` modul a Cloudflare Turnstile szolgáltatást integrálja a regisztrációs és a meghívási űrlapokba. Beállított állapotban a regisztrációs űrlap a régi képes captcha helyett a Turnstile-widgetet jeleníti meg, a kiszolgáló pedig a Turnstile API-n ellenőrzi a beküldött tokent. Ha a modul nincs beállítva, a régi captcha marad tartalék megoldásként.

A modul beállítása nem modulparaméterekkel, hanem a projektkonfigurációban (`local_vars.php.inc`) megadott állandókkal történik:

```php
define('CLOUDFLARE_TURNSTILE_SITE_KEY', 'your-site-key');
define('CLOUDFLARE_TURNSTILE_SECRET_KEY', 'your-secret-key');
```

A modul csak akkor engedélyezett, ha a `CLOUDFLARE_TURNSTILE_SECRET_KEY` nem üres. Mindig adja meg mindkét állandót: ha a webhelykulcs hiányzik, a modul a titkos kulcsot használja webhelykulcsként. A widget szkriptje a `https://challenges.cloudflare.com/turnstile/v0/api.js` címről töltődik be, az ellenőrző kérést pedig a kiszolgáló küldi a Cloudflare-nek, ezért a kiszolgálónak kimenő HTTPS-hozzáférésre van szüksége.

**Paraméterek:** Nincsenek.

### `cms`

A `cms` modul **Tartalom** csoportot ad a profilmenühöz **Oldalak** és **Bejegyzések** menüponttal. Célja a projektirányítópult és a Nextcloud API összekapcsolása. Az adminisztrációs oldal csak projektmesterek számára érhető el.

A modul fejlesztés alatt áll (0.0.1-es verzió), adminisztrációs oldala még nem kínál beállításokat.

**Paraméterek:** Nincsenek.

### `ioc_bird_list`

Az `ioc_bird_list` modul az IOC World Bird List madárlistát importálja a megosztott adatbázissémába. A projekt adminisztrációs oldalán lévő frissítőgomb letölti a törzslista XML-fájlját a `www.worldbirdnames.org` címről, létrehoz egy `shared.ioc_<verzió>` nevű táblát (a verzióban a pontok aláhúzásjelre cserélődnek), és feltölti a listával. A meglévő `shared.ioc_*` táblákat az adminisztrációs oldal listázza.

Az adminisztrációs oldal csak projektmesterek számára érhető el. A lista letöltéséhez a kiszolgálónak kimenő HTTP-hozzáférésre van szüksége.

**Paraméterek:** Nincsenek.

### `natura2000`

A `natura2000` modul Natura 2000 validációs felületet biztosít a felhasználói profilban. A projekt megfigyeléseit összeveti a `natura2000` adatbázissémában tárolt fajlistákkal és területekkel (például a `natura2000.sites`, a `natura2000.mdb_species` és a `natura2000.natura2000_fajok` táblával), és tartalmaz egy oldalt a kapcsolódó dokumentumok kezelésére.

A modul ettől a sémától, valamint magyar terület- és fajadatoktól függ, ezért csak az erre előkészített projektekben használható. Felülete és üzenetei jelenleg csak magyarul érhetők el. A profiloldal a Tabulator könyvtárat az `unpkg.com` címről tölti be.

**Paraméterek:** Nincsenek dokumentálva.

### `pds_test_project_installer`

A `pds_test_project_installer` modul átmeneti belépési pont azokhoz a projektekhez, amelyekben a régi PDS-tesztinstaller engedélyezve van. Már nem telepít és nem módosít projektadatokat. Az új PDS-tesztkatalógusokat a `protocol_installer` modullal kell telepíteni a PDS mobilűrlap-tesztkatalógus kiválasztásával. A modul letiltása érintetlenül hagyja a meglévő PDS-telepítéseket.

A katalógus telepítése után engedélyezze a `pds_test_evaluator` modult a modulok adminisztrációjában.

**Paraméterek:** Nincsenek.

## Táblaszintű modulok

### `box_load_selection`

A `box_load_selection` modul az újrafelhasználható geometriákat térképi térbeli lekérdezésekhez használja. Projekttáblánként külön engedélyezhető, paraméterezhető és hozzáféréshez köthető. A modul a projektszintű `geometry_management` modultól függ; először a `geometry_management` modult kell engedélyezni.

Ha az aktuális táblán engedélyezve van, egy **Térbeli lekérdezés** mező jelenik meg a térképoldalon. A felhasználók kiválaszthatnak egy elérhető projektgeometriát, és térbeli lekérdezést futtathatnak rajta. Poligongeometriák esetén a felület lehetővé teheti annak kiválasztását, hogy a poligon határát metsző rekordok szerepeljenek-e a lekérdezésben.

**Paraméterek:** Egy opcionális JSON-lista korlátozhatja az elérhető térbeli kapcsolatokat. Ha nincs megadva, a modul – ahol alkalmazható – a `contains`, `intersects`, `crosses` és `disjoint` kapcsolatokat kínálja fel.

### `additional_columns`

Az `additional_columns` modul több adattáblában található rekordok összekapcsolásához használt oszlopokat határoz meg.

Ha a táblák közös azonosítóval kapcsolódnak egymáshoz, a lekérdezések az adott azonosítóhoz tartozó kapcsolódó rekordokat is tartalmazhatják. A felhasználók a térképoldalon található **Táblakapcsolások figyelmen kívül hagyása** lehetőség kiválasztásával megkerülhetik ezeket a kapcsolásokat.

Egy projekt például külön táblákban tárolhatja a szülő- és utódrekordokat, és közös kotorékazonosítót használhat kapcsolóoszlopként.

Ezt a modult a `join_tables` modullal együtt használja.

A modul a következőket adja vissza:

- oszlopok tömbjét a `0` indexen; és
- oszlopnevek asszociatív tömbjét az `1` indexen.

**Paraméterek:**

```json
[
  "column_name_1",
  "column_name_2"
]
```

### `allowed_columns`

Az `allowed_columns` modul oszlopszintű korlátozásokkal egészíti ki a sorszintű adathozzáférési szabályokat.

A sorszintű szabályok határozzák meg, hogy a felhasználó mely rekordokhoz férhet hozzá. Ez a modul határozza meg, hogy mely oszlopok maradnak láthatók, amikor egy rekord `restricted` vagy `no-geom` szabály hatálya alá esik, illetve amikor nincs egyező szabály.

A modul olyan projektekhez készült, amelyek alapszintű hozzáférése nem nyilvános, és amelyek adattáblái a megfelelő szabálytáblát használják.

**Paraméterek:**

```json
{
  "for_sensitive_data": [
    "column_visible_for_sensitive_records"
  ],
  "for_no-geom_data": [
    "column_visible_for_records_without_geometry_access"
  ],
  "for_general": [
    "column_visible_when_no_rule_matches"
  ]
}
```

A paraméterek jelentése:

- A `for_sensitive_data` az érzékeny rekordok esetében látható oszlopokat sorolja fel.
- A `for_no-geom_data` a `no-geom` rekordok esetében látható oszlopokat sorolja fel. Ha ez a kulcs hiányzik, azoknál a rekordoknál minden oszlop hozzáférhető.
- A `for_general` azokat az oszlopokat sorolja fel, amelyek akkor láthatók, ha egyetlen szabály sem illeszkedik. Ha ez a kulcs hiányzik, ebben az esetben minden oszlop korlátozott.

Tesztelje a tényleges jogosultságokat nyilvános, hitelesített, csoporttag és adminisztrátori fiókokkal. Az oszlopkorlátozások nem helyettesítik a megfelelő adatbázis- és API-hozzáférés-vezérlést.

### `bold_yellow`

A `bold_yellow` modul azonosítja az eredmény-összefoglalók fontos mezőit.

A konfigurált oszlopok félkövér sárga kiemeléssel jelennek meg a részletes eredménylistákban. A mobilalkalmazás ezt a konfigurációt használja az **Összegyűjtött adatok** összefoglaló címkéiben megjelenő értékek kiválasztására is.

**Paraméterek:**

```json
[
  "column_name_1",
  "column_name_2"
]
```

### `box_load_coord`

A `box_load_coord` modul egy **Pozíció** blokkot ad a térkép alá.

A blokk:

- megjeleníti a mutató aktuális helyének koordinátáit; és
- lehetővé teszi a felhasználó számára szélességi és hosszúsági értékek megadását, valamint a megfelelő pont térképen való elhelyezését.

A paraméterek a koordináta-rendszerek felhasználóknak megjelenő neveit EPSG-kódokhoz rendelik.

**Paraméterek:**

```json
{
  "wgs84": "4326",
  "eov": "23700"
}
```

Csak a projekt és térképi összetevői által támogatott koordináta-rendszereket konfiguráljon.

### `box_load_last_data`

A `box_load_last_data` modul egy **Gyors lekérdezések** mezőt ad a térképoldalhoz.

A következő lekérdezéseket biztosítja:

- az aktuális felhasználó legutóbbi feltöltése;
- bármely felhasználó legutóbbi feltöltése; és
- a legutóbb feltöltött rekordok.

Az első két lehetőség egy rekordot ad vissza. A paraméter a harmadik lehetőség által visszaadott rekordok számát szabályozza. A dokumentált alapértelmezett érték 10.

**Paraméterek:**

```json
[
  10
]
```

### `box_custom`

A `box_custom` modul egy projektspecifikus egyéni mezőt tölt be a térképoldalon.

Az egyéni megvalósítást a projekt `local/includes/modules/` könyvtárában kell elhelyezni. Az osztályának legalább a `print_box()` és `print_js()` metódusokat biztosítania kell.

A következő helyen tárolt egyéni modul esetében:

`local/includes/modules/hrsz_query.php`

a paraméter a fájl alapnevét tartalmazza:

```json
[
  "hrsz_query"
]
```

A megfelelő osztály elvárt neve `hrsz_query_Class`.

Az egyéni modulkódnak ellenőriznie kell a bemeneteket, escape-elnie kell a kimenetet, érvényesítenie kell a jogosultságokat, és paraméterezett adatbázis-lekérdezéseket kell használnia.

### `identify_point`

Az `identify_point` modul lehetővé teszi a felhasználók számára egy vagy több pont azonosítását a térképen, és megjeleníti a kiválasztott attribútumértékeket egy térképi felugró ablakban.

**Paraméterek:**

```json
[
  "column_name_1",
  "column_name_2"
]
```

Csak olyan oszlopokat adjon meg, amelyekhez a modul célközönsége jogosult hozzáférni.

### `cameratrap_api`

A `cameratrap_api` modul kommunikációt biztosít egy kameracsapda-irányítópult és a Nextcloud API között.

Funkciói a következők:

- kamerák és elemzések kezelése;
- képek fel- és letöltése;
- elemzések indítása; és
- az integrációhoz szükséges Nextcloud-hitelesítő adatok kezelése.

A modul modulspecifikus adatbázis-objektumokat hoz létre vagy használ. Engedélyezése előtt vizsgálja felül az SQL-telepítőfájlját és hozzáférési követelményeit.

**Paraméterek:** Nincsenek dokumentálva.

### `nextcloud_connect`

A `nextcloud_connect` modul összekapcsolja az OpenBioMaps rendszert egy Nextcloud-kiszolgálóval. Felhasználóiprofil-integrációt biztosít, és JWT-tokeneket bocsát ki hitelesítéshez.

A Nextcloud URL-jeit, hitelesítő adatait, aláírási titkait, tokenjeinek élettartamát és TLS-ellenőrzését biztonságosan, a telepített kiadás által elvárt mechanizmusokon keresztül kell konfigurálni.

**Paraméterek:** Nincsenek dokumentálva.

### `validation`

A `validation` modul belső API-t és adminisztrációs felületet biztosít az adatérvényesítési algoritmusokhoz.

Funkciói a következők:

- érvényesítési szabályok kezelése;
- rekordok érvényesítése; és
- érvényesítési műveletek naplózása.

A projektspecifikus érvényesítési megvalósítások további ellenőrzéseket végezhetnek a feltöltött adatokon.

**Paraméterek:** Nincsenek dokumentálva. A további szabályok a modul adminisztrációs felületén és az érvényesítési összetevőkön keresztül kezelhetők.

### `download_restricted`

A `download_restricted` modul adminisztrátor által felügyelt letöltés-engedélyezési munkafolyamatot vezet be.

Az azonnali letöltési hozzáférés helyett a felhasználók kérelmet nyújtanak be, amelyben leírják az adatok tervezett felhasználását. Az adminisztrátorok a modul adminisztrációs felületén jóváhagyhatják vagy elutasíthatják a kérelmet.

A modul a következőket biztosítja:

- letöltési kérelem űrlapja;
- adminisztrátori jóváhagyási munkafolyamat; és
- integráció a `results_buttons` modullal.

A `results_buttons` modullal együtt használva az exportálási lehetőségek csak azoknak a felhasználóknak érhetők el, akiknek kérelme és jogosultságai engedélyezik a letöltést.

A modul engedélyezése nem szünteti meg a kiszolgálóoldali hozzáférés-ellenőrzések szükségességét. Tesztelje a közvetlen exportálási URL-eket és API-kat annak ellenőrzésére, hogy a letöltési korlátozások nem kerülhetők-e meg.

**Paraméterek:** Nincsenek. A modul a saját adminisztrációs felületét használja.

### `extra_params`

Az `extra_params` bővítmény további bemeneti paramétereket biztosít az űrlapokhoz.

A bővítmény pontos szintaxisát és elérhetőségét a telepített OpenBioMaps-kiadás alapján kell ellenőrizni, mivel nem minden verzióban található meg ilyen nevű önálló modul.

**Paraméterek:** Itt nincs dokumentálva stabil paraméterformátum.

### `grid_view`

A `grid_view` modul alternatív poligonrácsok használatával jeleníti meg az adatokat. Ilyenek például az UTM-rácsok, a KEF-rácsok, az illesztett pontok és a dinamikusan létrehozott rácspoligonok.

Amikor egy rácsnézet aktív, a modul által szolgáltatott geometria helyettesíti a rekord eredeti geometriáját az adott megjelenítésben.

A modul megvalósítása többek között a következő metódusokat teszi elérhetővé:

- `print_box()`;
- `default_grid_geom()`; és
- `get_grid_layer()`.

#### Paraméterek

```json
{
  "layer_options": [
    "kef_5 (layer_data_grid)",
    "original (layer_data_points)"
  ]
}
```

Minden `layer_options` bejegyzés egy geometriaoszlopot társít egy MapServer-réteghez:

- a zárójel előtti szöveg a `YOURTABLE_qgrids` egyik oszlopa; és
- a zárójelen belüli szöveg a megfelelő MapServer-réteg neve.

A példában:

- a `kef_5` a `YOURTABLE_qgrids` geometriaoszlopa;
- a `layer_data_grid` a megjelenítéséhez használt MapServer-poligonréteg;
- az `original` a forrásgeometriát tárolja; és
- a `layer_data_points` az eredeti pontokat jeleníti meg.

Egy rácsgeometriához kompatibilis MapServer-réteg szükséges. A `layer_data_grid` rétegnek például poligonrétegnek kell lennie, ha poligonrácsokat jelenít meg.

#### Rácstábla

A modul létrehozza a `YOURTABLE_qgrids` táblát, ha még nem létezik. A tábla ezután kibővíthető a projekthez szükséges geometriaoszlopokkal.

A modul létrehozhat egy `update_grid_geoms` triggert és kezdeti oszlopmegjegyzéseket is. Ezek a létrehozott objektumok általában projektspecifikus felülvizsgálatot és módosítást igényelnek.

Állítsa be a rácsbeállítások felhasználóknak megjelenő neveit oszlopmegjegyzésként:

```sql
COMMENT ON COLUMN public.YOURTABLE_qgrids.original IS 'Original';
COMMENT ON COLUMN public.YOURTABLE_qgrids.kef_5 IS 'KEF 5×5';
```

Az SQL-azonosítókat következetesen használja. Ne konfiguráljon például `kef_5` értéket a modulban, ha az oszlopot `kef5` néven hozza létre.

#### Trigger a rácstáblán

A következő példa egy projektspecifikus rácsfrissítő függvényt hív meg:

```sql
CREATE TRIGGER update_grid_geoms
BEFORE INSERT OR UPDATE ON public.YOURTABLE_qgrids
FOR EACH ROW
EXECUTE PROCEDURE public.update_qgrid_geoms_arg(
  '0.1',
  '0.1',
  't',
  't',
  't',
  't',
  '0.05'
);
```

> **Fontos:** A trigger argumentumainak száma és sorrendje pontosan meg kell, hogy egyezzen az `update_qgrid_geoms_arg()` telepített definíciójával. Az alábbi korábbi példafüggvény a `TG_ARGV[8]` indexig olvas argumentumokat, míg a fenti példatrigger csak hét argumentumot ad át. Ne telepítse ezeket a példákat változtatás nélkül. Vizsgálja meg a telepített `grid_view.sql` fájlt és az adatbázis-függvényt, majd adjon meg minden szükséges argumentumot.

#### Trigger a forrástáblán

A forrástáblához olyan trigger szükséges, amely átmásolja a változásokat a rácstáblába:

```sql
CREATE TRIGGER qgrids
BEFORE INSERT OR DELETE OR UPDATE ON public.YOURTABLE
FOR EACH ROW
EXECUTE PROCEDURE insert_originalgeom_into_qgrids();
```

Egy példafüggvény törzse:

```sql
BEGIN
  IF TG_OP = 'INSERT' THEN
    EXECUTE format(
      'INSERT INTO %I_qgrids (row_id, original) SELECT %L, %L::geometry',
      TG_TABLE_NAME,
      NEW.obm_id,
      NEW.obm_geometry
    );
    RETURN NEW;
  END IF;

  IF TG_OP = 'UPDATE' THEN
    EXECUTE format(
      'UPDATE %I_qgrids SET original = %L::geometry WHERE row_id = %L',
      TG_TABLE_NAME,
      NEW.obm_geometry,
      NEW.obm_id
    );
    RETURN NEW;
  END IF;

  IF TG_OP = 'DELETE' THEN
    EXECUTE format(
      'DELETE FROM %I_qgrids WHERE row_id = %L',
      TG_TABLE_NAME,
      OLD.obm_id
    );
    RETURN OLD;
  END IF;

  RETURN NULL;
END;
```

Ez csak egy triggerfüggvény törzse, nem teljes `CREATE FUNCTION` utasítás.

#### Rácsfrissítő függvény

A következő korábbi példa a tervezett műveleteket szemlélteti:

```sql
DECLARE
  snap_x numeric := TG_ARGV[0];
  snap_y numeric := TG_ARGV[1];
  kef5 boolean := TG_ARGV[2];
  utm10 boolean := TG_ARGV[5];
  snap boolean := TG_ARGV[6];
  snap_polygon boolean := TG_ARGV[7];
  snap_polygon_size numeric := TG_ARGV[8];
BEGIN
  IF TG_OP = 'UPDATE' THEN
    IF kef5 THEN
      EXECUTE format(
        'SELECT geometry FROM shared."kef_5x5" WHERE ST_Within(%L::geometry, geometry)',
        NEW.original
      )
      INTO NEW.kef_5;
    END IF;

    IF snap THEN
      EXECUTE format(
        'SELECT ST_SnapToGrid(%L::geometry, %L, %L)',
        NEW.original,
        snap_x,
        snap_y
      )
      INTO NEW.snap;
    END IF;

    IF snap_polygon THEN
      EXECUTE format(
        'SELECT ST_Expand(ST_SnapToGrid(%L::geometry, %L, %L), %L)',
        NEW.original,
        snap_x,
        snap_y,
        snap_polygon_size
      )
      INTO NEW.snap_polygon;
    END IF;

    RETURN NEW;
  END IF;

  IF TG_OP = 'INSERT' THEN
    IF kef5 THEN
      EXECUTE format(
        'SELECT geometry FROM shared."kef_5x5" WHERE ST_Within(%L::geometry, geometry)',
        NEW.original
      )
      INTO NEW.kef_5;
    END IF;

    IF snap THEN
      EXECUTE format(
        'SELECT ST_SnapToGrid(%L::geometry, %L, %L)',
        NEW.original,
        snap_x,
        snap_y
      )
      INTO NEW.snap;
    END IF;

    IF snap_polygon THEN
      EXECUTE format(
        'SELECT ST_Expand(ST_SnapToGrid(%L::geometry, %L, %L), %L)',
        NEW.original,
        snap_x,
        snap_y,
        snap_polygon_size
      )
      INTO NEW.snap_polygon;
    END IF;

    RETURN NEW;
  END IF;

  RETURN NEW;
END;
```

Ez szintén csak egy függvénytörzs. Az `utm10` változó deklarálva van, de a bemutatott megvalósítás nem használja. Vizsgálja felül és egészítse ki a függvényt a projekt által igényelt rácstípusokhoz.

#### Kezdeti feltöltés

Miután a rácstábla és a triggerek elkészültek, a meglévő forrásgeometriák átmásolhatók egy üres rácstáblába:

```sql
INSERT INTO YOURTABLE_qgrids (row_id, original)
SELECT obm_id, obm_geometry
FROM YOURTABLE;
```

Példa egy illesztett geometria frissítésére:

```sql
UPDATE YOURTABLE_qgrids AS q
SET snap = source.snapped_geometry
FROM (
  SELECT
    obm_id,
    ST_SnapToGrid(obm_geometry, 0.13, 0.09) AS snapped_geometry
  FROM YOURTABLE
) AS source
WHERE q.row_id = source.obm_id;
```

Példa egy megosztott rácstáblából származó poligonok használatával végzett frissítésre:

```sql
UPDATE YOURTABLE_qgrids AS q
SET kef_5 = source.grid_geometry
FROM (
  SELECT
    data.obm_id,
    grid.obm_geometry AS grid_geometry
  FROM YOURTABLE AS data
  LEFT JOIN shared.kef_5x5 AS grid
    ON ST_Within(data.obm_geometry, grid.obm_geometry)
) AS source
WHERE q.row_id = source.obm_id;
```

Ebben a példában a `shared.kef_5x5` tartalmazza az előre meghatározott rácspoligonokat. Más geometria, például a `snap`, dinamikusan is létrehozható.

A sémamódosításokat és tömeges frissítéseket először tesztkörnyezetben futtassa. Készítsen biztonsági mentést az adatbázisról, ellenőrizze a térbeli indexeket, valamint a null, érvénytelen, határvonalon fekvő és nem pont típusú geometriák kezelését.

### `job_manager` érvényesítési feladatok

Az érvényesítési feladatkezelő háttérfolyamatokat konfigurál egy projekthez.

Adminisztrációs oldalán az adminisztrátorok a következőket konfigurálhatják:

- egyszerűsített ütemezés perc-, óra- és napértékekkel; valamint
- feladatspecifikus paraméterek JSON formátumban.

Egy feladat hozzáadása regisztrálja azt a projekt feladattáblájában, és sablonfájlokat hozhat létre az érvényesítési modul és a feladatok könyvtáraiban.

Ennek az összetevőnek az elérhetősége és pontos neve a telepített érvényesítési modultól függhet. A háttérfeladatok csak akkor futnak, ha a projekt `jobs.php` futtatója ütemezve van a kiszolgálón.

**Paraméterek:** Háttérfeladat-nevek listája.

#### `observation_lists`

Az `observation_lists` feladat a mobilalkalmazás által feltöltött megfigyelési listákat dolgozza fel.

A feltöltött megfigyelések kezdetben egy ideiglenes táblába kerülnek. A feladat:

- kitölti az `obm_observation_list_id` értékét;
- kiszámítja vagy átmásolja a lista kezdetének, végének és időtartamának értékeit; valamint
- a teljes listákat átmásolja a céltáblájukba.

A hiányos listákat kihagyja, hogy később dolgozza fel őket.

Feladatparaméterek:

- `list_start_column`: a lista kezdetét tároló oszlop;
- `list_end_column`: a lista végét tároló oszlop;
- `list_duration_column`: az időtartamot tároló oszlop;
- `only_time`: csak az időt tárolja-e a teljes időbélyeg helyett;
- `time_as_int`: percekre alakítsa-e az időt vagy az időtartamot.

Példa:

```json
{
  "YOURTABLE": {
    "list_start_column": "time_of_start",
    "list_end_column": "time_of_end",
    "list_duration_column": "duration",
    "only_time": true,
    "time_as_int": true
  }
}
```

#### `incomplete_observation_lists`

Az `incomplete_observation_lists` feladat a hiányosan maradó listákat kezeli.

Ha a várt és a beérkezett megfigyelések száma közötti különbség a beállított tűréshatáron belül van, a listát a következő `observation_lists` futás feldolgozhatja, és a rendszer üzenetet küld.

Ha a különbség meghaladja a tűréshatárt, a feladat rendszerüzenetet küld, de a listát kézi feldolgozásra hagyja.

Feladatparaméterek:

- `mail_to`: annak a szerepkörnek a numerikus azonosítója, amelynek tagjai megkapják az üzenetet;
- `diff_tolerance`: a megengedett különbség, amely felett kézi feldolgozás szükséges;
- `days_offset`: ennyi napot kell várni a hiányos lista feldolgozása előtt.

Példa:

```json
{
  "YOURTABLE": {
    "mail_to": 1265,
    "diff_tolerance": 2,
    "days_offset": 2
  }
}
```

Tesztelje az értesítések címzettjeit és a feladatok ütemezését, mielőtt éles környezetben erre a munkafolyamatra támaszkodna.

### `join_tables`

A `join_tables` modul kapcsolódó rekordokat jelenít meg egy adatlapon.

A jelenleg dokumentált megvalósítás egyszerű `LEFT JOIN` műveleteket támogat, kapcsolt táblánként egy egyenlőségi feltétellel.

**Paraméterek:**

```json
[
  {
    "table": "events",
    "join_on": [
      {
        "ref_field": "obm_id",
        "join_field": "patient_id"
      }
    ]
  },
  {
    "table": "measurements",
    "join_on": [
      {
        "ref_field": "obm_id",
        "join_field": "record_id"
      }
    ]
  }
]
```

Minden kapcsolt tábla esetében:

- a `table` a kapcsolandó tábla;
- a `ref_field` az aktuális rekord mezője; és
- a `join_field` a kapcsolt tábla egyező mezője.

Ahol szükséges, ezt a modult az `additional_columns` modullal együtt használja. Gondoskodjon a kapcsolómezők indexeléséről, és arról, hogy a felhasználók jogosultak legyenek minden kapcsolt tábla adataihoz hozzáférni.

### `list_manager`

A `list_manager` modul az adatfeltöltésekhez és lekérdezésekhez használható újrafelhasználható kifejezéslistákat kezeli.

A következőket biztosítja:

- listák létrehozása és szerkesztése;
- listák adatbázistáblákhoz és -oszlopokhoz rendelése;
- listatartalmak előállítása meglévő adatokból;
- listaadatok tárolása az adatbázisban; és
- felhasználói visszajelzés sikertelen listaművelet esetén.

A modul modális párbeszédpanelt használ a listaértékek szerkesztéséhez. Adminisztrációs funkcióihoz csak azok a felhasználók férjenek hozzá, akik jogosultak módosítani a feltöltési és lekérdezési szókészleteket.

**Paraméterek:** Nincsenek. A modul saját felhasználói felületét és modulspecifikus adatbázis-objektumait használja.

### `massive_edit`

A `massive_edit` modul lehetővé teszi a jogosult felhasználóknak, hogy a térképoldalon kiválasztott több rekordot a fájlfeltöltési felületen keresztül szerkesszenek.

A tömeges módosítások sok rekordot érinthetnek. Ellenőrizze a jogosultságokat, készítsen biztonsági mentést, és tesztelje a szerkesztett fájlt egy kis kijelölésen, mielőtt nagyobb frissítést alkalmazna.

**Paraméterek:** Nincsenek.

### `move_project`

A `move_project` modul egy másik OpenBioMaps-kiszolgálóra költöztet egy projektet.

Ez egy kísérleti modul. Használata előtt készítsen és ellenőrizzen biztonsági mentéseket, és ellenőrizze az alkalmazásverziók, adatbázis-bővítmények, projektfájlok, felhasználók, modulok, MapServer-konfigurációk és titkos adatok kompatibilitását a célkiszolgálón.

**Paraméterek:** Nincsenek dokumentálva.

### `read_table`

A `read_table` modul egyedi hivatkozáson keresztül görgethető HTML-táblaként tesz elérhetővé egy SQL-táblát vagy -nézetet.

**Paraméterek:**

```json
[
  {
    "table": "schema.table_name",
    "label": "Displayed table name",
    "orderby": "column_name"
  }
]
```

Minden bejegyzés a következőket tartalmazza:

- `table`: séma szerinti minősítéssel ellátott tábla- vagy nézetnév;
- `label`: a felhasználóknak megjelenő címke; és
- `orderby`: az alapértelmezett rendezéshez használt oszlop.

Egy egyedi vagy nehezen kitalálható hivatkozás önmagában nem elegendő hozzáférés-vezérlés. Ellenőrizze, hogy a modul érvényesíti-e a kívánt projekt-, csoport- és rekordszintű jogosultságokat.

### `results_asList`

A `results_asList` modul összecsukható, diákhoz hasonló bejegyzésekként jeleníti meg a lekérdezési eredményeket.

**Paraméterek:** Nincsenek.

### `results_asGPX`

A `results_asGPX` modul GPX-fájlként exportálja a lekérdezési eredményeket.

**Paraméterek:**

```json
{
  "name": "name_column",
  "description": [
    "description_column_1",
    "description_column_2"
  ]
}
```

A `name` oszlop adja a GPX-elem nevét. A `description` oszlopok értékei szerepelnek az elem leírásában.

Csak a telepített GPX-exportálóval kompatibilis geometriák exportálhatók.

### `results_asCSV`

A `results_asCSV` modul CSV-fájlként exportálja a lekérdezési eredményeket.

**Paraméterek:**

```json
{
  "sep": ",",
  "quote": "\""
}
```

- A `sep` határozza meg a mezők elválasztójelét.
- A `quote` határozza meg a mezőket közrefogó karaktert.

Olyan beállításokat válasszon, amelyek kompatibilisek az export megnyitására használt szoftverrel. Az exportnak továbbra is érvényesítenie kell minden alkalmazandó sor- és oszlopszintű hozzáférési szabályt.

### `results_asJSON`

A `results_asJSON` modul JSON formátumban exportálja a lekérdezési eredményeket.

**Paraméterek:** Nincsenek.

### `results_asTable`

A `results_asTable` modul teljes képernyős, minden elérhető mezőt tartalmazó HTML-táblaként jeleníti meg a lekérdezési eredményeket.

A következőket biztosítja:

- teljes rekordmegjelenítés;
- rendezhető oszlopok; és
- rekordok megtekintésére vagy szerkesztésére szolgáló hivatkozások, ha a felhasználó rendelkezik a szükséges jogosultságokkal.

Minden elérhető mező megjelenítése nagy eredményhalmazok esetén erőforrás-igényes lehet, és olyan mezőket tehet hozzáférhetővé, amelyeket korlátozni kellene. Konfigurálja a hozzáférés-vezérlési modulokat, és minden felhasználói csoport esetében tesztelje a kimenetet.

**Paraméterek:** Nincsenek.

### `results_asKML`

A `results_asKML` modul KML-fájlként exportálja a lekérdezési eredményeket.

**Paraméterek:**

```json
{
  "name": "name_column",
  "description": [
    "description_column_1",
    "description_column_2"
  ]
}
```

A `name` oszlop adja a KML-elem nevét. A `description` oszlopok értékei szerepelnek az elem leírásában.

### `results_buttons`

A `results_buttons` modul vezérlőelemeket ad a térképoldalhoz a lekérdezési eredmények letöltéséhez, mentéséhez, megosztásához és könyvjelzőzéséhez.

Az elérhető letöltési formátumok a megfelelő engedélyezett exportmoduloktól függnek. Ezek között szerepelhet CSV, GPX, KML, SHP és JSON.

A modul a következőket biztosíthatja:

- letöltési gombok;
- mentett lekérdezések és eredmények;
- mentett térbeli kijelölések;
- könyvjelzők;
- megosztás felhasználókkal vagy más OpenBioMaps-projektekkel; és
- integráció a `download_restricted` modullal.

Példakonfiguráció:

```json
{
  "bookmarks": "off",
  "sharing": "off",
  "server_share": "on"
}
```

Konfigurációs kulcsok:

- a `bookmarks` engedélyezi vagy letiltja a lekérdezési könyvjelzőket;
- a `sharing` engedélyezi vagy letiltja az általános megosztást; és
- a `server_share` engedélyezi vagy letiltja a másik kiszolgálóval vagy projekttel való megosztást.

A dokumentált alapértelmezések engedélyezik a könyvjelzőket, és letiltják a megosztást, valamint a kiszolgálók közötti megosztást. A letöltési gombok akkor jelennek meg, ha a hozzájuk tartozó exportmodulok engedélyezve vannak.

Ha a `download_restricted` aktív, a letöltések elérhetősége a kérelmezési és jóváhagyási munkafolyamattól is függ.

### `results_asStable`

A `results_asStable` modul kompakt, rendezhető eredménytáblát jelenít meg a térképoldalon.

A teljes eredménytáblával ellentétben csak a konfigurált oszlopokat jeleníti meg. A rekordok megtekintésére vagy szerkesztésére szolgáló hivatkozásokat is tartalmazhat, ha a felhasználó rendelkezik a szükséges jogosultságokkal.

**Paraméterek:**

```json
[
  "column_name_1",
  "column_name_2"
]
```

A modulnév a korábbi `results_asStable` írásmódot használja; ne nevezze át a konfigurációban.

### `results_specieslist`

A `results_specieslist` modul összesíti az aktuális lekérdezési eredményben előforduló fajokat.

A következőket jelenítheti meg:

- fajnevek;
- az egyes fajokhoz tartozó rekordok száma;
- a rögzített egyedek száma; és
- betűrendes vagy taxonómiai rendezési lehetőségek.

A fajnevekhez és egyedszámokhoz használt oszlopok a projekt sémájától és a modul megvalósításától függenek.

**Paraméterek:** Nincsenek dokumentálva.

### `results_summary`

A `results_summary` modul megjeleníti az aktuális lekérdezés által visszaadott különálló rekordok teljes számát.

Együttműködik a hozzáférési szabályokkal, így a korlátozott rekordokat csak akkor számolja, ha a felhasználó jogosult hozzáférni azokhoz.

Már egy összesített darabszám is érzékeny információt fedhet fel. Tesztelje a modult korlátozott rekordokkal és minden releváns hozzáférési szinten.

**Paraméterek:** Nincsenek.

### `results_table`

A `results_table` bővítmény teljes HTML-táblát hoz létre a lekérdezési eredményekből.

Az OpenBioMaps kiadásától függően ezt a funkciót más megvalósítási nevű modul, például a `results_asTable` vagy a `results_asHtmlTable` biztosíthatja. Pontosan azt a modulnevet használja, amely a telepített rendszer moduladminisztrációs oldalán szerepel.

**Paraméterek:** Nincsenek dokumentálva.

### `restricted_data`

A `restricted_data` modul szabályalapú korlátozásokat alkalmaz a projekt adataira.

A projektnek megfelelően konfigurált hozzáférési szabályokkal és kapcsolódó adatbázis-objektumokkal kell rendelkeznie. Tesztelje a korlátozásokat a térképoldalon, az adatlapoldalon, az exportokban, az API-kon és a közvetlen modulhivatkozásokon keresztül.

**Paraméterek:** Nincsenek.

### `spa_integration`

A `spa_integration` modul egy egyoldalas alkalmazást integrál egy OpenBioMaps-projekttel.

Modulspecifikus adminisztrációs beállításokat igényel. A konfiguráció után tesztelni kell az útválasztást, a hitelesítést, a jogosultságkezelést, a statikus erőforrások útvonalait és a közvetlen böngészőnavigációt.

**Paraméterek:** A modul adminisztrációs felületén kezelhetők.

### `text_filter`

A `text_filter` modul szöveges szűrőket ad a térképoldalhoz és a lekérdezési API-hoz. A konfigurált oszlopok és szűrőoperátorok alapján állítja össze az SQL-lekérdezés szűrési részét.

Példa:

```json
[
  "common_name",
  "obm_taxon",
  "notes::colour_rings",
  "obm_datum",
  "obm_uploading_date",
  "obm_uploader_user",
  "data.abundance:nested(data.count):autocomplete",
  "data.count:values():",
  "obm_files_id",
  "species::autocomplete"
]
```

A bejegyzések egy oszlophivatkozást és azt követő modulspecifikus módosítókat tartalmazhatnak, például:

- `autocomplete`;
- `values()`;
- `nested(...)`; vagy
- `::` karakterekkel elválasztott címke vagy másodlagos mező.

A szintaxis tömör és verziófüggő. Gondosan másolja át az ismerten működő kifejezéseket, kizárólag megbízható adatbázis-azonosítókat használjon, és minden szűrőt teszteljen a térképoldalon és a lekérdezési API-n keresztül is.

### `text_filter2`

A `text_filter2` modul fejlett taxonómiai és általános szöveges szűrőket biztosít. A `text_filter` modulhoz hasonlóan feltételeket ad az SQL-lekérdezéshez.

**Paraméterek:**

```json
{}
```

A további beállítások a modul felhasználói felületén vagy projektspecifikus konfigurációban kezelhetők.

### `transform_data`

A `transform_data` modul átalakítja a rekordértékeket, mielőtt azok megjelennének az eredményterületeken vagy bekerülnének a támogatott exportokba.

Az elérhető átalakítások a következők:

- `geom`: kattintható geometriahivatkozást hoz létre, amely megnyitja a helyet a térképen;
- `geom_nolink`: hivatkozás nélkül jeleníti meg az egyszerűsített WKT-t;
- `geom_wkt`: a szokásos WKT-ábrázolást jeleníti meg;
- `date_yearonly`: kinyeri az évet egy dátumból;
- `translate`: előre meghatározott szöveges állandókat fordít le a felhasználóknak megjelenő szövegre;
- `obslistlink`: hivatkozást hoz létre egy megfigyelésilista-azonosítóból; és
- `uplid`: a modul által támogatott feltöltésazonosító-átalakítást alkalmazza.

Példa:

```json
{
  "obm_geometry": "geom",
  "other_geometry": "geom_nolink",
  "obm_uploading_id": "uplid",
  "date_time_field": "date_yearonly",
  "method": "translate",
  "obm_observation_list_id": "obslistlink"
}
```

Minden kulcs egy adatbázis-oszlop, minden érték pedig az adott oszlopon alkalmazott átalakítás.

Az átalakítások a megjelenítést befolyásolják, az alapul szolgáló tárolt értéket nem. Ellenőrizze, hogy az átalakított hivatkozások és értékek nem fednek-e fel korlátozott adatokat.

### `pds_test_evaluator`

A `pds_test_evaluator` modul a PDS-űrlapfeltöltés kézi tesztfutásait rögzíti és értékeli. Inicializáláskor létrehozza a saját tábláit a projektsémában. Előbb a PDS mobilűrlap-tesztkatalógust kell telepíteni a `protocol_installer` modullal, különben a modul engedélyezése hibaüzenettel meghiúsul. Az adminisztrációs oldal előkészíti a tesztforgatókönyveket és a referenciaválaszokat.

**Paraméterek:** Nincsenek.

### `custom_data_check`

A `custom_data_check` modul projektspecifikus ellenőrzéseket futtat a feltöltött adatokon. A paraméter értéke az ellenőrzőfüggvények nevének pontosvesszővel elválasztott listája. Az ellenőrzőfüggvényeket a modul forráskódjában, az egyéni függvények számára fenntartott blokkban kell megvalósítani, és `true` vagy `false` értéket adnak vissza. A `colour_rings` ellenőrzés példaként be van építve. A `select_list(` kezdetű nevű ellenőrzéseket a modul mindig elfogadja.

A modul minimális logikájú keretrendszer, használatra kész ellenőrzéseket nem tartalmaz: új ellenőrzés hozzáadásához a modul forrását kell szerkeszteni.

**Paraméterek:**

```text
check_function_1;check_function_2
```

### `results_asHtmlTable`

A `results_asHtmlTable` modul a lekérdezés eredményét egyszerű HTML-táblázatként jeleníti meg. A munkamenet ideiglenes lekérdezéstáblájából legfeljebb 2000 sort olvas be. A standard eredménynézetek nem használják a modult; helyette használja a `results_asTable` modult.

**Paraméterek:** Nincsenek.

### `results_asPDF`

A `results_asPDF` modul a lekérdezési eredményeket mPDF-fel előállított PDF-jelentésként exportálja. A jelentést név szerint kell kiválasztani, és az alkalmazás témájának `views/documents/report_<név>.mustache` Mustache-sablonjából jön létre. Az alapértelmezett téma az `observations` jelentést tartalmazza.

További jelentések, vagy egy jelentés egyéni adat-előkészítése a projekt `local/includes/modules/results_asPDF_CM.php` fájljában adhatók meg, amely a `results_asPDF_CM` osztályt definiálja. A statikus `prepare_report_<név>($de)` metódus adja vissza a sablon adatait, az opcionális statikus `config_<név>()` metódus pedig az mPDF-konfigurációt.

**Paraméterek:** Nincsenek.

### `results_asSHP`

A `results_asSHP` modul a lekérdezési eredményeket ESRI Shapefile-okként exportálja ZIP-archívumban. A lekérdezett táblához geometriaoszlopot kell megadni. Az eredményben talált minden geometriatípushoz egy Shapefile készül, a fájlokat pedig az `ogr2ogr` (GDAL) alakítja át EPSG:4326 vonatkoztatási rendszerben.

Az `ogr2ogr` parancsnak telepítve kell lennie a kiszolgálón. Ha nincs a `PATH`-on, állítsa be a rendszerkonfigurációban a `GDAL_PATH` állandót a parancsot tartalmazó könyvtárra.

**Paraméterek:** Nincsenek dokumentálva.

### `service_envimap`

A `service_envimap` modul összeköti a projektet az EnviMap szolgáltatással. Elküldi az érdeklődési terület GeoJSON-ját, listázza az elérhető szolgáltatásokat, lekéri egy szolgáltatáskérés állapotát, és szolgáltatás-hozzáférési hivatkozásokat és gombokat jelenít meg a felületen. Az adminisztrációs oldal csak projektmesterek számára érhető el.

Az API alap-URL-je az `ENVIMAP_API_BASE` állandóval állítható be. Ha nincs megadva, a `https://api.envimap.hu/v1/` cím használatos.

**Paraméterek:** Nincsenek dokumentálva.

### `snap_to_grid`

A `snap_to_grid` modul elavult, és a `grid_view` teljesen helyettesíti. A be nem jelentkezett látogatók számára az `ST_SnapToGrid` függvénnyel kerekíti a geometriaoszlopot. A paraméter két, pontosvesszővel elválasztott rácsméret; az alapértékek `0.13;0.09`. Új projektekben ne használja.

**Paraméterek:**

```text
x_size;y_size
```

### `ebp`

Az `ebp` modul EuroBirdPortal-modulvázlat. Egy táblaszintű modul sablonszerkezetét tartalmazza (fordítható szövegek, AJAX-függőségek, külső függőségek és egy példa SQL-t tartalmazó `init` metódus), de éles funkcionalitást nem. Éles projektben ne engedélyezze.

**Paraméterek:** Nincsenek.

## Háttérben futó segédszkriptek

A modulkönyvtár alábbi fájljai nem modulok. Az alkalmazás indítja őket parancssori szkriptként a háttérben, ezért nem szabad engedélyezni vagy kézzel meghívni őket.

### `fill_table_with_rows`

A `results_asTable` aszinkron munkafolyamata. Azután tölti fel a találati táblát sorokkal részletekben, hogy az első sorok már megjelentek a böngészőben. Az első részlet alapértelmezés szerint 80 sor. A szkript csak parancssorból fut, a munkamenet-változókat környezeti változón keresztül kapja, amelyet induláskor eltávolít.

### `fill_stable_with_column`

A `results_asStable` aszinkron munkafolyamata. A háttérben tölti fel a stabil találati tábla oszlopait, és a munkamenetet egy base64-kódolású környezeti változóból állítja vissza.

### `computation_fork`

A `computation` modul parancssori szkriptje: csomagba rendezi a projekt `computational_packages` könyvtárának egy számítási csomagját, feltölti a beállított munkakiszolgálóra, és az alkalmazás gyorsítótárán keresztül jelzi az előrehaladást. A csomag beállításait a csomag `computation_conf.yml` fájljából olvassa, a munkakiszolgáló URL-jét pedig a `computation_state.yml` fájlba menti. A PHP YAML-bővítmény szükséges.

## Üzembe helyezési ellenőrzőlista

Mielőtt egy modult éles környezetben engedélyezne:

1. Ellenőrizze, hogy a modul szerepel-e a telepített OpenBioMaps-verzióban.
2. Ellenőrizze a JSON-paraméterek érvényességét.
3. Azonosítsa a szükséges modulokat, adatbázis-objektumokat, MapServer-rétegeket, feladatokat, külső szolgáltatásokat és PHP-bővítményeket.
4. Vizsgálja felül a modulhozzáférést és a csoporthozzáférést.
5. Tesztelje nyilvános, hitelesített, csoporttag és adminisztrátori fiókokkal.
6. Tesztelje a közvetlen URL-eket, exportokat és API-kat a hozzáférés-vezérlés megkerülhetőségének ellenőrzésére.
7. Ellenőrizze az alkalmazás, a PHP, a PostgreSQL és a háttérfeladatok naplóit.
8. Készítsen biztonsági mentést a projektről, mielőtt adatbázis-objektumokat módosító vagy tömeges frissítéseket végző modulokat engedélyezne.
9. Dokumentálja a konfigurációt, valamint a modul letiltásának vagy visszaállításának eljárását.
10. Ismételje meg a teszteket az OpenBioMaps frissítése után.
