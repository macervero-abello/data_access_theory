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
{% tab title="Document XML" %}
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