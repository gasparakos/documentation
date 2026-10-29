.. |br| raw:: html

    <br>
    

API-dokumentáció
****************

.. _new-api:

OpenAPI
=======
A 3.0-s API-verziótól kezdve:

`<https://gitlab.com/openbiomaps/api/obm-project-api/#openbiomaps-project-api>`_

Példa Swagger UI felületre:

`<https://openbiomaps.org/projects/checkitout/api/v3/swagger-ui>`_

Az OpenBioMaps Project API az ``obm-project-api`` tárolóban fejlesztett REST/GraphQL háttérrendszer, amely az OpenBioMaps-projekteket külső kliensekkel, például mobilalkalmazásokkal és R-szkriptekkel köti össze. Minden válasz JSON formátumú. Minden projektnek saját API-példánya van.

Alapútvonal
-----------

Az API alapútvonala ``{obm-server-url}/projects/{project-name}/api``. A verzióelőtag az alábbi végpontútvonalak része, például ``https://openbiomaps.org/projects/checkitout/api/v3/forms``.

Hitelesítés
-----------

A védett végpontok az OBM-kiszolgáló OAuth 2 hozzáférési tokenjét használják (lásd lentebb az OAUTH szakaszt). Minden védett végponthoz a felsorolt hatókörök (scope) valamelyike szükséges. Új hozzáférési token a ``/v3/auth/refresh`` címen kérhető.

Végpontok
---------

.. list-table::
   :header-rows: 1
   :widths: 18 40 42

   * - Név
     - URI
     - Szükséges hatókörök (bármelyik)
   * - OpenAPI
     - ``/v3/specification``
     - nyilvános
   * - Swagger UI, Scalar UI
     - ``/v3/swagger-ui``, ``/v3/scalar-ui``
     - nyilvános
   * - userProjects
     - ``/v3/projects``
     - apiprofile, webprofile, get_project, get_project_list
   * - projectDetails
     - ``/v3/project-details``
     - apiprofile, webprofile, get_project, get_project_vars
   * - projectForms
     - ``/v3/forms``
     - apiprofile, webprofile, get_form, get_form_list
   * - formDetails
     - ``/v3/forms/{formId}``
     - apiprofile, webprofile, get_form, get_form_data, put_data
   * - messageCounter
     - ``/v3/message-count``
     - apiprofile, webprofile, get_message_count
   * - tracklogs
     - ``/v3/tracklogs``
     - apiprofile, webprofile, tracklog
   * - dataTables
     - ``/v3/data-tables``
     - apiprofile, webprofile, get_tables
   * - dataTable
     - ``/v3/data-tables/{schema}/{dataTable}``
     - az útválasztó nem ellenőriz hatókört
   * - columnUniqueValues
     - ``/v3/data-tables/{schema}/{dataTable}/{column}/unique-values``
     - az útválasztó nem ellenőriz hatókört
   * - getData
     - ``/v3/get-data``
     - apiprofile, webprofile, get_data
   * - getRules
     - ``/v3/get-rules``
     - apiprofile, webprofile, get_data
   * - sharedGeometries
     - ``/v3/shared-geometries``
     - apiprofile, webprofile, get_data
   * - getPublicData
     - ``/v3/get-public-data``
     - nyilvános
   * - authRefresh
     - ``/v3/auth/refresh``
     - nyilvános
   * - statistics
     - ``/v3/statistics/calculate``
     - nyilvános

A ``/v3/shared-geometries`` végpont a megosztott geometriákat GeoJSON FeatureCollectionként streameli, kurzoralapú lapozással (oldalanként legfeljebb 1000 elem). Térbeli adatfájlokat (Shapefile, GeoPackage, KML, KMZ) is fogad, amelyeket az ``ogr2ogr`` importál a megosztott geometriák táblájába. A megosztott geometriák módosítása még nincs megvalósítva.

