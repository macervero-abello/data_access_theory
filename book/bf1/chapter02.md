# Tractament de fitxers XML

Tot i que els fitxers XML estan essent substituïts en alguns camps de la informàtica per formats més lleugers (per exemple, el format JSON en el món web), encara s'utilitzen molt per emmagatzemar dades i, sobretot, per crear fitxers de configuració d'aplicacions.

La seva lectura, escriptura i modificació, però, no es fa utilitzant algorismes clàssics (tractant línia a línia), sinó que existeixen dos tipus de *parsers* que ajuden a tractar tota la seva estructura DOM (*Document Object Model*). El *parser* més bàsic és el SAX (*Simple API for XML*) i el més complet és el *DOM Parser*.

**SAX Parser**

Aquest *parser* està basat en el tractament d'events, els quals s'originen a mesura que es va llegint el document per detectar les etiquetes, els atributs i els seus valors.

La lectura de l'XML es fa de manera seqüencial, sense carregar-lo a memòria, la qual cosa fa que sigui molt eficient. Ara però, aquesta restricció fa que el *SAX Parser* només serveixi per
1. llegir un document XML i
2. comprovar si la seva estructura és correcta.

En cap cas pot crear un nou fitxer XML, afegir-hi nou contingut o modificar el que ja té.

**DOM Parser**

Per poder fer el tractament d'un fitxer XML, el *DOM Parser* necessita carregar tota la seva estructura (jerarquia en forma d'arbre) a *RAM*, la qual cosa el fa més flexible pel que fa a les funcionalitats que pot oferir, tenint en compte que és més lent d'execució.

Amb el *DOM Parser* es poden crear nous documents XML, afegir-hi nou contingut i modificar-ne el ja existent.

## *SAX Parser*

Per poder implementar un *SAX Parser* s'ha de crear un gestor SAX que sigui subclasse de [`DefaultHandler`](https://docs.oracle.com/en/java/javase/25/docs/api/java.xml/org/xml/sax/helpers/DefaultHandler.html) (*package* `org.xml.sax.helpers`). Aquest gestor (*handler*) implementarà els mètodes de la seva superclasse adaptant-los a les etiquetes, atributs i valors presents en el document XML que cal tractar. Això implica que, per cada document XML s'haurà de crear un *handler* adaptat, específicament, a la seva estructura, és a dir, no hi ha un *Sax Parser* general que serveixi per tots els casos.

En concret, la nova subclasse haurà de sobreescriure els mètodes heretats més bàsics, els quals són els següents:
* `startDocument()`: és el *callback* (el mètode) que s'executa quan es detecta l'inici d'un document XML
* `startElement(String uri, String localName, String qName, Attributes attributes)`: és el *callback* que s'executa cada cop que comença un nou element (una nova etiqueta). El valor dels atributs és el següent:
    * `uri`: el prefix de l'etiqueta corresponent al *Namespace* del document
    * `localName`: el nom de l'etiqueta
    * `qName`: la concatenació del prefix i del nom de l'etiqueta
    * `attributes`: els atributs associats a l'etiqueta
