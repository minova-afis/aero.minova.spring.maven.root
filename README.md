# Basis für Minova-Dienste

> Dies ist die verpflichtende Basis für alle Applikationen und Dienste.
> Dies wird nicht für Plugins, Extensions und co. verwendet.

## Einführung
Diese POM sollte als parent in jedem Dienst oder Applikation direkt verwendet werden.
Es wird Java 17 verwendet,
da dies die [aktuelle LTS Version](https://en.wikipedia.org/wiki/Java_version_history) ist.

Unter Windows mit Chocolatey kann Java 17 über [choco install openjdk17](https://chocolatey.org/packages/openjdk17) installiert werden.
Das JDK wird standardmäßig unter `C:\Program Files\OpenJDK\openjdk-17*\` installiert.

## Beispiele

* [CAS](https://github.com/minova-afis/aero.minova.cas) TODO von Reiner
* [Fuelportal](https://github.com/minova-afis/aero.minova.afis.fuelportal) TODO von Reiner
* [ADM](https://github.com/minova-afis/aero.minova.afis.dispatch.manager) TODO von Martins
* [AirportCollector](https://github.com/minova-afis/ch.minova.service.airportcollector) TODO von Kerstin

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
</project>
```

## Angabe von Abhängigkeiten

TODO

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