A ``/v3/statistics/calculate`` végpont a felhasználó által megadott érdeklődési terület fajainak és rekordjainak számát adja vissza. A bemeneti geometria WKT formátumú, WGS84 (SRID 4326) POLYGON vagy MULTIPOLYGON lehet.

A ``/v3/get-data`` és a ``/v3/get-public-data`` GraphQL-végpont. Szűrőoperátoraikat, lekérdezéstípusaikat és válaszsémáikat a `GraphQL User Guide <https://gitlab.com/openbiomaps/api/obm-project-api/-/blob/main/GraphQLUserGuide.md>`_ ismerteti.

A Project API telepítése
------------------------

A Project API még nem része az ``obm-composer`` telepítésnek, ezért a régebbi és a kézzel telepített kiszolgálókon kézi beállítás szükséges. Az API-hoz PHP 8.3 vagy újabb, Composer, valamint PostgreSQL 13 vagy újabb PostGIS-szel szükséges.

1. Töltse le az ``obm-project-api`` tároló kiadását, és telepítse a függőségeket a ``composer install --no-dev`` paranccsal.
2. Másolja a fájlokat az OBM-projektkönyvtár ``api`` mappájába (``projects/{project-name}/api``). Minden projekthez saját másolat kell.
3. Adja hozzá a következő átírási szabályt a projekt ``.htaccess`` fájljához, közvetlenül a ``RewriteEngine On`` sor után:

   .. code-block:: apacheconf

      RewriteCond %{REQUEST_URI} ^/projects/{YOUR_OBM_PROJECT_NAME}/api/(.*)?
      RewriteCond %{REQUEST_URI} !^/projects/{YOUR_OBM_PROJECT_NAME}/api/index\.php
      RewriteRule ^api/?(.*)$ api/index.php/$1 [NC,L,QSA]

   Ha a projekt saját tartományon érhető el, ezt használja helyette:

   .. code-block:: apacheconf

      RewriteCond %{REQUEST_URI} !^/api/index\.php
      RewriteRule ^api/?(.*)$ api/index.php/$1 [NC,L,QSA]