* `characters(char[] ch, int start, int length)`: és el *callback* que s'executa per tractar el valor dels diferents elements de l'estructura XML
* `endElement(String uri, String localName, String qName)`: és el *callback** que s'executa cada cop que s'acaba un element (els seus atributs són els mateixos que rep el mètode `startElement()`, exceptuant l'atribut `attributes`)
* `endDocument()`: és el *callback* que s'executa quan finalitza el document XML

Imaginem que tenim el següent document XML amb el seu DTD (*Document Type Definition*) corresponent:

{% tabs %}
{% tab title="DTD tournament.dtd" %}
```dtd
<!ELEMENT tournament (name, videogame, player+)>

<!ELEMENT name (#PCDATA)>
<!ELEMENT videogame (#PCDATA)>

<!ELEMENT player (name, team, points)>
<!ATTLIST player id ID #REQUIRED>

<!ELEMENT team (#PCDATA)>
<!ELEMENT points (#PCDATA)>
```
{% endtab %}
{% tab title="Document XML tournament.xml" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE tournament SYSTEM "tournament.dtd">

<tournament>
    <name>Gaming Cup 2026</name>
    <videogame>Valorant</videogame>

    <player id="P01">
        <name>Laia</name>
        <team>Pixel Warriors</team>
        <points>250</points>
    </player>

    <player id="P02">
        <name>Marc</name>
        <team>Cyber Dragons</team>
        <points>320</points>
    </player>
</tournament>
```
{% endtab %}
{% endtabs %}

El procés per per poder-ne llegir les dades consisteix a seguir els passos següents:
1. Crear les classes `Player` i `Tornament` per tal que puguin representar i emmagatzemar les dades contingudes en el fitxer.
2. Crear el *handler* `TournamentSAXHandler` per tal que pugui tractar les etiquetes `tournament`, `name`, `videogame`, `player`, `team` i `points`.
3. Crear un objecte `XMLReader` i proporcionar-li el `TounamentSAXHandler` per tal que pugui llegir el document `tournament.xml`. Per crear aquest objecte fa falta
    1. crear una instància de `SAXParserFactory` i
    2. demanar a la `SAXParserFactory` una instància d'un `SAXParser`

A continuació es mostra i s'explica tot el codi necessari, tenint en compte que no s'ha aplicat cap patró de disseny per fer l'exemple.

**Classes `Player` i `Tounament`**

{% tabs %}
{% tab title="Classe Player" %}
```java
package cat.inscaparrella.dam2.model;

public class Player {
    private String id;
    private String name;
    private String team;
    private int points;

    public Player() {
        this.id = "";
        this.name = "";
        this.team = "";
        this.points = 0;
    }

    public void setId(String id) { this.id = id; }
    public void setName(String name) { this.name = name; }
    public void setTeam(String team) { this.team = team; }
    public void setPoints(int points) { this.points = points; }

    @Override
    public String toString() {
        return "Player [ id=" + id +
                ", name=" + name +
                ", team=" + team +
                ", points=" + points + " ]";
    }
}
```
{% endtab %}
{% tab title="Classe Tournament" %}
```java
package cat.inscaparrella.dam2.model;

import java.util.ArrayList;

public class Tournament {
    private String name;
    private String videogame;
    private ArrayList<Player> players;

    public Tournament() {
        this.name = "";
        this.videogame = "";
        this.players = new ArrayList<>();
    }

    public void setName(String name) { this.name = name; }
    public void setVideogame(String videogame) { this.videogame = videogame; }
    public void addPlayer(Player player) { this.players.add(player); }

    @Override
    public String toString() {
        return "Tournament [ name=" + name +
                ", videogame=" + videogame +
                ", players=" + players.toString() + " ]";
    }
}
```
{% endtab %}
{% endtabs %}

Com es pot observar, són classes senzilles i bàsiques l'únic objectiu de les quals poder representar les entitats del fitxer.

**Classe `TournamentSAXHandler`**

```java
package cat.inscaparrella.dam2.utils;

import cat.inscaparrella.dam2.model.Player;
import cat.inscaparrella.dam2.model.Tournament;
import org.xml.sax.Attributes;
import org.xml.sax.helpers.DefaultHandler;

public class TournamentSAXHandler extends DefaultHandler {
    private static final String TOURNAMENT = "tournament";
    private static final String NAME = "name";
    private static final String VIDEOGAME = "videogame";
    private static final String PLAYER = "player";
    private static final String TEAM = "team";
    private static final String POINTS = "points";
    private static final String ID = "id";

    private Tournament tournament;
    private Player player;
    private StringBuilder content;
    private String lastElementStarted;

    @Override
    public void startDocument() {
        this.tournament = new Tournament();
    }

    @Override
    public void startElement(String uri, String lname, String qname, Attributes attributes) {
        switch(qname) {
            case TOURNAMENT:
                this.lastElementStarted = TOURNAMENT;
                break;
            case NAME:
            case VIDEOGAME:
            case TEAM:
            case POINTS:
                this.content = new StringBuilder();
                break;
            case PLAYER:
                this.player = new Player();
                this.player.setId(attributes.getValue(0));
                this.lastElementStarted = PLAYER;
                break;
        }
    }

    @Override
    public void endElement(String uri, String lname, String qname) {
        switch(qname) {
            case NAME:
                if(this.lastElementStarted.equals(TOURNAMENT)) {
                    this.tournament.setName(this.content.toString());
                } else if(this.lastElementStarted.equals(PLAYER)) {
                    this.player.setName(this.content.toString());
                }
                break;
            case VIDEOGAME:
                this.tournament.setVideogame(this.content.toString());
                break;
            case PLAYER:
                this.tournament.addPlayer(player);
                break;
            case TEAM:
                this.player.setTeam(this.content.toString());
                break;
            case POINTS:
                this.player.setPoints(Integer.parseInt(this.content.toString()));
                break;
        }
    }

    @Override
    public void characters(char[] characters, int start, int length) {
        this.content.append(characters, start, length);
    }

    public Tournament getTournament() {
        return this.tournament;
    }
}
```

Els atributs d'aquesta classe, a part de les constats d'ajuda, són:
* un objecte de tipus `Tournament` i un objecte de tipus `Player`, els quals s'aniran omplint amb les dades del document;
* un objecte `StringBuilder` que anirà emmagatzemant els valors de cada etiqueta i
* un objecte `String` (`lastElementStarted`) que indicarà quina és l'última etiqueta llegida, si `<tournament>` o `<player>`; aquest atribut es necessita per poder diferenciar si les diverses etiquetes `name` pertanyen a l'objecte `Tournament` o a l'objecte `Player`.

El mètode `startDocument()` només s'ha d'encarregar d'inicialitzar l'objecte `Tournament` i deixar-lo preparat per anar-hi carregant les dades.

El mètode `startElement()` ha de diferenciar quin tipus d'etiqueta s'està tractant actualment, de tal manera que
1. si l'etiqueta és `<tournament>` s'ha d'actualitzar l'atribut `lastElementStarted` al valor `tournament` per deixar marcat que, ara mateix, s'està tractant aquest tipus objecte i les seves dades;
2. si l'etiqueta és `<player>`
    1. s'ha de crear un nou objecte `Player`;
    2. s'ha de carregar l'atribut `id` a l'objecte `Player` utilitzant el *setter* corresponent i
    3. s'ha d'actualitzar l'atribut `lastElementStarted` al valor `player` per deixar marcat que, ara mateix, s'està tractant aquest tipus objecte i les seves dades;
3. si l'etiqueta és `<name>`, `<videogame>`, `<team>` o `<points>` s'ha de crear un nou `StringBuilder` i deixar-lo preparat per a carregar-hi el valor de l'element.

El mètode `endElement()` també ha de diferenciar el tipus d'etiqueta que s'està tractant per tal de poder carregar les dades correctament, Així doncs
1. si l'etiqueta és `<name>` caldrla analitzar el valor de l'atribut `lastElementStarted` per tal de saber si correspon al nom del torneig o del jugador; fet això ja es podrà cridar al mètode *setter* de l'objecte correcte;
2. si l'etiqueta és `<videogame>` només farà falta cridar al *setter* corresponent de l'objecte `Tournament`;
3. si l'etiqueta és `<player>` es cridarà al mètode `addPlayer()` de l'objecte `Tournament`, el qual afegirà el nou jugador al torneig, i
4. si l'etiqueta és `<team>` o `<points>` es cridarà al *setter* corresponent de l'objecte `Player`.

El mètode `characters()` s'encarrega de capturar el valor de les diverses etiquetes i emmagatzemar-lo dins de l'objecte `StringBuilder`.

El mètode `getTournament()` permet retornar les dades del torneig acabat de llegir del fitxer.

**Lectura del fitxer amb un XMLReader**

```java
package cat.inscaparrella.dam2.view;

import cat.inscaparrella.dam2.utils.TournamentSAXHandler;
import org.xml.sax.SAXException;
import org.xml.sax.XMLReader;
import org.xml.sax.helpers.DefaultHandler;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;
import java.io.File;
import java.io.IOException;

public class TorunamentSAXParserMain {

    private static final String XML_PATH = "xml_files" + File.separator + "tournament.xml";

    public static void main() throws ParserConfigurationException, SAXException, IOException {
        SAXParserFactory saxFactory = SAXParserFactory.newInstance();
        SAXParser saxParser = saxFactory.newSAXParser();
        XMLReader saxReader = saxParser.getXMLReader();

        DefaultHandler saxHandler = new TournamentSAXHandler();
        saxReader.setContentHandler(saxHandler);
        saxReader.parse(XML_PATH);

        System.out.println(((TournamentSAXHandler) saxHandler).getTournament().toString());
    }
}
```

Com es pot veure al codi, per poder obtenir un objecte de tipus `XMLReader` cal invocar a `SAXParserFactory` per tal d'obtenir un `SAXParser` i, finalment, demanar-li l'`XMLReader`:
```java
SAXParserFactory saxFactory = SAXParserFactory.newInstance();
SAXParser saxParser = saxFactory.newSAXParser();
XMLReader saxReader = saxParser.getXMLReader();
```

Un cop obtingut l'`XMLReader` se li ha de configurar el *handler* necessari per poder llegir el document XML desitjat:
```java
DefaultHandler saxHandler = new TournamentSAXHandler();
saxReader.setContentHandler(saxHandler);
```

Fet això, ja es pot llegir (o *parsejar*) el fitxer:
```java
saxReader.parse(XML_PATH);
```

{% hint style="warning" icon="pen" %}
**Exercici**

A sota d'aquest enunciat es mostra un fitxer XML i el seu DTD.

Implementa un programa que permeti
1. mostrar les dades de la plataforma;
2. mostrar el catàleg de tots els videojocs;
3. filtrar els videojocs que estan disponibles;
4. mostrar tots els jugadors ordenats segons el seu nivell i
5. transformar l'XML en un CSV de videojocs amb la capçalera següent:
`ID;AVALIABLE;TITLE;DEVELOPER;RELEASE_YEAR;GENRE;PLATFORMS`

Per exemple:
```
ID;AVALIABLE;TITLE;DEVELOPER;RELEASE_YEAR;GENRE;PLATFORMS
G01;SÍ;Cyberpunk;CD PROJEKT RED;2020;Action RPG;[PC,PlayStation 5,XboxSeries X|S]
```
{% endhint %}

{% tabs %}
{% tab title="DTD gaming.dtd" %}
```dtd
<!ELEMENT gaming (service, videogames, players)>

<!ELEMENT service (name, country, creationYear)>
<!ELEMENT name (#PCDATA)>
<!ELEMENT country (#PCDATA)>
<!ELEMENT creationYear (#PCDATA)>

<!ELEMENT videogames (videogame+)>

<!ELEMENT videogame (title, developer, releaseYear, genre, platforms)>
<!ATTLIST videogame
    id ID #REQUIRED
    available (true | false) #REQUIRED
>

<!ELEMENT title (#PCDATA)>
<!ELEMENT developer (#PCDATA)>
<!ELEMENT releaseYear (#PCDATA)>
<!ELEMENT genre (#PCDATA)>

<!ELEMENT platforms (platform+)>
<!ELEMENT platform (#PCDATA)>

<!ELEMENT players (player+)>

<!ELEMENT player (nickname, level, favoriteGame)>
<!ATTLIST player id ID #REQUIRED>

<!ELEMENT nickname (#PCDATA)>
<!ELEMENT level (#PCDATA)>
<!ELEMENT favoriteGame (#PCDATA)>
```
{% endtab %}
{% tab title="Document XML gaming.xml" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE gaming SYSTEM "gaming.dtd">

<gaming>
    <service>
        <name>GameZone</name>
        <country>Spain</country>
        <creationYear>2022</creationYear>
    </service>

    <videogames>
        <videogame id="G01" available="true">
            <title>Cyberpunk 2077</title>
            <developer>CD PROJEKT RED</developer>
            <releaseYear>2020</releaseYear>
            <genre>Action RPG</genre>

            <platforms>
                <platform>PC</platform>
                <platform>PlayStation 5</platform>
                <platform>Xbox Series X|S</platform>
            </platforms>
        </videogame>

        <videogame id="G02" available="true">
            <title>Minecraft</title>
            <developer>Mojang Studios</developer>
            <releaseYear>2011</releaseYear>
            <genre>Sandbox</genre>

            <platforms>
                <platform>PC</platform>
                <platform>PlayStation 5</platform>
                <platform>Xbox Series X|S</platform>
                <platform>Nintendo Switch</platform>
            </platforms>
        </videogame>

        <videogame id="G03" available="true">
            <title>Fortnite</title>
            <developer>Epic Games</developer>
            <releaseYear>2017</releaseYear>
            <genre>Battle Royale</genre>

            <platforms>
                <platform>PC</platform>
                <platform>PlayStation 5</platform>
                <platform>Xbox Series X|S</platform>
                <platform>Nintendo Switch</platform>
            </platforms>
        </videogame>
    </videogames>

    <players>
        <player id="P01">
            <nickname>ShadowNinja</nickname>
            <level>42</level>
            <favoriteGame>G01</favoriteGame>
        </player>

        <player id="P02">
            <nickname>AlexCraft</nickname>
            <level>27</level>
            <favoriteGame>G02</favoriteGame>
        </player>

        <player id="P03">
            <nickname>VictoryRoyale</nickname>
            <level>56</level>
            <favoriteGame>G03</favoriteGame>
        </player>
    </players>
</gaming>
```
{% endtab %}
{% endtabs %}

## *DOM Parser*
Tal com s'ha indicat anteriorment, el *DOM Parser* pot llegir, crear i modificar fitxers XML però, per poder-ho fer, necessita tenir carregat tot el document a RAM, la qual cosa fa que sigui un *parser* més flexible, però també més lent.

Per fer l'explicació d'aquest apartat s'utilitzarà el document XML que es mostra a continuació amb la seva DTD corresponent.

{% tabs %}
{% tab title="DTD festivals.dtd" %}
```dtd
<!ELEMENT festivals (festival+)>
<!ATTLIST festivals year CDATA #REQUIRED>

<!ELEMENT festival (name, city, venue, dates, price, genres, concerts)>
<!ATTLIST festival id ID #REQUIRED>

<!ELEMENT name (#PCDATA)>
<!ELEMENT city (#PCDATA)>
<!ELEMENT venue (#PCDATA)>

<!ELEMENT dates (start, end)>
<!ELEMENT start (#PCDATA)>
<!ELEMENT end (#PCDATA)>

<!ELEMENT price (#PCDATA)>
<!ATTLIST price currency CDATA #REQUIRED>

<!ELEMENT genres (genre+)>
<!ELEMENT genre (#PCDATA)>

<!ELEMENT concerts (concert+)>

<!ELEMENT concert (artist, stage, time, duration)>
<!ELEMENT artist (#PCDATA)>
<!ELEMENT stage (#PCDATA)>
<!ELEMENT time (#PCDATA)>
<!ELEMENT duration (#PCDATA)>
```
{% endtab %}
{% tab title="Document XML festivals.xml" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE festivals SYSTEM "festivals.dtd">

<festivals year="2026">
    <festival id="F001">
        <name>Primavera Sound</name>
        <city>Barcelona</city>
        <venue>Parc del Fòrum</venue>
        <dates>
            <start>2026-06-04</start>
            <end>2026-06-06</end>
        </dates>
        <price currency="EUR">185.00</price>
        <genres>
            <genre>Indie</genre>
            <genre>Rock</genre>
            <genre>Electrònica</genre>
        </genres>
        <concerts>
            <concert>
                <artist>Arctic Monkeys</artist>
                <stage>Estrella Damm</stage>
                <time>22:30</time>
                <duration>90</duration>
            </concert>
            <concert>
                <artist>Rosalía</artist>
                <stage>Amazon Music</stage>
                <time>00:30</time>
                <duration>75</duration>
            </concert>
            <concert>
                <artist>Fred again..</artist>
                <stage>Pull&amp;Bear</stage>
                <time>02:00</time>
                <duration>90</duration>
            </concert>
        </concerts>
    </festival>

    <festival id="F002">
        <name>Viña Rock</name>
        <city>Villarrobledo</city>
        <venue>Recinte firal</venue>
        <dates>
            <start>2026-04-30</start>
            <end>2026-05-02</end>
        </dates>
        <price currency="EUR">75.00</price>
        <genres>
            <genre>Rock</genre>
            <genre>Punk</genre>
            <genre>Hip-hop</genre>
        </genres>
        <concerts>
            <concert>
                <artist>La Raíz</artist>
                <stage>Negrita</stage>
                <time>21:00</time>
                <duration>80</duration>
            </concert>
            <concert>
                <artist>Green Valley</artist>
                <stage>Villarrobledo</stage>
                <time>23:00</time>
                <duration>75</duration>
            </concert>
            <concert>
                <artist>Natos y Waor</artist>
                <stage>Negrita</stage>
                <time>01:00</time>
                <duration>90</duration>
            </concert>
        </concerts>
    </festival>

    <festival id="F003">
        <name>Sónar</name>
        <city>Barcelona</city>
        <venue>Fira Barcelona</venue>
        <dates>
            <start>2026-06-18</start>
            <end>2026-06-20</end>
        </dates>
        <price currency="EUR">160.00</price>
        <genres>
            <genre>Techno</genre>
            <genre>House</genre>
            <genre>Experimental</genre>
        </genres>
        <concerts>
            <concert>
                <artist>Amelie Lens</artist>
                <stage>SónarClub</stage>
                <time>23:30</time>
                <duration>90</duration>
            </concert>
            <concert>
                <artist>Charlotte de Witte</artist>
                <stage>SónarPub</stage>
                <time>01:00</time>
                <duration>90</duration>
            </concert>
            <concert>
                <artist>Anyma</artist>
                <stage>SónarLab</stage>
                <time>03:00</time>
                <duration>75</duration>
            </concert>
        </concerts>
    </festival>
</festivals>
```
{% endtab %}
{% endtabs %}

### Càrrega del fitxer XML a memòria
Si es vol llegir o modificar un fitxer XML ja existent, el primer pas que cal fer sempre és carregar-lo a memòria. Per aconseguir aquest objectiu es necessiten les classes següents:
1. `DocumentBuilderFactory`: és la *factoria* que permet crear el *parser* DOM.
2. `DocumentBuilder`: la seva responsabilitat és carregar a memòria l'estructura d'arbre corresponent al fitxer XML que s'està gestionant.
3. `Document`: representa tot el document XML, al qual accedeix a través del seu element arrel, i ofereix totes les funcions necessàries per recórrer l'estructura d'arbre i crear, modificar o eliminar nodes (elements XML).

Tot plegat es combina tal com mostra el codi següent:
```java
package cat.inscaparrella.dam2.utils;

import org.w3c.dom.Document;
import org.xml.sax.SAXException;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import java.io.IOException;

public class FestivalsDOMParser {
    private Document xmlDoc;
    
    public void loadXMLFile(String path) throws ParserConfigurationException, SAXException, IOException {
        DocumentBuilderFactory dfactory = DocumentBuilderFactory.newInstance();
        DocumentBuilder dbuilder = dfactory.newDocumentBuilder();
        this.xmlDoc = dbuilder.parse(path);

        this.xmlDoc.getDocumentElement().normalize();
    }
}

```

Les dues primeres línies del mètode `loadXMLFile()` creen els objectes `DocumentBuilderFactory` i `DocumentBuilder`. Tot seguit, i a partir del `DocumentBuilder`, es crea el `Document` fent un `parse()` del fitxer XML, és a dir, carregant-ne tots els nodes (elements) a memòria i retornant la referència al node arrel, a través del qual es podrà recórrer la resta.

L'última instrucció del mètode accedeix al contingut del node arrel i normalitza el document. La normalització consisteix en eliminar nodes buits i fusionar nodes equivalents per tal d'evitar dades duplicades. A continuació es mostra un exemple d'aquests procés:

{% tabs %}
{% tab title="Document XML festivals.xml no normalitzat" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<festivals>
    <festival id="F001">
        <name>Primavera Sound</name>
        <city>Barcelona</city>
        <concerts>
            <concert>
                <artist>Arctic Monkeys</artist>
                <genre>Rock</genre>
                <country>United Kingdom</country>
            </concert>
            <concert>
                <artist>Rosalía</artist>
                <genre>Pop</genre>
                <country>Spain</country>
            </concert>
        </concerts>
    </festival>

    <festival id="F002">
        <name>Viña Rock</name>
        <city>Villarrobledo</city>
        <concerts>
            <concert>
                <artist>Rosalía</artist>
                <genre>Pop</genre>
                <country>Spain</country>
            </concert>
            <concert>
                <artist>Arctic Monkeys</artist>
                <genre>Rock</genre>
                <country>United Kingdom</country>
            </concert>
        </concerts>
    </festival>
</festivals>
```
{% endtab %}
{% tab title="Document XML festivals.xml normalitzat" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<festivalData>
    <festivals>
        <festival id="F001">
            <name>Primavera Sound</name>
            <city>Barcelona</city>
        </festival>

        <festival id="F002">
            <name>Viña Rock</name>
            <city>Villarrobledo</city>
        </festival>
    </festivals>

    <artists>
        <artist id="A001">
            <name>Arctic Monkeys</name>
            <genre>Rock</genre>
            <country>United Kingdom</country>
        </artist>

        <artist id="A002">
            <name>Rosalía</name>
            <genre>Pop</genre>
            <country>Spain</country>
        </artist>
    </artists>

    <concerts>
        <concert id="C001">
            <festivalRef>F001</festivalRef>
            <artistRef>A001</artistRef>
        </concert>

        <concert id="C002">
            <festivalRef>F001</festivalRef>
            <artistRef>A002</artistRef>
        </concert>

        <concert id="C003">
            <festivalRef>F002</festivalRef>
            <artistRef>A002</artistRef>
        </concert>

        <concert id="C004">
            <festivalRef>F002</festivalRef>
            <artistRef>A001</artistRef>
        </concert>
    </concerts>
</festivalData>
```
{% endtab %}
{% endtabs %}

Com es pot veure, en el primer cas, les dades dels artistes *Arctic Monkeys* i *Rosalía* aparéixen duplicades perquè participen tant en el *Primavera Sound* com en el *Viña Rock*. Això implica que, en cas de voler modificar o esborrar les dades d'aquesta cantant, s'haurà de recórrer tot el document per trobar-ne totes les instàncies. Si no es fa així, el fitxer acabarà contenint dades obsoletes o inconsistents.

En canvi, en el segon cas, els elements del document s'han reestructurat per tal d'evitar la repetició de la informació (és un procés similar al de la normalització de les bases de dades relacionals).

{% hint style="warning" %}
**Atenció**

La normalització d'un document XML pot comportar canvis importants en la seva estructura, per tant, també caldrà modificar la DTD (o *Schema*) associada.
{% endhint %}

Evidentment, el mètode `normalize()` no fa canvis estructurals tant radicals com els que s'han mostrat en l'exemple de normalització d'un document XML. Ara però, el que sí que fa és el següent:
1. Elimina espais en blanc abans o després del node arrel (...<festivals></docufestivalsment>...)
2. Elimina espais en blanc dins de les etiquetes d'obertura (<...>) i tancament (</...>)
3. Elimina espais en blanc entre els atributs i els seus valors
4. Elimina declaracions de *namespace* innecessàries
5. Elimina salts de línia i espais en text dels atributs, les etiquetes, els comentaris, etc.

### Guardar el fitxer XML
Si es vol modificar un fitxer XML ja existent o crear-ne un de nou, caldrà, sí o sí, acabar guardant-lo en memòria permanent (en un fitxer físic). Per aconseguir aquest objectiu es necessiten les classes següents:
1. `DOMSource`: fa de suport per poder transformar l'arbre DOM que hi ha emmagatzemat en memòria RAM a un fitxer físic.
2. `TransformerFactory`: és la *factoria* que permet crear el *transformador* DOM.
3. `Transformer`: la seva responsabilitat és transformar l'arbre DOM al resultat desitjat, normalment, un fitxer físic, amb l'ajuda del `DOMSource`.
4. `StreamResult`: gestiona el flux de dades que cal escriure a fitxer un cop el DOM de l'XML ha estat transformat.

Tot plegat es combina tal com mostra el codi següent:
```java
package cat.inscaparrella.dam2.utils;

import org.w3c.dom.Document;
import org.xml.sax.SAXException;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerException;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.IOException;

public class FestivalsDOMParser {
    private Document xmlDoc;

    public void loadXMLFile(String path) throws ParserConfigurationException, SAXException, IOException {...}

    public void saveXMLFile(String path) throws TransformerException {
        DOMSource source = new DOMSource(this.xmlDoc);
        TransformerFactory tfactory = TransformerFactory.newInstance();
        Transformer transformer = tfactory.newTransformer();
        
        StreamResult result = new StreamResult(path);
        transformer.transform(source, result);
    }
```

Les tres primeres línies del mètode `saveXMLFile()` creen els objectes `DOMSource`, `TransformerFactory` i `Transformer`. A continuació es crea el flux de dades que s'han d'escriure a fitxer `StreamResult`. Finalment, s'obté el fitxer físic utilitzant el gestor del DOM `DOMSource` i el flux de dades `StreamResult`.

### Lectura del fitxer XML (recorregut dels seus elements)
Per poder llegir el document s'ha d'utiltizar el mètode `getElementsByTagName()` de la classe `Document`. Aquest mètode retorna tots els elements (etiquetes) d'un tipus dins d'un objecte de tipus `NodeList`, que no és res més que un llistat d'objectes `Node` (cada `Node` representa un element o etiqueta).

El llistat de nodes `NodeList` es pot recórrer iterativament sabent-ne la seva mida mitjançant el mètode `getLength()` i accedint a cada `Node` a través del mètode `item()`. A partir de cada objecte `Node` es pot
1. accedir als seus fills a través dels mètodes `getChildNodes()` (retorna un `NodeList`), `getFirstChild()` (retorna un `Node`) i `getLastChild()` (retorna un `Node`);
2. accedir al seu `Node` *parent* amb el mètode `getParentNode()`;
3. obtenir-ne el tipus (etiqueta, atribut, etc.) mitjançant el mètode `getNodeType()`;
4. obtenir-ne el nom amb el mètode `getNodeName()`;
5. obtenir-ne el valor amb el mètode `getTextContent()`;
6. obtenir-ne els atributs amb el mètode `getAttributes()` (retorna un `NamedNodeMap`, un *map* de nodes) i
7. accedir a nodes *germans* (nodes que es troben dins del mateix nivell de l'arbre DOM i pengen del mateix `Node` *parent*) mitjançant els mètodes `getNextSibling()` i `getPreviousSibling()`.

Per veure altres mètodes de consulta de la classe `Node`, consulteu l'API de [`Node`](https://docs.oracle.com/en/java/javase/25/docs/api/java.xml/org/w3c/dom/Node.html) i de [`NamedNodeMap`](https://docs.oracle.com/en/java/javase/25/docs/api/java.xml/org/w3c/dom/NamedNodeMap.html).

A continuació es mostra el codi necessari per recórrer tot el fitxer `festivals.xml` (utilitza una funció recursiva per accedir als `childs` dels diversos nodes)

{% tabs %}
{% tab title="Programa FestivalsDOMParserMain.java" %}
```java
package cat.inscaparrella.dam2.view;

import cat.inscaparrella.dam2.utils.FestivalsDOMParser;
import org.xml.sax.SAXException;

import javax.xml.parsers.ParserConfigurationException;
import java.io.IOException;

public class FestivalsDOMParserMain {
    public static void main() throws ParserConfigurationException, SAXException, IOException {
        FestivalsDOMParser domParser = new FestivalsDOMParser();
        domParser.loadXMLFile("xml_files/festivals.xml");
        System.out.println(domParser.toString());
    }
}
```
{% endtab %}
{% tab title="FestivalDOMParser.java" %}
```java
package cat.inscaparrella.dam2.utils;

import org.w3c.dom.Document;
import org.w3c.dom.NamedNodeMap;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;
import org.xml.sax.SAXException;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerException;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.IOException;

public class FestivalsDOMParser {
    private Document xmlDoc;

    public void loadXMLFile(String path) throws ParserConfigurationException, SAXException, IOException {...}

    public void saveXMLFile(String path) throws TransformerException {...}

    @Override
    public String toString() {
        NodeList nodes = this.xmlDoc.getElementsByTagName("festivals");
        return this.traverseNodeList(nodes, "");
    }

    private String traverseNodeList(NodeList nodes, String prefix) {
        String str = "";

        Node currentNode;

        NamedNodeMap attributes;
        Node currentAttribute;

        for(int i=0; i<nodes.getLength(); i++) {
            currentNode = nodes.item(i);
            str += this.getNodeStringData(currentNode, prefix);

            if(currentNode.hasAttributes()) {
                attributes = currentNode.getAttributes();
                for(int j=0; j<attributes.getLength(); j++) {
                    currentAttribute = attributes.item(i);
                    if(currentAttribute != null)
                        str += this.getNodeStringData(currentAttribute, prefix + "\t\t");
                }
            }
            if(currentNode.hasChildNodes()) {
                str += this.traverseNodeList(currentNode.getChildNodes(), prefix + "\t");
            }
        }

        return str;
    }

    private String getNodeStringData(Node node, String prefix) {
        if(node.getNodeType() == Node.TEXT_NODE) return "";

        String str = prefix + node.getNodeName();
        short nodeType;

        nodeType = node.getNodeType();
        if(nodeType == Node.ELEMENT_NODE) {
            str += " (ETIQUETA)";
            if(node.getChildNodes().getLength() > 0
                    && node.getChildNodes().item(0).getNodeType() == Node.TEXT_NODE
                    && !node.getChildNodes().item(0).getTextContent().isBlank()) {
                //Text content will allways be the first child
                str += ": " + node.getChildNodes().item(0).getTextContent();
            }
        } else if(nodeType == Node.ATTRIBUTE_NODE && node.getChildNodes().item(0).getNodeType() == Node.TEXT_NODE) {
            str += " (ATRIBUT)";
            if(node.getChildNodes().getLength() > 0) {
                //Text content will allways be the first child
                str += ": " + node.getChildNodes().item(0).getTextContent();
            }
        }

        return str + "\n";
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" icon="pen" %}
**Exercici**

El codi exemplificat a sobre d'aquest enunciat és força complex perquè utilitza la recursivitat per recórrer tots els nodes (elements) niats dins del document XML de manera automàtica. Ara però, si la consulta que es vol fer és molt més precisa, el codi es simplifica molt.

Donat el fitxer `festivals.xml` que s'utilitza per il·lustrar el capítol, crea un programa que mostri per pantalla quans festivals té emmagatzemats. Per cada fesival haurà de mostrar-ne el nom i la ciutat on es farà.

La sortida haurà de ser una cosa similar a la següent:
```txt
Nombre de festivals: 3

Primavera Sound - Barcelona
Viña Rock - Villarrobledo
Sónar - Barcelona
```

Una possible ajuda per fer l'exercici sense haver de recórrer tots els *childs* d'un node. Si, donat un `Node` feu cast a [`Element`](https://docs.oracle.com/en/java/javase/25/docs/api/java.xml/org/w3c/dom/Element.html) tindreu accés directe als mètodes `getElementsByTagName()`, `getAttributeNode()`, `getAttribute()`, entre altres. Això pot facilitar la implementació de l'exercici perquè permet accedir a les etiquetes i atributs a través del seu nom més directament.
{% endhint %}

### Modificació del fitxer XML
Per exemplificar la modificació de les dades d'un fitxer XML ens centrarem en
1. canviar el valor d'una dada ja existent i
2. eliminar un element (etiqueta o node).

No s'exemplificarà cap modificació que impliqui afegir nous elements perquè es pot extrapolar aquest tipus de canvi a partir de la creació d'un nou fitxer XML, que s'explica en el següent apartat.

#### Canviar el valor d'una dada ja existent
Suposem que volem canviar el preu del *Primavera Sound*; per fer-ho caldrà
1. accedir al festival que té aquest *name*;
2. accedir al seu *price* i modificar-lo i 
3. guardar el document amb les noves dades

El codi que assoleix aquest objectiu és el següent:

{% tabs %}
{% tab title="Programa FestivalsDOMParserMain.java" %}
```java
package cat.inscaparrella.dam2.view;

import cat.inscaparrella.dam2.utils.FestivalsDOMParser;
import org.xml.sax.SAXException;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.TransformerException;
import java.io.IOException;

public class FestivalsDOMParserMain {
    public static void main() throws ParserConfigurationException, SAXException, IOException, TransformerException {
        FestivalsDOMParser domParser = new FestivalsDOMParser();
        domParser.loadXMLFile("xml_files/festivals.xml");
        domParser.changeFestivalPrice("Primavera Sound", 90.0f);
        domParser.saveXMLFile("xml_files/changed_festivals.xml");
    }
}
```
{% endtab %}
{% tab title="FestivalDOMParser.java" %}
```java
package cat.inscaparrella.dam2.utils;

import org.w3c.dom.*;
import org.xml.sax.SAXException;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerException;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.IOException;

public class FestivalsDOMParser {
    private Document xmlDoc;

    public void loadXMLFile(String path) throws ParserConfigurationException, SAXException, IOException {...}

    public void changeFestivalPrice(String festivalName, float price) {
        NodeList nodes = this.xmlDoc.getElementsByTagName("festival");
        Element festivalNode;
        Node nameNode, priceNode;

        int i=0;
        boolean found = false;
        while(i<nodes.getLength() && !found) {
            festivalNode = (Element) nodes.item(i);
            nameNode = festivalNode.getElementsByTagName("name").item(0);
            if(nameNode.getTextContent().equals(festivalName)) {
                priceNode = festivalNode.getElementsByTagName("price").item(0);
                priceNode.setTextContent(Float.toString(price));

                found = true;
            }
            i++;
        }
    }

    public void saveXMLFile(String path) throws TransformerException {...}

    @Override
    public String toString() {...}

    private String traverseNodeList(NodeList nodes, String prefix) {...}

    private String getNodeStringData(Node node, String prefix) {...}
}
```
{% endtab %}
{% endtabs %}

La clau que fa que aquest codi no hagi de recórrer tots els nodes és el *cast* que es fa com a primera instrucció dins de la sentència `for`. En passar el `Node` a `Element` s'obté accés als mètode `getElementsByTagName()`, la qual cosa fa que es puguin demanar, directament, les etiquetes `name` i `price` de cada etiqueta `festival` (sense necessitat de visitar tots els *child*).


#### Eliminar un element
Suposem que volem eliminat el festival *Sónar*ç; per fer-ho caldrà
1. accedir al festival que té aquest *name*;
2. eliminar-lo dels *child* de l'etiqueta *festivals* i
3. guardar el document amb les noves dades

El codi que assoleix aquest objectiu és el següent:

{% tabs %}
{% tab title="Programa FestivalsDOMParserMain.java" %}
```java
package cat.inscaparrella.dam2.view;

import cat.inscaparrella.dam2.utils.FestivalsDOMParser;
import org.xml.sax.SAXException;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.TransformerException;
import java.io.IOException;

public class FestivalsDOMParserMain {
    public static void main() throws ParserConfigurationException, SAXException, IOException, TransformerException {
        FestivalsDOMParser domParser = new FestivalsDOMParser();
        domParser.loadXMLFile("xml_files/festivals.xml");
        domParser.deleteFestival("Sónar");
        domParser.saveXMLFile("xml_files/deleted_festivals.xml");
    }
}
```
{% endtab %}
{% tab title="FestivalDOMParser.java" %}
```java
package cat.inscaparrella.dam2.utils;

import org.w3c.dom.*;
import org.xml.sax.SAXException;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerException;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.io.IOException;

public class FestivalsDOMParser {
    private Document xmlDoc;

    public void loadXMLFile(String path) throws ParserConfigurationException, SAXException, IOException {...}

    public void changeFestivalPrice(String festivalName, float price) {...}

    public void deleteFestival(String festivalName) {
        Node root = this.xmlDoc.getElementsByTagName("festivals").item(0);

        NodeList nodes = this.xmlDoc.getElementsByTagName("festival");
        Element festivalNode;
        Node nameNode;

        int i=0;
        boolean found = false;
        while(i<nodes.getLength() && !found) {
            festivalNode = (Element) nodes.item(i);
            nameNode = festivalNode.getElementsByTagName("name").item(0);
            if(nameNode.getTextContent().equals(festivalName)) {
                root.removeChild(festivalNode);
                found = true;
            }
            i++;
        }
    }

    public void saveXMLFile(String path) throws TransformerException {...}

    @Override
    public String toString() {...}

    private String traverseNodeList(NodeList nodes, String prefix) {...}

    private String getNodeStringData(Node node, String prefix) {...}
}
```
{% endtab %}
{% endtabs %}

Com es pot comprovar, és un codi molt similar a l'anterior. La diferència és que, un cop trobat el festival desitjat, s'esborra del node arrel (variable `root`) utilitzant el mètode `removeChild()`.

### Creació del fitxer XML
Per fer més fàcil i entenedora l'explicació, aquest apartat utilitza l'exemple del fitxer XML que es mostra a continuació, molt més reduït que el fitxer `festivals.xml` però, al mateix temps, complet per poder analitzar tots els elements necessaris.

{% tabs %}
{% tab title="DTD books.dtd" %}
```dtd
<!ELEMENT books (book+)>

<!ELEMENT book (title, author, year)>
<!ATTLIST book isbn13 CDATA #REQUIRED>

<!ELEMENT title (#PCDATA)>
<!ELEMENT author (#PCDATA)>
<!ELEMENT year (#PCDATA)>
```
{% endtab %}
{% tab title="Document XML books.xml" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE books SYSTEM "books.dtd">

<books>
    <book isbn13="978-8445000656">
        <title>The Hobbit</title>
        <author>J.R.R. Tolkien</author>
        <year>1937</year>
    </book>

    <book isbn13="978-8499890944">
        <title>1984</title>
        <author>George Orwell</author>
        <year>1949</year>
    </book>
</books>
```
{% endtab %}
{% endtabs %}

Per poder crear un nou fitxer XML es necessita fer diversos passos:
1. Crear el `Document` que gestionarà l'arbre DOM a memòria RAM
2. Crear els `Element` necessaris mitjançant el mètode `createElement()`, que representen els diversos nodes (etiquetes) del document
3. En cas de ser necessari, definir els atributs d'aquests `Element` mitjançant el mètode `setAttribute()`
4. Anar afegint els `Element` a l'estructura DOM utilitzant el mètode `appendChild()`
5. Finalment, guardar l'estructura DOM en un fitxer físic a memòria permanent

El codi associat a totes aquestes tasques és el següent:

{% tabs %}
{% tab title="Classe Book.java" %}
```java
package cat.inscaparrella.dam2.model;

public class Book {
    private String isbn13;
    private String title;
    private String author;
    private int year;

    public Book(String isbn13, String title, String author, int year) {
        this.isbn13 = isbn13;
        this.title = title;
        this.author = author;
        this.year = year;
    }

    public String getISBN13() { return this.isbn13; }
    public String getTitle() { return this.title; }
    public String getAuthor() { return this.author; }
    public int getYear() { return this.year; }
}

```
{% endtab %}
{% tab title="Classe BooksDOMParser.java" %}
```java
package cat.inscaparrella.dam2.utils;

import cat.inscaparrella.dam2.model.Book;
import org.w3c.dom.*;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.OutputKeys;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerException;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.stream.StreamResult;
import java.util.List;

public class BooksDOMParser {
    private Document xmlDoc;

    public void createXMLFile(List<Book> books) throws ParserConfigurationException {
        DocumentBuilderFactory dfactory = DocumentBuilderFactory.newInstance();
        DocumentBuilder dbuilder = dfactory.newDocumentBuilder();
        this.xmlDoc = dbuilder.newDocument();

        //Create root element
        Element root = this.xmlDoc.createElement("books");
        this.xmlDoc.appendChild(root);

        Element book, title, author, year;
        Text content;
        for(int i=0; i<books.size(); i++) {
            //Create book i
            book = this.xmlDoc.createElement("book");

            //Add attribute ISBN13
            book.setAttribute("isbn13", books.get(i).getISBN13());

            //Create element title with its text content & append to book
            title = this.xmlDoc.createElement("title");
            content = this.xmlDoc.createTextNode(books.get(i).getTitle());
            title.appendChild(content);
            book.appendChild(title);

            //Create element author with its text content & append to book
            author = this.xmlDoc.createElement("author");
            content = this.xmlDoc.createTextNode(books.get(i).getAuthor());
            author.appendChild(content);
            book.appendChild(author);

            //Create element year with its text content & append to book
            year = this.xmlDoc.createElement("year");
            content = this.xmlDoc.createTextNode(Integer.toString(books.get(i).getYear()));
            year.appendChild(content);
            book.appendChild(year);

            //Append book i to root
            root.appendChild(book);
        }
    }

    public void saveXMLFile(String path) throws TransformerException {
        DOMSource source = new DOMSource(this.xmlDoc);
        TransformerFactory tfactory = TransformerFactory.newInstance();
        Transformer transformer = tfactory.newTransformer();

        transformer.setOutputProperty(OutputKeys.INDENT, "yes");
        transformer.setOutputProperty("{http://xml.apache.org/xslt}indent-amount", "4");
        transformer.setOutputProperty(OutputKeys.DOCTYPE_SYSTEM, "books.dtd");

        StreamResult result = new StreamResult(path);
        transformer.transform(source, result);
    }
}
```
{% endtab %}
{% tab title="Programa BooksDOMParserMain.java" %}
```java
package cat.inscaparrella.dam2.view;

import cat.inscaparrella.dam2.model.Book;
import cat.inscaparrella.dam2.utils.BooksDOMParser;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.TransformerException;
import java.util.ArrayList;
import java.util.List;

public class BooksDOMParserMain {
    public static void main() throws ParserConfigurationException, TransformerException {
        List<Book> books = new ArrayList<>();
        BooksDOMParser domParser = new BooksDOMParser();

        books.add(new Book("978-8445000656", "The Hobbit", "J.R.R. Tolkien", 1937));
        books.add(new Book("978-8499890944", "1984", "George Orwell", 1949));

        domParser.createXMLFile(books);
        domParser.saveXMLFile("xml_files/books.xml");
    }
}
```
{% endtab %}
{% endtabs %}