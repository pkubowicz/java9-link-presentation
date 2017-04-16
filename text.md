## Czego oczekiwać od modułów w&nbsp;Javie&nbsp;9

<small>Piotr Kubowicz [@pkubowicz](https://twitter.com/pkubowicz)</small>

---

## Plan podróży

1. Czym w Javie będą moduły?
2. Moduły a mikroserwisy
3. Moduły a biblioteki i monolity
4. Wsparcie w IDE (IntelliJ Idea)
5. Narzędzia do budowania

---

## Project Jigsaw

- [długa historia](http://www.java9countdown.xyz/) (grudzień 2014)
- cele
  - niezawodność: jasne zależności
  - mocna enkapsulacja: ukrywanie publicznych typów
- zastosowanie
  - wewnętrzna struktura runtime'u Javy
  - kod użytkownika

---

## Czym jest moduł

```
module example.greeter.protocol {
    requires java.base;
    requires guava;
    exports example.greeter.protocol;
}
```

Note:
nie ma opcji, że zabraknie klasy - weryfikacja na starcie

---

## Organizacja JRE

<ul>
<li>java.base (najpewniej jest tu każda klasa, o której pomyślisz)</li>
<li class="fragment" data-fragment-index="0">java.logging</li>
<li class="fragment" class="fragment" data-fragment-index="0">java.management (JMX)</li>
<li class="fragment" data-fragment-index="0">java.instrument</li>
<li class="fragment" data-fragment-index="1">java.corba :)</li>
<li class="fragment" data-fragment-index="2">[razem 24](http://download.java.net/java/jdk9/docs/api/index.html?overview-summary.html)</li>
</ul>


Note:
corba - to, co wszyscy kochamy i potrzebujemy

---

## Nie trzeba używać modułów

Kod można uruchamiać "po staremu" - przez classpath.

Działa np. Tomcat 8, ElasticSearch 2.4

---

## Jak deklarować moduł

- moduł to JAR z plikiem `module-info.class`
- tworzymy `src/main/java/module-info.java`
- takiego pliku nie da się skompilować przy<br>`-source 1.8`
- po skompilowaniu takiego JAR-a nie wczyta Java 8

Note:
pokazać po raz pierwszy projekt w IDE

---

## Moduły w mikroserwisach

[jlink](http://docs.oracle.com/javase/9/tools/jlink.htm) - tworzy z modułów *custom runtime image*

<pre>build/custom-jre:
bin  conf  include  legal  lib  release

build/custom-jre/bin:
java  keytool

build/custom-jre/lib:
classlist jrt-fs.jar libjimage.so libnio.so    modules  tzdb.dat
jexec     jvm.cfg    libjsig.so   libverify.so security
jli       libjava.so libnet.so    libzip.so    server
</pre>

`custom-jre/bin/java -m moduł/klasa`

---

## Moduły w mikroserwisach

- np. Docker z prostym serwerem HTTP z OpenJDK 8 na Debianie Jessie: **310 MB**
- to samo zlinkowane z Javą 9: **199 MB**
- OpenJDK 8 na Alpine Linuksie: **81 MB**
- Alpine + Java 9: **35-41 MB**
- bez Dockera: katalog **30 MB**, .tar.bz2 **15 MB**

---

## Moduły w mikroserwisach

Weźmy prawdziwy serwer, ale lekki np. Undertow ([czołówka wydajności](https://www.techempower.com/benchmarks/#section=data-r13&hw=ph&test=plaintext))

java.base, java.naming, java.security.jgss, java.sql, java.logging, java.management, java.security.sasl

Custom JRE: 37 MB

---

## Jak wyciągnąć zależności?

[jdeps](docs.oracle.com/javase/9/tools/jdeps.htm) (ale nie z JDK 8)

<pre>% jdeps -summary --class-path \
xnio-api-3.3.6.Final.jar:jboss-logging-3.2.1.Final.jar \
undertow-core-2.0.0.Alpha1.jar
</pre>
<pre>
undertow-core-2.0.0.Alpha1.jar -> java.sql
undertow-core-2.0.0.Alpha1.jar -> jdk.unsupported
</pre>
<pre>% jdeps -verbose:class undertow-core-2.0.0.Alpha1.jar</pre>
<pre>
io.undertow.util.FastConcurrentDirectDeque -> sun.misc.Unsafe
JDK internal API (jdk.unsupported)
</pre>

---

## Moduły dla monolitów

<pre>exports example.greeter.protocol;</pre>

- pisząc moduł udostępniamy klientom tylko te publiczne klasy, które wybierzemy
- resztę możemy zmieniać jak chcemy i mamy gwarancję, że nic nie popsujemy klientom

---

## Moduły dla monolitów

<pre>requires guava;</pre>

- klienci nie widzą naszych zależności
- możemy je dodawać i usuwać jak chcemy

<pre>requires transitive guava;</pre>

- zależność to część naszego API - udostępniamy ją klientom

---

## Wsparcie IDE

- Idea od [2017.1](https://blog.jetbrains.com/idea/2017/03/support-for-java-9-modules-in-intellij-idea-2017-1/)
  - sugeruje modyfikacje w `module-info.java`
  - ale też ciągle podpowiada nieeksportowane klasy
- Eclipse - [podobno](https://waynebeaton.wordpress.com/2016/09/14/java-9-module-info-files-in-the-eclipse-ide/)

Note:
pokazać w Idei zabronione importy na external-libs

---

## Uruchamianie z modułami

Zamiast
<pre>
java -cp build/modules/greeter-protocol.jar:
build/modules/greeter-server.jar example.greeter.server.Runner
</pre>
uruchamiamy
<pre>
java --module-path build/modules -m example.greeter.server/
example.greeter.server.Runner
</pre>

---

## Weryfikacja przy uruchamianiu

<pre>
Error occurred during initialization of boot layer
java.lang.module.FindException: Module example.greeter.protocol
not found, required by example.greeter.server
</pre>

---

## Zależności z Javy 8 jako moduły

Automatyczne moduły - JAR wrzucony do *module path* staje się modułem.

Nazwa modułu z nazwy pliku: `guava-21.0.jar -> guava`

---

## Problemy z modułami

- zakaz *split packages* - dany pakiet tylko w 1 module, nawet nieeksportowany
  - wiele bibliotek nie było pisanych z taką myślą
- Jigsaw nie zajmuje się konfliktem wersji, to zadanie Gradle/Mavena
  - czyli nawet z modułami możemy dostać ClassNotFoundException w runtimie

---

## Kiedy biblioteki będą modułami?

- nieprędko
- Spring 5.0 M5 - kompatybilny z Javą 9, na razie bez module-info

---

## Budowanie

- Gradle - jeszcze [nie działa](https://discuss.gradle.org/t/java-9-build-9-ea-155-mixed-mode-problems/21618) z Javą 9
- Maven - chyba [da się](https://github.com/cfdobber/maven-java9-jigsaw)

---

## Przygotowanie na moduły w Javie 8

- Java Library Plugin w Gradle'u
  - nowy, promowany sposób kompilowania Javy
  - [ukrywanie zależności przed klientami](https://blog.gradle.org/incremental-compiler-avoidance#introducing-the-java-library-plugin)
- Java Software Model
  - eksperymentalny, brak wsparcia w IDE
  - dodatkowo [ukrywanie publicznych klas przed klientami](https://docs.gradle.org/3.5/userguide/java_software.html#sec:specifying_api_classes)

---

## Do samodzielnego czytania

- http://docs.oracle.com/javase/9/migrate/toc.htm
- http://openjdk.java.net/projects/jigsaw/quick-start
- [slajdy The Java 9 Module System In Action](https://codefx-org.github.io/talk-jigsaw-walkthrough/2016-05-20-JEEConf)
- [Advanced Modular Development na Java One 2016](https://www.youtube.com/watch?v=WWbw8u5jaaU)
- [The State of the Module System](http://openjdk.java.net/projects/jigsaw/spec/sotms/) - zaawansowane tematy; nieaktualna składnia!
- [aktualny format deklaracji modułu](http://cr.openjdk.java.net/~mr/jigsaw/spec/lang-vm.html)
- [RedHat strzela focha](https://developer.jboss.org/blogs/scott.stark/2017/04/14/critical-deficiencies-in-jigsawjsr-376-java-platform-module-system-ec-member-concerns)

---

## Kod i slajdy

- [github.com/pkubowicz/java9-link](https://github.com/pkubowicz/java9-link)
- [www.slideshare.net/PiotrKubowicz1](https://www.slideshare.net/PiotrKubowicz1)

<img src="img/Cc-white.svg" height="100" alt="Creative Commons" class="transparent">
<img src="img/Cc-by_new_white.svg" height="100" alt="Attribution" class="transparent">

Prezentacja na licencji <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International</a>
