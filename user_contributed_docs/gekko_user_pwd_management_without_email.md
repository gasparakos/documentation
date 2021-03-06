# Jelszókezelés a gekko OBM felületén levelezőkiszolgáló beállítása nélkül
===================================================================================
2017.02.13

Az OpenBioMaps keretrendszer gekko variánsa egy virtuális gépen fut, amelyet akár otthonra is telepíthető.
A felhasználók és jelszavak beállítása, a felhasználó azonosítása az OpenBioMaps böngészőben elérhető felületén rendkívül
egyszerű, levelezőrendszeren keresztül küldött linkek segítségével lehetséges.

Otthoni, vagy irodai gépre telepített, a saját gépen vagy kis belső hálózaton használt gekko virtuális gépen azonban
gyakran nem lehet, és/vagy nem akarjuk konfigurálni a gekko levelező kiszolgálóját. Hogyan lehet ebben az esetben a 
felhasználókat és a hozzájuk tartozó jelszavakat kezelni?


## Alapfeltételek megteremtése

A jelenlegi verzióban (1.24) a gekko-n nincs telepítve a php5-mcrypt csomag. Ezt a virtuális gépen a terminálban,
rendszergazdaként, az alábbi parancs kiadásával kell megtenni:

*apt-get install php5-mcrypt*

A telepítő megkérdezi, valóban elfogadjuk-e a kijelölt módosításokat, amelyet az ’Y’ leütésével tehetünk meg.
A telepítés sikeres lefutása után az apache2 kiszolgáló újraindítása válhat szükségessé, amihez írjuk be a konzolba
az alábbi parancsot:

*/etc/init.d/apache2 restart*

## Új felhasználó hozzáadása

Új felhasználót a bejelentkezés után lehet meghívni az adatbázisunkba. Ehhez a felhasználónevünkre kattintva
felbukkanó menüből a Profile menüpontot kell választani.

Itt a Meghívások oldal megnyitása után a név, e-mail cím és a meghívó szövegének beállítása után „küldd el” a meghívót.
Valójában e-mailt nem lehet küldeni (hiszen nem állítottuk be a levelezést), de a háttéradatbázisban megtalálhatóak a
megfelelő információk az aktiváláshoz.
A gisadmin felhasználóval be kell lépni a phpPgAdmin felületre (http://192.168.56.101/phpPgAdmin).
Itt a biomaps/public/invites táblát kell megnyitni, ahol az összes aktív meghívón megtalálható. 

A name mezőben a meghívott nevét,
a mail mezőben a felhasználó bejelentkezéséhez szükséges e-mail címet,
és a code mezőben egy automatikusan generált karakterláncot láthatunk.

A meghívót pedig az alábbi módon lehet elfogadni:

http://__KISZOLGALO__/biomaps/projects/__ADATBAZIS_NEVE__/index.php?register=__CODE__

A fenti hivatkozásban a __KISZOLGALO__ helyére a gekko hálózati elérési útját illeszd be,
az __ADATBAZIS_NEVE__ kifejezés helyére annak az adatbázisnak a nevét, amelybe meghívtad a felhasználót,
a __CODE__ helyére pedig a fenti táblázat code mezőjében található karakterláncot írd be!

A template adatbázishoz így nézhet ki egy ilyen meghívás hivatkozása:

http://192.168.56.101/biomaps/projects/template/index.php?register=c4cc1951ed3664dde0264baa8a5f050b

A megfelelő értékekkel behelyettesített hivatkozásra kattintva az új felhasználó profiljába kerülünk,
ahol alapértelmezetten egy automatikusan létrehozott karakterláncot állít be a rendszer. Ezt megváltoztatva, vagy elfogadva állítsd be jelszót, majd elkezdheted az új felhasználóval is a gekko OBM felületének használatát.


## Teendők elfelejtett, vagy megújítandó jelszó esetén

Előfordulhat, hogy elfelejtjük a bejelentkezéshez használt jelszót, de egy autentikációs frissítés esetén is szükség
lehet jelszófrissítésre.
Ehhez a bejelentkezési képernyőn található Elfelejtett jelszó linkre kell kattintani. A megjelenő oldalon add meg a
kérdéses felhasználó által regisztrált e-mail címet.
Az előzőekhez hasonlóan most is a phpPgAdmin felületet lehet segítségül hívni. Most azonban nem az invites táblát,
hanem a biomaps/public/users táblát kell használni. Ebben a táblában a már regisztrált felhasználók találhatók. 
Most a reactivate mezőben található értéket kell segítségül hívni:

http://__KISZOLGALO__/biomaps/projects/__ADATBAZIS_NEVE__/index.php?addr=__EMAIL__&activate=__CODE__

A fenti hivatkozásban a __KISZOLGALO__ helyére a gekko hálózati elérési útját illeszd be,
az __ADATBAZIS_NEVE__ kifejezés helyére annak az adatbázisnak a nevét, amelybe meghívtuk a felhasználót,
az __EMAIL__ helyére a felhasználó regisztrált e-mail címét, a __CODE__ helyére pedig a fenti táblázat
reactivate mezőjében található karakterláncot.

Egy, a template adatbázisban regisztrált felhasználó jelszómódosításhoz használt linkje az alábbihoz hasonlóan kell kinézzen:

http://192.168.56.101/biomaps/projects/template/index.php?addrgekko@openbiomaps.org&activate=Dfg458hz87kl58e4

A megfelelő értékekkel behelyettesített hivatkozásra kattintva az előzőekhez hasonlóan a felhasználó profiljába kerülünk, ahol jelszóként alapértelmezetten egy automatikusan létrehozott karakterláncot állít be a rendszer. Ezt megváltoztatva, vagy elfogadva állítsd be jelszót, majd elkezdheted a gekko OBM felületének használatát.

--

**Fontos, hogy a fenti módszerrel csak az OBM grafikus felületén használt felhasználókra vonatkozó információkat lehet kezelni. A postgreSQL adatbázis (pl. gisadmin), vagy a virtuális gép (gekko) felhasználóit ilyen módon nem lehet manipulálni!**
