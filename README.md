# Basis für Minova-Dienste

> Dies ist die verpflichtende Basis für alle Applikationen und Dienste.
> Dies wird nicht für Plugins, Extensions und co. verwendet.

## Einführung
Diese POM sollte als parent in jedem Dienst oder Applikation direkt verwendet werden.
Es wird Java 21 verwendet,
da dies die [aktuelle LTS Version](https://en.wikipedia.org/wiki/Java_version_history) ist.

Unter Windows mit Chocolatey kann Java 21 über [choco install openjdk21](https://chocolatey.org/packages/openjdk21) installiert werden.
Das JDK wird standardmäßig unter `C:\Program Files\OpenJDK\openjdk-21*\` installiert.

## Versionen

| Version | Spring Boot | Hinweis |
|---|---|---|
| 11.3.x | 3.5.16 | Spring Boot 3.5, ohne CVE-Overrides. |
| 11.4.0 | 3.5.16 | Spring Boot 3.5 mit CVE-Overrides (siehe unten). Letzte Version der 11er-Linie. |
| 12.0.0 | 4.2.0 | Migration auf Spring Boot 4 (Spring Framework 7, Spring Security 7, Tomcat 11, Jackson 3). Breaking Change. Als [Mini-Anleitung Anpassungen im CAS](https://github.com/minova-afis/aero.minova.cas/issues/1504#issuecomment-7192336277). |

### CVE-Overrides (ab 11.4.0)

Spring Boot 3.5.16 ist der letzte verfügbare Stand der 3.5er-Linie und enthält bekannte Schwachstellen in
transitiven Abhängigkeiten. Deshalb werden folgende Versionen im Profil `spring-boot` überschrieben:

| Abhängigkeit | Version | Property |
|---|---|---|
| Jackson 2 (`com.fasterxml.jackson`) | 2.21.7 | `jackson2.cve.version` |
| Jackson 3 (`tools.jackson`) | 3.1.7 | `jackson3.cve.version` |
| Netty | 4.1.137.Final | `netty.cve.version` |
| Tomcat (`tomcat-embed-core`, `-el`, `-websocket`) | 10.1.59 | `tomcat.cve.version` |
| httpcore5, httpcore5-h2 | 5.4.3 | `httpcore5.cve.version` |
| PostgreSQL JDBC | 42.7.12 | `postgresql.cve.version` |
| org.json | 20231013 | `org-json.cve.version` |

Die Overrides müssen vor dem Import von `spring-boot-dependencies` stehen, da bei importierten BOMs der erste Treffer gewinnt.
Properties wie `tomcat.version` haben bei einem importierten BOM keine Wirkung.
Sie können in einem Projekt über die oben genannten Properties angepasst werden.

Nicht behebbar in der 3.5er-Linie: Die CVEs in `spring-webmvc` 6.2.x (Fix nur in Spring Framework 7).
Diese werden erst mit Version 12.0.0 (Spring Boot 4) gelöst.
Mit 12.0.0 entfallen die Overrides wieder, da Spring Boot 4.2.0 die Versionen selbst liefert.

### Migration auf 12.0.0 (Spring Boot 4)

Spring Boot 4 bringt unter anderem folgende Änderungen mit, die in den Projekten geprüft werden müssen:
* Spring Framework 7 und Spring Security 7
* Tomcat 11
* Jackson 3 (neue Packages `tools.jackson`)

## Beispiele

* [CAS](https://github.com/minova-afis/aero.minova.cas)
* [ADM](https://github.com/minova-afis/aero.minova.afis.dispatch.manager)

## Spring Boot Projekt in Maven und mit Java aufsetzen.

Man erstellt sich eine POM mit folgender Vorlage, welche des Projekt definiert.
In der Vorlage muss noch der Name des Projektes angepasst werden.
Als nächstes übernimmt man noch die Datei `.gitignore` mit deren Hilfe unwichtige Dateien
bei der Versionverwaltung ignoriert werden.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<artifactId>[Name des Projektes]</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<packaging>pom</packaging>
	<parent>
		<groupId>aero.minova</groupId>
		<artifactId>spring.maven.root</artifactId>
		<version>[Aktuelle Version]</version>
	</parent>
	<scm>
		<connection>scm:git:https://github.com/minova-afis/aero.minova.[Name des Projektes].git</connection>
		<tag>HEAD</tag>
	</scm>
	<dependencies>
		<dependency>
			<groupId>aero.minova</groupId>
			<artifactId>service</artifactId>
			<version>12.11.0</version>
		</dependency>
	</dependencies>
</project>
```

## Code Guidelines

### Optionale Attribute

Optionals können ein gutes Mittel zu sein, um die Optionalität von Werten darzustellen (Billion Dollar Mistake).
Allerdings werden diese bei DTOs und DAOs häufig nicht richtig unterstützt und sollten für diese auch nicht verwendet werden.
Bei DAOs gibt es eine Ausnahme, indem man beim Getter des betroffenen Attibutes ein Optional zurückgeben kann.

```java
@Entity
@Data
@NoArgsConstructor
public class MovementChange {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long keyLong;
	@OneToOne
	private Movement original;
	@OneToOne
	private Movement cancellation;
	@OneToOne(optional = true)
	private Movement correction;

	public Optional<Movement> getCorrection() {
		return Optional.ofNullable(correction);
	}
}
```

Wenn man Optionals nicht verwendet, sollte man das betroffene Attribut nach Möglichkeit als optional annotieren:
* In DAOs: @OneToOne(optional = true)
* In DTOs: @org.springframework.lang.Nullable

