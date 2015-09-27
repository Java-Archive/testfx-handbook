# TestFX – TDD für JavaFX
TDD in der GUI Entwicklung ist für viele Entwickler ein unbeliebtes Thema. 
Aus Sicht eines Entwicklers sind die passenden Werkzeuge meist weniger 
praktisch. Es werden Tools benötigt, mit denen schnell eine partielle 
Komponente getestet werden kann. Genau das verspricht TestFX.

## Was ist TestFX?
TestFX [www.testfx.org](www.testfx.org) erhebt den Anspruch dem 
Entwickler schnell und unkompliziert zu helfen JUnit-Tests 
für JavaFX-Elemente zu schreiben. Wir werden uns das nun im Detail ansehen. 
Zu Beginn steht immer die Projektinitialisierung. 
In unserem Fall ist dieses recht einfach, da es sich lediglich 
um eine minimalistische pom.xml mit zwei Abhängigkeiten handelt. 
Bei Verwendung des JDK 8 ist die pom.xml ein wenig kleiner, da 
JavaFX keine besondere Definition benötigt, wie es beim JDK7 notwendig ist. 
TestFX selber ist in der aktuellen Version für JavaFX8 vorgesehen. 
Wenn man TestFX zusammen mit Java7 verwenden möchte, kann man am besten 
den LTS Branch () aus dem git-Repository (auf github) verwenden. 
Die notwendigen Abhängigkeiten um bei einem Projekt die Funktionalität 
von TestFX hinzuzufügen sind im nachfolgenden Listing zu sehen. 
Sicherlich sind die Versionsnummern anzupassen. 
Nun steht der Verwendung von TestFX nichts mehr im Wege und es 
kann mit dem Schreiben von JUnit-Tests begonnen werden.
```
<dependency>
  <groupId>org.testfx</groupId>
  <artifactId>testfx-core</artifactId>
  <version>4.0.0 </version>
</dependency>
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-all</artifactId>
    <version>1.3</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## Hello TestFX-World
Wie seit Jahrzehnten üblich beginnen wir mit dem Klassiker, einem HelloWorld. Der Grundaufbau eines JUnit-Test auf Basis von TestFX besteht aus einer Klasse abgeleitet von der abstrakten Klasse org.loadui.testfx.GUITest. Die zu implementierende Methode getRootNode() liefert das zu testende GUI Element als Instanz der Klasse javafx.scene.Parent zurück. Die programmatisch erzeugte Instanz ist diejenige, auf der die nachfolgenden Tests ausgeführt werden. In unserem Beispiel ist es ein Button.
@Category(TestFX.class)
public class SimpleButtonTest extends GuiTest {
  @Override
  protected Parent getRootNode() {
    Button btn = new Button();
    btn.setId("btn");
    btn.setText("Hello World");
    btn.setOnAction((actionEvent)
        -> btn.setText( "was clicked" ));
    return btn;
  }
}
Listing 1.1: Listingunterschrift
Hier kann man schön erkennen, dass die GUI-Elemente in keine Hülle wie z.B. eine VBox eingebettet werden müssen. Es kann tatsächlich auf einer beliebigen GUI-Komponente gearbeitet werden. Mit diesem Schritt ist die Basis geschaffen um die funktionalen Testmethoden zu schreiben.
1.1.2.1	Der erste Test
Wie bei JUnit üblich werden die Testmethoden mit @Test annotiert um sie bei der Ausführung zu berücksichtigen. Nun stellt sich die Frage, wie auf die Instanz die durch die Methode getRootNode() erzeugt wird, zugegriffen werden kann. In unserem Fall die Instanz der Klasse Button. Hier liefert TestFX einige Servicemethoden. Die hier notwendige ist find(String query) aus der Klasse GUITest selbst. Der Name des Attributes aus der Methodensignatur lässt vermuten, dass hier mehr als nur eine ID übergeben werden kann. In diesem Fall ist es die Möglichkeit einen CSS-Selector zu verwenden, was wir in unserem Beispiel nicht verwenden, wir nehmen einfach direkt die ID der Instanz: die FXID #btn.
@Test
public void shouldClickButton(){
  Button button = find( "#btn" );
  click(button);
  verifyThat( "#btn", hasText("was clicked") );
}
Listing 1.1: Listingunterschrift
Nachdem die Instanz erhalten wurde, kann mit den funktionalen Tests begonnen werden. Auch hier stellt TestFX dem Entwickler eine Menge Servicemethoden bereit. Allgemein gesagt, wird immer der Ansatz verfolgt alles mit dem Builder-Pattern abzubilden. Damit lassen sich die Ausdrücke die Navigation und Verwendung der GUI-Elemente beschreiben sehr kompakt formulieren. Und damit sind wir bei dem zweiten Grundgedanken von TestFX. Die Formulierung der Testfälle soll möglichst dem Verhalten bei der Verwendung nachempfunden werden. In unserem Beispiel das Klicken auf den Button, das durch die Methode click(button) realisiert wird. Ist die gewünschte Aktion erfolgt, wird überprüft ob das Ergebnis der Reaktion mit dem der Erwartung übereinstimmt. Hier soll durch das Drücken auf den Button der Text von „Hello World“ auf „was clicked“ verändert werden. Ob das stattgefunden hat, kann mittels der Methode verifyThat(fxid, matcher) geprüft werden. An dieser Stelle wird der Entwickler mit dem Framework Hamcrest konfrontiert, da die logischen Überprüfungen mittels Matcher realisiert worden sind. Beispiele befinden sich in dem Package org.loadui.testfx.matchers. 


1.1.3	TestFX - Internals
Sehen wir uns nun die Funktionsweise und den Aufbau von TestFX an. Als Startpunkt nehmen wir die Klasse GuiTest, da sie der Einstiegspunkt für den Entwickler sind. Die Klasse GuiTest ist abstract, erfordert lediglich die Implementierung der Methode getRootNode() und bietet eine Menge Servicefunktionen. Die Methoden bilden maßgeblich das Grundgerüst zur Definition der Testfälle. Methoden sind 
•	find: Auffinden bestimmter GUI Elemente
•	click: Simulieren eines Maus-Klicks auf einem GUI Element
•	rightClick: Simulieren eines Maus-Klicks mit der rechten Maustaste
•	doubleClick: Simulieren eines Doppelklicks 
•	drag: Drag und Drop simulieren
•	move: Mausbewegung auf eine Position bzw. ein GUI Element
•	press/release: Mausklick/Tastenklick mit zeitlicher Dauer
•	scroll: Rollen des Mausrades
•	push: Drücken von Tasten
•	und vieles mehr…
Die Definitionen sind in dem Interface FxRobot zu finden. Damit stehen einem als Entwickler fast alle Interaktionsmöglichkeiten offen, um Benutzerverhalten zu simulieren. Innerhalb der Klasse ToolkitAppSetupFactory befindet sich die Klasse ToolkitApplication extends Application und definiert den Einstiegspunkt um eine JavaFX Applikation zu starten. Letzten Endes muss jeder jUnit Test in einer eigenen oder alles Tests nacheinander in derselben Applikation laufen. Und hier beginnen die meisten Probleme. Denn schreibt man als Entwickler auf herkömmlichen Wege die JavaFX jUnit Tests, so landet man schnell bei der Tatsache, dass immer nur eine Instanz der Klasse Application zur selben Zeit innerhalb einer JVM aktiv sein darf. 

Nun Stellt sich die Frage, wie n verschiedene jUnit-Tests gestartet werden. Der Grundgedanke ist recht einfach. Beim Start der Tests (getriggert durch @Before) in der Methode setupGuiTest() wird AppRobotTestBase.setupApplication() aufgerufen. Hier beginnt der Aufbau der JavaFX-Applikation in die jeweils die GUI Komponente von der Methode getRootNode() eingebettet wird.
Jeder Test wird demnach in einem eigenen Thread verpackt. Da jede jUnit-Test-Klasse von GuiTest abgeleitet worden ist, besteht immer eine 1 zu 1 Beziehung zwischen einer Instanz der jUnit-Testklasse und der JavaFX Application. Die Tests innerhalb einer Klasse laufen damit immer in einer einzelnen JavaFX – Instanz. Wichtig zu beachten ist, das die Tests demnach nicht beliebig parallel ausgeführt werden können derzeitig.
1.1.4	Wie setze ich TestFX ein?
Bei der Verwendung von TestFX stellt sich sofort die Frage, wie man am besten die Tests aufteilen und aufbauen wird. Zu beachten ist, dass das zu testende GUI Element nur einmal pro Instanz der TestKlasse erzeugt wird. Werden also mehr als ein Test in eine TestKlasse geschrieben, so muss sichergestellt sein, dass eine beliebige Reihenfolge der Tests zu dem gewünschten Ergebnis führen wird. Die implizite Annahme, dass die Tests in der Reihenfolge ausgeführt werden, in der sie definiert worden sind, ist hier eine sehr typische Fehlerquelle. Aus diesem Grunde sollte immer nur ein Test in einer TestKlasse definiert werden. 
