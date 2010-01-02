--- 
title:      A nyolc királynő
created_at: 2010-01-02 11:16:41.464954 +01:00
layout:     post
--- 
Az egész tavaly Húsvétkor kezdődött. Verpeléten voltunk Kriszti
anyukájánál három napot. Kellett valami elfoglaltság, ezért megvásároltam
Stuart Halloway [Programming Clojure](http://pragprog.com/titles/shcloj/programming-clojure) című
könyvének PDF verzióját. Nem nagyon kommunikáltam eztán a külvilággal a következő
néhány napban, amíg a végére nem értem. Remek olvasmány volt, nagyon
jól bemutatja a [Clojure](http://clojure.org) és a funkcionális
programozás lenyűgözően elegáns mivoltát. Belelkesedtem, de aztán
lecsendesedett a dolog. Felvettem egy rakás Clojure-rel foglalkozó
blog-ot a reader-ben, amiket szorgalmasan olvastam is, de kódot nem
nagyon írtam. Egészen most karácsonyig.

Amikor is (nem meglepő módon) kiderült, hogy egy programnyelvet nem lehet
megtanulni, ha csak olvasol róla. Bármennyit is olvasol róla. A
nyelvet használni kell, s közben vért izzadni a legtriviálisabb
problémák megoldásáért is. Amiket következő alkalommal persze már
csuklóból old meg az ember.

Első feladatként a [Nyolc királynő
problémát](http://en.wikipedia.org/wiki/Eight_queens_puzzle) tűztem
magam elé. A megoldása nem különösebben bonyolult, éppen megfelelő egy
új nyelv felfedezéséhez. A nyolc királnyő probléma arról szól, hogyan lehet 8 db királynőt
feltenni egy sakktáblára úgy, hogy ne támadják egymást. Összesen 92 db
megoldása van.

Backtracking
------------

Az módszert, amivel a nyolc királynő problémát meg szokás oldani,
[backtracking](http://en.wikipedia.org/wiki/Backtracking)-nek
hívják. A lényege pontokba szedve röviden a következő:

* Az összes olyan állapotot keressük, melyek megoldásai a  problémának.
* A folyamat során mindig rendelkezünk egy állapottal, ami általában részleges, de néha teljes megoldás.
* Minden részleges megoldáshoz meg tudjuk mondani az őt követő állapotok halmazát. Ezek egy lépéssel közelebb vannak egy teljes megoldáshoz.
* Egy állapotról el kell tudnunk dönteni, hogy az teljes megoldása-e a problémának.
* Ha egy részleges megoldáshoz tartozó követő állapotok halmaza üres, akkor zsákutcába kerültünk, ezért visszalépünk az őt megelőző állapotba
  (ez a backtrack), és ott egy másik részleges megoldás felé indulunk tovább.

Ha a fenti pontokat a nyolc kiránynő problámra vetítem, akkor a következőket kapom:

* Mind a 96 megoldást szeretném megkapni.
* Az állapotot egy tömb reprezentálja, ami kezdetben üres. Később a tömb i. eleme mondja meg, hogy az i. oszlopban lévő királynő melyik sorban van.
* Részleges megoldás esetén k oszlophoz rendeltünk királynőt, ahol k < 7 (a számozást nullával kezdjük). Ekkor a követő állapotok halmaza azon állapotokat
  tartalmazza, melyekben már a k + 1-dik oszlopban is van királynő, és ez semelyik előzőt sem támadja.
* Mivel minden részleges megoldásunkra igaz, hogy senki nem támad senkit, egy állapot teljes megoldás, ha az állpot tömbjénak hossza 8.

Clojure implementáció
---------------------

A kód megtekinthető egyben a [ebben a gist](http://gist.github.com/267522)-ben, de itt most
szeretnék végigmenni rajta, hogy lássuk, hogyan épül fel a program. Csapjunk is a lovak közé!

{% highlight clojure %}
(def N 8)

(defn distance [x y] (Math/abs (- x y)))

(defn solution? [state]
  (= N (count state)))

(defn next-states [s]
  (map #(conj s %) (range N)))

{% endhighlight %}

`N` a tábla oldalának hossza, sakktábla esetén 8. A `distance` egy
segédfüggvény, két szám különbségének abszolút értékét számolja ki. A
`solution?` prediktátum eldönti egy állapotról, hogy az teljes
megoldás-e. Esetünkben ez akkor igaz, ha a tömb hossza 8. 

Az első izgalmasabb dolgot a `next-states` függvény végzi. Kiszámolja az
`s` nevű paraméterként kapott állapotból elérhető állapotok listáját. A `(range N)`
esetünkben a `(0 1 2 3 4 5 6 7)` szekvenciává értékelődik ki. A `map`
végigmasíroz ezen a kollekción, és mindegyik elemet hozzácsapja
`s`-hez. A map értéke minden esetben egy nyolc elemű vector lesz. 
Lássunk két példát a `next-states`-re:

{% highlight clojure %}
(next-states [])  ; => ([0] [1] [2] [3] [4] [5] [6] [7])
(next-states [1]) ; => ([1 0] [1 1] [1 2] [1 3] [1 4] [1 5] [1 6] [1 7])
{% endhighlight %}

El kell tudnunk dönteni azt, hogy két királynő támadja-e egymást egy
bizonyos állapotban. Épp ezt teszi a következő függvény. 

{% highlight clojure %}
(defn attack? [s i j]
  (let [xi (s i)
        xj (s j)]
    (or (= xi xj)
        (= (distance xi xj) (distance i j)))))
{% endhighlight %}

Az `s` paraméter az állapotvektor, az `i` és `j` pedig a két
összehasonlítandó királynő indexei. A két királynő az
alábbi esetekben támadja egymást:

* egy oszlopban vannak -- ez itt a reprezentációból következően nem lehetséges
* egy sorban vannak -- `(= xi xj)` feltétel
* átlósan támadják egymást -- ez akkor van, ha a sorindexeik
  különbsége megegyezik az oszlopindexeik különbségével, azaz `(= (distance xi xj) (distance i j))`

Azt is el kell tudnunk dönteni, hogy egy állapot érvényes-e, azaz
nincs-e két olyan királynő, akik támadják egymást. Mivel az
algoritmusunk csak érvényes állapotból lép tovább, elég csak azt
megvizsgálnunk, hogy az utoljára felrakott királynő támadja-e
valamelyik már korábban fentlévőt.

{% highlight clojure %}
(defn valid-state? [s]
  (let [max-idx (dec (count s))
        pairs (map #(list % max-idx) (range max-idx))
        no-attack? (fn [pair]
                     (not (attack? s (first pair) (last pair))))]
    (every? no-attack? pairs)))
{% endhighlight %}

A `max-idx` az utoljára felrakott királynő indexe. A `pairs` azon indexpárok
listája, amelyeket vizsgálnunk kell. Abban az esetben példul, ha most
raktuk fel a negyedik királnyőt a `pairs` értéke `((0 3) (1 3) (2 3))`
lesz, hiszen az új (3-as indexű) királynőt össze kell vetnünk a
0,1,2-es indexűvel. A `no-attack?` egy függvény (predikátum), ami paraméterként egy
indexpárt kap, és a korábban megírt `attack?` függvény segítségével
eldönti, hogy a pár-hoz tartozó királynők támadják-e egymást. A
`valid-state?` visszatérési értékét végül az `(every? no-attack? pairs)`
kifejezés szolgáltatja. Az `every?` a `pairs`-ban lévő összes
indexpárral meghívja a `no-attack?`-ot, és ha mindegyik `true`-val tér vissza, akkor az értéke
igaz lesz, egyébként hamis.

Következő függvényünk megmondja, hogy egy állapotból milyen következő
érvényes állapotokba mehetünk tovább.

{% highlight clojure %}
(defn next-valid-states [s]
  (filter valid-state? (next-states s)))
{% endhighlight %}

Majd ezután nincs más hátra, mint megkeresnünk a megoldásokat. Ezt a
rekurzív `backtrack` függvény végzi. Paramérként egy állapotot kap. Ha
ez az állapot egy megoldás, akkor visszatér vele, ez a rekurzió egyik
alapesete (az alapeset - base case - az az ág, amikor a rekurzív függvény nem
hívja meg magát). Minden más esetben kiszámolja a következő állapotok listáját,
majd a `mapca`-t mindegyikre rekurzívan meghívja a `backtrack`-et és a
végén konkatenálja az eredményként kapott listákat. A másik alapeset
az, amikor a következő állapotok listája egy üres lista. Az
`eight-queens`-nek, ami a végső megoldást szolgáltatja nem kell mást
tennie, mint egy üres vektorral meghívnia a `backtrack`-et.

{% highlight clojure %}
(defn backtrack [state]
  (if (solution? state)
    [state]
    (let [next-states (next-valid-states state)]
      (mapcat backtrack next-states))))

(defn eight-queens [] (backtrack []))
{% endhighlight %}

Végül néhány segédfüggvény, ami az eredmények kiírását végzi.

{% highlight clojure %}
(defn format-solution [solution]
  (let [letters (vec (seq "ABCDEFGH"))
        positions (map #(str-join "" [%1 (+ 1 %2)]) letters solution)]
    (str-join ", " positions)))

(defn print-solutions [solutions]
  (doseq [solution solutions]
    (println (format-solution solution))))

(defn print-all-solution []
  (print-solutions (eight-queens)))
{% endhighlight %}

Kicsit hosszúra sikerült ez a poszt, de ha valaki végiolvassa remélem
felkelti az érdklődését a Clojure iránt. Hajrá!