4. Szükség esetén hozzon létre egy ``.env`` fájlt az API mappájában az alapértékek felülírásához. Az API enélkül is működik. A támogatott kulcsok: ``ENV`` (``PROD``, ``STAGING``, ``DEV``, ``TESTING``; alapérték ``PROD``), ``LOG_LEVEL`` (``DEBUG``, ``INFO``, ``WARNING``, ``ERROR``, ``CRITICAL``; alapérték ``INFO``), ``LOG_FORMAT`` (``text`` vagy ``json``; alapérték ``text``), ``LOG_FILE`` (alapérték ``php://stdout``), ``SYSTEM_VARS`` (alapérték ``/etc/openbiomaps/system_vars.php.inc``) és ``LOCAL_VARS`` (alapérték az API mappa szülőkönyvtárában lévő ``local_vars.php.inc``).
5. Az új API-ra irányított PDS-hívásokhoz lásd az ``obm-project-api`` README beállítási megjegyzéseit.

.. _old-api:

PDS API
=======
Az 1.0–2.6-os API-verziókban.

HTTP-metódusok: GET, POST

API-eszközök: hitelesítés, adatlekérés, adatfeltöltés, beállítások frissítése

Ez az első OpenBioMaps API (1.0, 2.0–2.6), amelynek kivezetése 2026
szeptemberétől várható. Helyét a 3-as API-verzió (Swagger–OpenAPI) veszi át.

API-kezelők:
------------

Hitelesítéskezelő (OAUTH):

/oatuh/token.php: hitelesítés és engedélyezés

Hitelesítésen alapuló felület (PDS/Oauth):

/projects/*projectname*/*API_VERSION*/pds.php: adatlekérés, adatfeltöltés,
beállítások frissítése

Nem hitelesített kérések (web):

/projects/*projectname*/index.php

A PDS API verziója
..................

Példa: http://openbiomaps.org/projects/dead_animals/v2.1/pds.php

Ha a verziókarakterlánc hiányzik az URL-ből, az alapértelmezett verzió az
1.1, amely kompatibilis a 2.0-s verzióval, és visszafelé kompatibilis az
1.0-s verzióval.


OAUTH
-----

A https://bshaffer.github.io/oauth2-server-php-docs/ megoldáson alapuló
OAuth2-megvalósítás. Az OAUTH-t a webes felület és a PDS is használja.

Változók
........

- grant_type: password
- username: regisztrált e-mail-cím
- password: jelszókarakterlánc
- scope: a hitelesített munkamenetben kért hozzáférési hatókörök listája

A kliensek HTML-alapú hitelesítése szükséges.

Az elérhető kliensek: mobile, R, web.

Az OAUTH kizárólag a következő típusú kéréseket fogadja el:

- application/x-www-form-urlencoded
- multipart/form-data

Hatókörök:

- get_form_data
- get_form_list
- put_data
- ...

.. _pds-api:

PDS API
-------

Az OpenBioMaps fő API-felülete. Alapvetően R- és mobilkliensekhez készült.
A hitelesítéshez OAUTH-t használ.

Az OAUTH-feldolgozás miatt kizárólag ``application/x-www-form-urlencoded``
és ``multipart/form-data`` típusú kéréseket fogad el.

PDS-változók
............

- scope: adatmetódusok; lásd alább
- value: a legtöbb hatókör használja
- header: adatfeltöltéskor a táblaoszlopok neveinek JSON-listája
- ignore_warning: adatfeltöltéskor a feltöltési figyelmeztetések figyelmen kívül hagyása
- form_id: a feltöltési űrlap azonosítója a ``put_data`` használatakor
- data: a feltöltött adatok JSON-tömbje


GET típusú hatókörök
....................

**get_project_vars**

A projekt általános változóinak lekérdezése. Nem bejelentkezett felhasználók
számára is elérhető.

További paraméterek:

- project [text]: ha nincs megadva, az alapértelmezett projekt a *template*

Visszatérési érték:

- project_url [url string]: a projekt webcíme
- project_description [text string]: a projekt rövid leírása
- game [on/off]: elérhető-e játék az androidos mobilalkalmazásban
- public_mapserv [url string]: a nyilvánosan hozzáférhető térképszolgáltatás URL-je
- rserver_port [numeric]: a projekt URL-jén elérhető R Shiny-szerver numerikus portszáma

**get_project_list**

A szerveren elérhető adatbázisprojektek listáját és alapinformációit adja
vissza. Ha a felhasználó már bejelentkezett, azoknak a projekteknek a
listáját adja vissza, amelyekben a felhasználó fiókkal rendelkezik, illetve
amelyekben nyilvános lekérdezési vagy feltöltési felület érhető el. Ha a
felhasználó nincs bejelentkezve, csak a nyilvános projekteket kérdezi le.

További paraméterek:

- only-project [text]: csak a kiválasztott projekt paramétereinek lekérdezése; alapértelmezés szerint minden hozzáférhető projekt lekérdezése
- accessible [text]: all/**accessible**. Ha az *accessible* paraméter meg van adva, és értéke „accessible” (alapértelmezett)

Visszatérési érték:

- project_table [string]
- creation_date [date string]
- Creator [string]
- email [string]
- stage [string]: experimental/testing/stable
- doi [string]
- running_date [date string]
- license [string]
- rum [string]
- collection_dates [date range string]
- subjects [text]

**get_form_list**

Az elérhető feltöltési űrlapok listájának lekérdezése.

**get_form_data**

A kiválasztott űrlap mezőinek lekérdezése.

További paraméterek:

- value [numeric]: egy űrlap numerikus azonosítója

Visszatérési érték: :ref:`lásd az alábbi példában <get_form_data_example>`.

A változók magyarázata:

*default value*: minden megfigyeléshez tartozó rögzített érték. A következő
beállításokkal szabályozható:

- '_input': ugyanúgy működik, mint bármely más, sticky jelzővel ellátott mező
- '_list': ugyanúgy működik, mint bármely más, sticky jelzővel ellátott listatípusú mező
- '_geometry': geometriatípusú mezőként működik
- '_login_name': ezt az értéket a bejelentkezett felhasználó neve felülírja; bejelentkezés nélkül ``_input`` értékként viselkedik
- '_email': ezt az értéket a bejelentkezett felhasználó e-mail-címe felülírja; bejelentkezés nélkül ``_input`` értékként viselkedik
- '_autocomplete': az input álneve
- '_boolean': normál logikai listaként jelenik meg
- '_attachment': normál csatolmánymezőként jelenik meg
- '_datum': normál dátummezőként jelenik meg
- '_auto_geometry': további beállítások nélküli geometriamező (map, set)
- '_none': nincs használatban

*column*: az adatbázisoszlop neve

*short_name*: az oszlop felhasználók számára látható neve

*list*: a kiválasztási menü elemeinek JSON-tömbje. Formátuma lehet
``{key:value}`` vagy ``[value,value]``.

*control*: adatellenőrzési parancsok: custom_check, minmax, spatial,
nocheck, NULL

*count*: JSON-tömb. Ha a ``control='minmax'``, ez a mező tartalmazza a
határértékeket, például ``1:100``.

*type*: az oszlop OpenBioMaps-típusa:

- autocomplete (JSON-tömb)
- autocomplete_list (JSON-tömb)
- boolean (kételemű lista)
- crings (színes gyűrűk – szöveg)
- date (YYYY-MM-DD vagy más egyértelmű formátum)
- datetime (YYYY-MM-DD HH:mm:ss)
- file_id (a szerver által azonosítóként használt fájlnevek)
- line (WKT-geometriakarakterlánc)
- list (JSON-tömb)
- numeric
- point (WKT-geometriakarakterlánc)
- polygon (WKT-geometriakarakterlánc)
- text
- time (HH:mm)
- timetominutes (0 és 1440 közötti numerikus érték)
- tinterval időintervallum (HH:mm - HH:mm)
- wkt (WKT-karakterlánc)
- array (JSON-tömb)

*genlist*: az automatikus kiegészítési menü elemeinek JSON-tömbje.
Formátuma lehet ``{key:value}`` vagy ``[value,value]``.

*obl*: 1, 2, 3 (kötelező, nem kötelező, enyhe hiba). Az enyhe hiba nem
kötelező mezőként kezelhető.

*api_params*: a vezérlőértékek JSON-tömbje. A 2.0-s API-verzióig csak a
``sticky`` szerepelhet tömbelemként.

A 2.0-s API-verzió feletti ``api_params``:

.. code-block:: json

  {
   "sticky":"off",
   "hidden":"off",
   "readonly":"off",
   "list_elements_as_buttons":"off",
   "once":"off",
   "unfolding_list": "off"
  }

*spatial_limit*: a térbeli korlátozás WKT-poligonkarakterlánca. Akkor
használatos, ha a vezérlő típusa ``spatial``.

*list_definition*: az összetett listadefiníció JSON-tömbje

*custom_function*: null

*custom_label*:

*field_description*:


**get_profile**

A kiválasztott felhasználó profiladatainak lekérése.

**get_data**

Adatsorok lekérése a kiválasztott adattáblából, vagyis a megfigyelési
adatokból.

**get_specieslist**

A projekt fajlistájának lekérése.

**get_history**

A kiválasztott adatsor előzményeinek lekérése.

**get_report**

Előre meghatározott lekérdezés végrehajtása és az eredmény lekérése.

**get_tables**

A projekt táblalistájának lekérése.

**get_trainings**

A 2.6-os API-verziótól nem érhető el.

Az elérhető képzések és űrlapok listájának lekérése.

Visszatérési érték:

- a képzések címeinek, azonosítóinak és leírásainak halmaza stb.

**get_training_questions**

A kiválasztott képzés kérdéslistájának lekérése.

A 2.6-os API-verziótól nem érhető el.

További paraméterek:

- value [numeric]: egy képzés numerikus azonosítója

Visszatérési érték:

- a kérdések, válaszok és beállítások halmaza

**training_results**

A felhasználók képzési állapotának listája minden űrlaphoz. Az állapot
értéke -1 (nincs elküldve), 0 (még nincs validálva) vagy 1 (kész, megfelelő)
lehet.

A 2.6-os API-verziótól nem érhető el.

**training_toplist**

A képzések toplistája. Átlag-, maximum- és darabszámértékek minden űrlaphoz.

A 2.6-os API-verziótól nem érhető el.

További paraméterek:

- value [text]: nevek nélküli összefoglaló (nonames)

**get_mydata_rows**

A feltöltött adatok JSON-tömbje.

További paraméterek:

- Value [numeric]: a tömb hosszkorlátja. Ha 0, nincs korlát; alapértelmezés szerint nincs korlát.


POST típusú hatókörök
.....................

**put_data**

Adatok küldése vagy feltöltése egy kiválasztott űrlap használatával.

A következők egyike lehet:

    - tracklog
    - form_id

A ``form_id`` kötelező paraméterei:

    - header
    - data

A ``form_id`` opcionális paraméterei:

    - metadata
    - api_warnings
    - srid
    - description
    - upload_table_post
    - default_values

Fájlfeltöltés.


PATCH típusú hatókörök
......................

*set_rules*

Meghatározott beállítások frissítése.


PDS-példák
==========

**Hitelesítési példák**
-----------------------

CURL használata:

``curl -u mobile:123 https://openbiomaps.org/oauth/token.php -d "grant_type=password&username=foo@foobar.hu&password=mysecretpassword&scope=get_form_data+get_form_list+put_data" | jq``

Konkrét hibaüzenetek:

.. code-block:: json

  {
    "error": "invalid_grant",
    "error_description": "Invalid username and password combination"
  }

Sikeres válasz:

.. code-block:: json

  {
    "access_token": "2cf59c094cc83498355ee9f520848efab6f71fe02",
    "expires_in": 3600,
    "token_type": "Bearer",
    "scope": "get_form_data get_form_list put_data apiprofile",
    "refresh_token": "e14dd3e0f13dffb17d36b2acfe9d161fd4ec1d4fb"
  }

Frissítési token használata:

``curl -F 'grant_type=refresh_token' -F 'refresh_token=e14dd3e0f13dffb17d36b2acfe9d161fd4ec1d4f3' -F 'client_id=R' https://openbiomaps.org/oauth/token.php | jq``

Visszatérési érték:

.. code-block:: json

  {
    "access_token":"ccc1d3e0f13dffb17d36b2acfe9d161fd4ec1d4de",
    "expires_in":3600,
    "token_type":"Bearer",
    "scope":"get_form_data get_form_list",
    "refresh_token":"a1e1d3e0f13dffb17d36b2acfe9d161fd4ec1d27c"
  }

.. _get_form_data_example:

**get_form_data példák**
------------------------

CURL használata:

``curl -F 'access_token=c53c9ec690fede4c3' -F 'scope=get_form_data' -F 'value=246' -F 'project=dead_animals' https://openbiomaps.org/projects/dead_animals/v2.3/pds.php | jq``

Konkrét hibaüzenetek:

.. code-block:: json

  {
   "status": "error",
   "message": "Form access denied.",
   "data": ""
  }

Sikeres válasz:

.. code-block:: json

  {
   "status": "success",
   "message": "",
   "data": {
     "form_header": {
       "login_name": "Gipsz Jakab",
       "login_email": "jakab.gipsz@openbiomaps.jupyter.ga",
       "boldyellow": [
         "species"
       ],
       "num_ind": "quantity",
       "tracklog_mode": "",
       "observationlist_mode": "false",
       "observationlist_time_length": "0",
       "periodic_notification_time": null
     },
     "form_data": [
       {
        "description": "...",
        "default_value": "...",
        "column": "species",
        "short_name": "Scientific species name",
        "list": ["..."],
        "control": "nocheck",
        "count": "{}",
        "type": "list",
        "genlist": null,
        "obl": "1",
        "api_params": {
          "sticky": "on",
          "hidden": "off",
          "readonly": "off",
          "list_elements_as_buttons": "on",
          "once": "off",
          "unfolding_list": "off"
        },
        "spatial_limit": null,
        "list_definition": {
          "multiselect": false,
          "selected": null,
          "triggerTargetColumn": [],
          "Function": ""
        },
        "custom_function": null,
        "column_label": null,
        "field_description": "..."
       }
    ]
   }
  }


**get_form_list példák**
------------------------

CURL használata:

``curl https://openbiomaps.org/projects/checkitout/pds.php -d "access_token=d4fba6585303bba8da3e6afc1eb9d2399499ef3eb&scope=get_form_list"``

.. code-block:: json

  {
   "status": "success",
   "message": "",
   "data": [
    {
      "id": "1017",
      "visibility": "Observation list - obligatory / tracklog no",
      "form_id": "1017",
      "published_form_id": "1016",
      "form_name": "Observation list - obligatory / tracklog no",
      "last_mod": "1674809097"
    },
    {
      "id": "938",
      "visibility": "relational columns test",
      "form_id": "938",
      "published_form_id": "937",
      "form_name": "relational columns test",
      "last_mod": "1660679646"
    }]
  }

**Adatfeltöltési példák**
-------------------------

CURL használata:

  curl -i -X POST \\ |br|
  -H "Content-Type:application/x-www-form-urlencoded" \\ |br|
  -H "Authorization:Bearer ..." \\ |br|
  -d "scope=put_data" \\ |br|
  -d "form_id=128" \\ |br|
  -d "header=[\"obm_geometry\",\"datum\",\"comment\",\"observer\"]" \\ |br|
  -d "data=[{\"obm_geometry\":\"point(48.071187 19.293714)\",\"datum\":\"2018-04-03\",\"comment\":\"asdad\",\"observer\":\"sdsaada\"}]" \\ |br|
  -d "ignore_warning=1" \\ |br|
  'https://openbiomaps.org/projects/checkitout/v2.5/pds.php'

JavaScript használata:

.. code-block:: javascript

    const xhr = new XMLHttpRequest();
    xhr.open("POST", "https://openbiomaps.org/projects/checkitout/v2.5/pds.php");
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded; charset=UTF-8");
    const encodedData = Object.keys(data)
        .map(key => encodeURIComponent(key) + '=' + encodeURIComponent(data[key]))
        .join('&');
    xhr.onload = () => {
      if (xhr.readyState == 4 && xhr.status == 201) {
        console.log(JSON.parse(xhr.responseText));
      } else {
        console.log(`Error: ${xhr.status}`);
      }
    };
    xhr.send(encodedData);

Adatfeltöltés több csatolmánnyal, vagyis fájllal:

    curl \\ |br|
    -F "access_token=..." \\ |br|
    -F 'scope=put_data' \\ |br|
    -F 'form_id=58' \\ |br|
    -F 'header=["species","obm_geometry","obm_files_id"]' \\ |br|
    -F 'batch=[\\ |br|
    {"data":[{"species":"Sylvia curruca","obm_geometry":"POINT(22.0 46.3)"}],"attached_files":"file1,file2"},\\ |br|
    {"data":[{"species":"Lanius Collurio","obm_geometry":"POINT(21.5 47.1)"}],"attached_files":"file3"}]' \\ |br|
    -F 'file1=@file1' \\ |br|
    -F 'file2=@file2' \\ |br|
    -F 'file3=@file3' \\ |br|
    http://localhost/biomaps/projects/template/pds.php

Csomagolt adatfeltöltés. Adatsor ZIP-archívumban. Ez a régi
mobilalkalmazás exportformátuma. A ZIP-fájl a következő fájlokat tartalmazza:
|br|

    geometry.wkt |br|
    PICT01.JPG |br|
    PICT02.JPG |br|
    note.txt |br|

A ZIP-fájl neve ``Sun May 13 08:52:51 CEST 2018.zip``, amely a megfigyelés
dátum-idő karakterláncából készült. A ``note.txt`` a megfigyeléshez tartozó
megjegyzést tartalmazza, amely az űrlap egyik oszlopához rendelhető. Ebben a
példában ez a ``species``. A másik három oszlopot nem szabad lecserélni vagy
figyelmen kívül hagyni. Ha az űrlap kötelező oszlopokat tartalmaz, azok a
``default_value`` paraméterrel tölthetők ki. Ebben a példában az
``egyedszam`` kötelező mező, amely az ``1`` értéket kapja. A csomagolt sorok
tovább csomagolhatók. Ebben az esetben a ``packed_line`` paramétert
``multipacked_lines`` értékre kell cserélni, a ZIP-archívumnak pedig a fent
ismertetett ZIP-fájlokat kell tartalmaznia.

    curl \\ |br|
    -F 'scope=put_data' \\ |br|
    -F 'table=dinpi' \\ |br|
    -F 'form_id=58' \\ |br|
    -F 'header=["obm_geometry","obm_files_id","faj","dt_to"]' \\ |br|
    -F 'default_values={"egyedszam":"1"}' \\ |br|
    -F 'packed_line=@Sun May 13 08:52:51 CEST 2018.zip' \\ |br|
    http://localhost/biomaps/pds.php


**get_project_list példa**
--------------------------

CURL használata:

Ez egy nem hitelesített PDS-kérés:

``curl https://openbiomaps.org/projects/checkitout/v2.5/pds.php -d "scope=get_project_list&value=" | jq``

Sikeres válasz:

.. code-block:: json

  {
  "status": "success",
  "data": [
    {
      "project_table": "checkitout",
      "creation_date": "2016-03-09",
      "Creator": "",
      "email": "",
      "stage": "sandbox",
      "doi": null,
      "running_date": null,
      "licence": "ODbL",
      "rum": "+++",
      "collection_dates": null,
      "subjects": null,
      "project_hash": "28gmst44rm8g",
      "project_url": "https://openbiomaps.org/projects/checkitout/",
      "project_description": "Checkitout! Sandbox.",
      "public_mapserv": "-",
      "training": "f",
      "rserver": "f",
      "language": "hu",
      "game": "off",
      "rserver_port": 0
    } 
  ] 
  }

Képzések magyarázatai és példái
-------------------------------

A 2.6-os API-verziótól nincs kliens.

Curl használata:

``curl -F 'scope=get_trainings' -F 'access_token=9d45...' -F 'project=dinpi' http://localhost/biomaps/pds.php``

Sikeres hívás eredménye:

.. code-block:: json

  {"status":"success",
   "data":[
    {"id":"1","form_id":"95","html":"<div>...",,"task_description":"<div>...","enabled":"t","title":"Gyakorlás I.","qorder":"1","project_table":"dinpi"}]}

``curl -F 'scope=get_training_questions' -F 'access_token=9d45...' -F 'project=dinpi' http://localhost/biomaps/pds.php``

Sikeres hívás eredménye:

.. code-block:: json

  {"status":"success",
   "data":[
    {"qid":"1", "training_id":"1", "caption":"...?", "answers":[{"Answer": "...","isRight": "false" } ],"qtype":"multiselect"}]}

A ``qtype`` értéke ``multi-select`` vagy ``single select`` lehet.

``curl -F 'scope=training_results' -F 'access_token=9bb4...' -F 'project=dinpi' http://localhost/biomaps/pds.php``

Sikeres hívás eredménye:

.. code-block:: json

  {"status":"success","data":"{"95":1,"96":0,"97":-1,"98":-1}"}

Az értékek jelentése: a 95-ös űrlap elkészült; a 96-os űrlap elkészült,
de még nincs validálva; a 97-es és 98-as űrlap még nincs befejezve.

``curl -F 'scope=training_toplist' -F 'value=nonames' -F 'access_token=5ac3...' -F 'project=dinpi' http://localhost/biomaps/pds.php``

Sikeres hívás eredménye:

.. code-block:: json

  {"status":"success",
   "data":{
    "95":{"mean":"0.50000000000000000000","count":"2","max":"0.7"},
    "96":{"mean":"0.70000000000000000000","count":"1","max":"0.7"},
    "97":{"mean":"0.70000000000000000000","count":"1","max":"0.7"},
    "98":{"mean":null,"count":"1","max":null}}}

``curl -F 'scope=training_toplist' -F 'access_token=5ac3...' -F 'project=dinpi' http://localhost/biomaps/pds.php``

.. code-block:: json

  {"status":"success","data":{
        "95":{"Gipsz Jakab":{"mean":"0.30000000000000000000","count":"1","max":"0.3"},
              "Foo Aladár":{"mean":"0.70000000000000000000","count":"1","max":"0.7"}},
        "96":{"Foo Aladár":{"mean":"0.70000000000000000000","count":"1","max":"0.7"}},
        "97":{"Foo Aladár":{"mean":"0.70000000000000000000","count":"1","max":"0.7"}},
        "98":{"Mr. Bean":{"mean":null,"count":"1","max":null}}}}


Általános API-válaszok
----------------------

Alapja: https://labs.omniti.com/labs/jsend

Mindig JSON-karakterlánc:

.. code-block:: json

  {
   "status":"X",
   "data":"",
   "message":""
  }

X: success, error, fail

Általános hibaüzenetek
----------------------

.. code-block:: json

  {
    "status": "error",
    "message": "The access token provided is invalid"
  }

.. code-block:: json

  {
    "status": "error",
    "message": "The request requires higher privileges than provided by the access token"
  }



WEB API
-------

Az ``index.php`` bizonyos esetekben API-szolgáltatásként is működik
(``?query=``), kizárólag ``_GET`` kérésekhez és nem hitelesített kérésekhez.

Ez az API ``text_filter`` modulokat használ egy SQL-lekérdezési utasítás
összeállításához.

WEB API-változók
................

query:
    API-végpont.

qtable:
    Az adatlekéréshez használt adattábla.

report:
    Adatlekérés tárolt lekérdezések használatával.

output:
    JSON-, XML-, CSV- vagy más fájlkimenet. Ha nincs beállítva, a kimenet a
    webes felület.

filename:
    A kimeneti fájl neve.

Az aktív, vagyis ismert OpenBioMaps-szerverek listájának lekérése a query
API használatával:

``curl https://openbiomaps.org/projects/openbiomaps_network/index.php -G -d 'query={"available":"up"}&output=json&filename=results.json'``

Szűrt tábla lekérése nem alapértelmezett táblából:

``curl https://openbiomaps.org/projects/pollimon/index.php -G -d 'query={"q":"2"}&output=json&qtable=pollimon_sample_plots'``

LQ API-végpont:

LQ:
    Tárolt lekérdezési eredmény adatainak megjelenítése.

Használati példa:

``wget https://openbiomaps.org/projects/checkitout/?report=2@szamossag&output=csv``
