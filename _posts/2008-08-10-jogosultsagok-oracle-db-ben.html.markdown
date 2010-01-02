--- 
title:      Jogosultságok Oracle DB-ben
created_at: 2008-08-10 12:24:10.047977 +02:00
blog_post:  true
layout:     post
filter:
  - textile
---

A minap szembesültem egy jogosultági problémával Oracle-ben, s mivel régen volt már az OCP vizsga, és én gyorsan felejtek, kénytelen voltam felfrissíteni ismereteimet a témában. Ha már így jártam, s mivel ketten is Oracle-ös posztokra nyilvánították ki igényüket a nyitóbejegyzés kommentjeiben, gondoltam írok a témáról.

Jogosultságok típusai
---------------------

Két fajta jogosultság létezik Oracle-ben: objektum szintű (object privileges), és rendszer szintű (system privileges). A műveletet, amivel jogot adunk egy user-nek bizonyos dologra szép magyarul grantolásnak nevezzük, és meglepő mód a GRANT utasítással lehet elvégezni. Visszavonni a REVOKE-kal lehet.

<dl>

<dt>Rendszer szintű privilégiumok</dt> 
<dd>
ilyen bizonyos műveletek végrehajtásának joga (pl.: csatlakozás az adatbázishoz, vagy táblatér létrehozása), és idetartoznak azon jogosultságok amik egy séma összes azonos típusú objektumára vonatkoznak (pl.: lekérdezés egy séma bármelyik táblájából).
</dd>

<dt>Objektum szintű privilégiumok</dt>
<dd>
egy bizonyos művelet végrehajtásának joga egy bizonyos séma egy bizonyos objektumán (pl.: sorok törlésének joga a HR user DEPARTMENTS tábláján). Bizonyos típusú objektumokhoz (CLUSTER, INDEX, TRIGGER, DATABASE LINK) nem tartoznak objektumszintű privilégiumok, hanem csak rendszer szintűek. Ha egy szinonímára grantolunk objektum szintű privilégiumot, akkor az egyenértékű azzal, mintha a szinoníma által hivatkozott objektumra grantolnánk. A szinoníma eldobása után a jog nem is fog elveszni.
</dd>

</dl>

Szerepek
--------

Jogosultságokat adhatunk felhasználóknak (user) és szerepeknek (role). Mire jók a szerepek? Segítségükkel összefoghatunk privilégiumokat, és ezeket egyszerre oszthatjuk ki felhasználóknak, sőt akár más szerepeknek is, így egyszerűsíthetjük az adiminsztrációt.

Szerepek munkamenet (session) szinten lehetnek ki- és bekapcsolt állapotban. SET ROLE utasítással tudjuk ezt megtenni. Fontos tudni, hogy Oracle-ön belüli programoknak (függvények, eljárások, csomagok) közvetlenül kell kiosztani a jogosultságokat, szerepeken keresztül kiosztott jogosultságot nem használhatnak. Hiszen fordítási időben nem tudhatja az adatbázis, hogy mikor majd a program futni fog, a szükséges szerep aktív állapotban lesz-e.

with admin option, with grant option
------------------------------------

Rendszer szintű privilégium és szerep esetén a WITH ADMIN OPTION megadásával megengedhetjük, hogy a felhasználó, aki kapta a jogot vagy szerepet, tovább is adhassa azt másoknak. Objektum szintű privilégiumok esetén a WITH GRANT OPTION-t kell használnunk, ha azt akarjuk, hogy tovább lehessen adni szerepeknek vagy felhasználóknak.

Ha WITH ADMIN OPTION-nel jogot adtunk Bobónak, majd visszavonjuk, de ő addigra már tovább grantolta Benőnek, akkor a továbbadott jogot Benő nem fogja elveszíteni. Viszont objektum szintű privilégium esetén, ha WITH GRANT OPTION-nel kiosztottuk Bobónak valami, aki továbbadta Benőnek, ezután visszavonjuk Bobótól, akkor ezzel a mozdulattal Benőtől is visszavontuk. Fincsi mi?

Jogosultságok kiosztása
-----------------------

Objektum szintű jogosultságot adunk Bobo nevű kollégának, hogy módosíthassa a departments tábla department_name oszlopát.

<pre>
SQL> GRANT UPDATE (department_name) ON departments TO bobo;</pre>

A looser_privs szerepnek jogot adunk, hogy bármilyen táblát eldobhasson, majd Bobo megkapja a looser_privs-t.

<pre>
SQL> GRANT DROP ANY TABLE TO looser_privs;
SQL> GRANT looser_privs TO bobo;</pre>

Majd rájövünk, hogy inkább visszavonunk mindent

<pre>
SQL> REVOKE UPDATE ON departments FROM bobo;
SQL> REVOKE DROP ANY TABLE FROM looser_privs;</pre>

Jogosultságok lekérdezése
-------------------------

A következő nézetekből tudjuk lekérdezni, hogy kinek milyen jogosultságai vannak:


<dl>
<dt>DBA_USERS</dt><dd>adatbázisban található felhasználók.</dd>
<dt>DBA_ROLES</dt><dd>adatbázisban található szerepek.</dd>
<dt>DBA_TAB_PRIVS</dt><dd>felhasználóknak és szerepeknek kiosztott objektum szintű jogosultságok. Ne zavarjon meg senkit a tábla neve, az összes</dd>
<dt>DBA_SYS_PRIVS</dt><dd>felhasználóknak és szerepeknek kiosztott rendszer szintű jogosultságok.</dd>
<dt>DBA_ROLE_PRIVS</dt><dd>felhasználóknak és szerepeknek kiosztott szerepek.</dd>
</dl>

A végére pedig jöjjön három trükkös lekérdezés ([forrás](http://www.adp-gmbh.ch/ora/misc/recursively_list_privilege.html)). Az első megadja, hogy egy user-nek, milyen szerepei vannak (rekurzívan) és a szerepeken keresztül milyen rendszerszintű privilégiumokkal rendelkezik.


<script src="http://gist.github.com/26456.js"></script>


Megkeresi egy adott rendszer szintű privilégiumhoz, hogy milyen szerepek rendelkeznek vele (rekurzívan) és ezek a szerepek milyen felhasználókhoz tartoznak.

<script src="http://gist.github.com/26458.js"></script>

Megkeresi, hogy adott séma adott objektumára milyen objektumszintű privilégiumok vannak kiosztva közvetlenül vagy szerepeken keresztül (rekurzívan).

<script src="http://gist.github.com/26459.js"></script>


