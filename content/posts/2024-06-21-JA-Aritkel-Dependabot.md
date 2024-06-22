---
title: "Automatische Dependency Updates mit GitHub Actions und Dependabot"
description: "In den meisten Firmen ist es immer das gleiche, 1-2 Personen kümmern sich um die Dependencies, während sich viele Kolleg*innen davor drücken und hoffen, dass irgendwer das aktuell hält. Also lieber gleich den ganzen Prozess mit GitHub Actions und Dependabot automatisieren."
date: 2024-06-21
---


In den meisten Firmen ist es immer das gleiche, 1-2 Personen kümmern sich um die Dependencies, während sich viele Kolleg*innen davor drücken und hoffen, dass irgendwer das aktuell hält. Also lieber gleich den ganzen Prozess mit GitHub Actions und Dependabot automatisieren.

== Das Leid mit veralteten Dependencies
Wer nachhaltige Software schreiben will, sollte diese immer aktuell halten. Wir erinnern uns noch alle an die unzähligen Projekte, die wir im Dezember 2021 / Januar 2022 aktualisieren mussten, als die log4j Sicherheitslücke (CVE-2021-44228) bekannt wurde. In vielen Projekten wurde log4j in einer verwundbaren Version direkt oder über Dependencies eingesetzt, diese galt es damals zu aktualisieren. Und plötzlich stellte man fest: Mist, die Dependency die man nutzt, hat eine log4j Abhängigkeit, die Dependency lässt sich aber wegen _breaking changes_ nicht upgraden. Deswegen mussten Workarounds eingesetzt werden, um sicherzugehen, dass nur die gepatchte Version von log4j verwendet wurde. Deshalb versuchen gute Software Entwickler*innen die Abhängigkeiten immer auf dem neuesten Stand zu halten. Mein Kollege Dennis Hörsch und ich haben uns deshalb eingehend mit Dependency-Management und entsprechenden Prozessen beschäftigt.

In dieser Schritt-für-Schritt-Anleitung werde ich euch einen Einblick geben, was die Möglichkeiten sind, was es für Probleme gab, und was es noch immer für Probleme gibt.

Für die einzelnen Schritte habe ich im Beispielprojekt <<BeispielProject>> jeweils einen Branch angelegt, um alle einzelnen Schritte nachvollziehbar zu machen. In der Anleitung schreibe ich immer dazu, in welchem Branch der beschriebene Stand ist. Der `main` Branch ist das fertige Endprodukt, dieser ist auch für Dependabot aktiviert.

Wir starten mit dem Spring Initializer <<SpringInitializer>>, hier wähle ich ein Gradle-Kotlin Projekt mit JDK 21, den Stable Release von Spring Boot und als Sprache Kotlin. In meinem Beispiel Repo ist dies der Branch *_step 0_*.
Wenn das Projekt aus dem Initializer bekommt, sind die Dependencies in der `build.gradle.kts` definiert. Gerade in größeren Projekten sieht man immer wieder, dass die Dependencies in der `settings.gradle.kts` mittels Versions Catalogs definiert werden. Dies hat den Vorteil, dass es  übersichtlich für alle Module definiert werden kann. Um möglichst nahe an der Realität zu bleiben, habe ich dies auch im Beispielprojekt so abgeändert.

== Pull Requests mit GitHub Actions automatisch testen
Mit GitHub Actions gibt es jetzt, ähnlich wie bei GitLab Pipelines, die Möglichkeit seine eigenen automatischen Prozesse abzubilden. So kann man komplette CI/CD Pipelines oder automatische Tests bei einem Pull-Request (PR) darüber abbilden. Deswegen legen wir jetzt im Basisverzeichnis des Repos einen Ordner namens `.github`, in dem sich ein weiterer Ordner namens `workflows` befindet.

In den `workflows` Ordner kommt jetzt eine Workflow-Datei mit folgendem Inhalt:  xref:#listing.testing-demo1-original[].
Bei jedem neuen Pull-Request auf den `main` Branch, in dem Änderungen im Pfad `demo1` wird der Test ausgeführt. Hier wird ein Ubuntu container (`runs-on: ubuntu-latest`) gestartet, in dem wird erst der Code augecheckt (`uses: actions/checkout`), dann wird Java (`uses: actions/setup-java`) mit der JDK 21 (`java-version: '21'`) verwendet. Bei der Java Distribution habt ihr die Wahl zwischen den gängigen Distributionen.

[[listing.testing-demo1-original]]
[source,text]
.testing-demo1.yml um den code zu testen
----
name: Test Demo 1

on:
pull_request:
paths:
- 'demo1/**'
branches: [ main ]

workflow_dispatch: {}

jobs:
test-demo-1:
runs-on: ubuntu-latest
defaults:
run:
working-directory: ./demo1
steps:
- uses: actions/checkout@v4
- name: Set up JDK 21
uses: actions/setup-java@v4
with:
distribution: 'liberica'
java-version: '21'
- run: ./gradlew clean test
----

Dies alles findet ihr im Branch *_step_1_*

Zum Testen fügen wir dem Quellcode einen Kommentar hinzu und pushen dies auf einen extra Branch und erstellen einen PR. Daraufhin kann man sehen, dass der Test läuft. Allerdings zeigt euch GitHub schon an, dass ihr den PR mergen könnt, ohne dass der Test durchgelaufen ist. xref:#bild.one[] Dies wollen wir natürlich nicht, weswegen wir uns jetzt um die Branch-Protection kümmern müssen.

.Obwohl die Tests laufen, könntet ihr mergen, das ist schlecht. ([Screenshot GitHub])
[id="bild.one"]
image::screenshot_able_to_merge_with_running_test.jpg[]

Klickt in dem Projekt auf _Settings_ -> _Branches_ -> _Add branch protection rule_. Wir wollen den `main` Branch schützen, also geben wir bei _Branch name pattern_ `main` an.
Dann wählen wir _Require status checks to pass before merging_ aus. Das blockiert das Mergen, falls die Tests noch nicht gelaufen oder fehlgeschlagen sind. Ein User mit Adminrechten, kann dies aber auch noch weiterhin, es gibt aber auch die Möglichkeit den Admin dazu zu zwingen die Regel einzuhalten (_Do not allow bypassing the above settings_ in den Branch Protection Rules). _Require branches to be up to date before merging_ wählen wir ebenfalls aus, um zu garantieren, dass immer der neueste Stand von `main` in einem PR verwendet wird. Dies garantiert uns, dass die Tests auch mit dem aktuellen Stand von `main` funktionieren. Normalerweise sollte dies ja ein Dev immer selber machen, aber so ist man auf der sicheren Seite.  xref:#bild.two[]

<img src="../images/screenshot_branch_protection.jpg" alt="So stellt ihr die Branch Protections ein ([Screenshot GitHub]"/>

Solltet ihr in euren Branch-Protection Rules eure Tests nicht finden, liegt das daran, dass diese zuvor mindestens einmal laufen müssen.
Hierfür geht ihr auf GitHub auf den Actions Buttons, wählt euren Workflow aus und klickt auf _Run workflow_, wie ihr es in xref:#bild.ten[] seht

<img src="../images/screenshot_triggering_workflow.jpg" alt="So triggert ihr einen Workflow ([Screenshot GitHub])"/>

Jetzt könnt ihr als Nicht-Admin nicht mehr einen Branch mergen, bevor der Test grün ist. xref:#bild.three[]

Damit sind wir jetzt bestens vorbereitet um Dependabot einzubauen

<img src="../images/screenshot_branch_merge_protected.jpg" alt="Nur noch Admins können jetzt noch mergen, solange der Test läuft. ([Screenshot GitHub])"/>

== Dependabot kümmert sich um eure Dependencies
Alle Änderungen hierzu befinden sich im Branch *_step_2_*

Dependabot war früher ein eigenes Unternehmen, bevor es im Mai 2019 von GitHub gekauft wurde. Seitdem wird es immer mehr in GitHub fest integriert, was wir uns hier zu Nutze machen. Dependabot analysiert Dependencies, prüft auf CVEs und veraltete Versionen unabhängig von der Sprache. Ihr könnt damit sowohl eure JVM Dependencies tracken, aber auch Abhängigkeiten in npm, GitHub Actions, Docker, pip, go, Terraform und noch einige weitere.<<DependabotEcoystem>>.
Hierfür fügt ihr im Ordner `.github` eine Datei namens `dependabot.yml` hinzu. Diese schaut aus wie in xref:#listing.dependabot_start[]. Wir nutzen in diesem Beispiel nur das `gradle` Ecosystem. Wir wollen das ganze täglich um 9 Uhr laufen lassen. Derzeit ist es nicht möglich das Scheduling auf Mo-Fr zu beschränken. Ihr könnt es aber alternativ auch nur einmal wöchentlich, während eines Arbeitstages, laufen lassen.

[[listing.dependabot_start]]
[source,text]
.dependabot.yml
----
version: 2
updates:
- package-ecosystem: "gradle"
  directory: "/demo1/"
  schedule:
  interval: "daily"
  time: "09:00"
  open-pull-requests-limit: 100
  labels:
    - "dependencies"
    - "gradle"
    - "kotlin"
    - "automatic-merge"
----
Die Labels _dependencies_, _gradle_, _kotlin_ und _automatic-merge_ gibt es noch nicht und müssen erst noch erstellt werden. Diese werden an zu einem von Dependabot erstellten PR hinzugefügt. Ebenso benötigen wir die Labels _patch_, _minor_ und _major_, welche die verschiedenen Versionsänderungen darstellen sollen. Labels erstellt ihr am einfachsten mit der GitHub CLI <<GithubCli>> xref:#listing.github_cli_label[] Die Labels _dependencies_, _gradle_, _kotlin_ sind nur für euch in einem Projekt um es bei vielen PRs übersichtlich zu halten, benötigt wird es für das Beispiel nicht.

[[listing.github_cli_label]]
[source,text]
.Erstellen eines Labels mit der GitHub CLI
----
$ gh label create LABELNAME
----

Um Dependabot zu testen downgraded ihr in der `build.gradle.kts` eine Version und pusht das ganze in den `main` Branch. Auf eurer GitHub Projektseite auf _Insights_ -> _Dependency Graph_ -> _Dependabot_, wählt ihr dann eure `build.gradle.kts` aus, und startet Dependabot. Nach einiger Zeit, werdet ihr dann einen PR von Dependabot sehen. In meinem Beispiel wurden gleich mehrere Sachen zum Updaten gefunden. xref:#bild.four[]


<img src="../images/screenshot_dependabot_prs.jpg" alt="Von Dependabot erstellte PRs ([Screenshot GitHub])"/>

Nun ändern wir in der extra angelegten `settings.gradle.kts` eine Version, z.b. die `version("lombok", "1.18.30")` zur Vorgänger Version `1.18.28`. Dann pushen wir das zu `main` und starten wie oben beschrieben Dependabot. Wie ihr sowohl in den logs, diese sind dort zu finden, wo ihr Dependabot startet, als auch auf GitHub unter Pull Requests sehen könnt, ist nichts passiert. Dependabot ignoriert leider die Änderungen in der `settings.gradle.kts`.

== Version Catalogs mit Hilfe eines libs.versons.toml File
Alle Änderungen findet ihr hierzu im *_step_3_* Branch.

Da die `settings.gradle.kts` nicht von Dependabot ausgwertet wird, müssen wir anders vorgehen. Seit neuestem unterstützt Dependabot auch Versionskataloge, allerdings muss es hierfür als  `.toml` File abgelegt werden. Dies ist laut Gradle-Dokumentation entweder in der höchsten Ebene des Projekt möglich (in diesem Fall also in `demo1`) oder aber im `gradle` Ordner, welches ich im Beispiel verwende. Wir löschen jetzt also komplett den `dependencyResolutionManagement` Teil in der `settings.gradle.kts` und erstellen wie folgt die `libs.versions.toml` Datei: xref:#listing.libs_versions_toml[]:


[[listing.libs_versions_toml]]
[source,text]
.Das neu erzeugte libs.versions.toml File, inkl der Plugins
----
[versions]
jacksonKotlin = "2.16.1"
junit-jupiter = "5.10.1"
hamkrest = "1.8.0.1"
kotlin = "1.9.22"
kotlinReflect = "1.9.22"
lombok = "1.18.28"
springboot = "3.2.1"
springDependencyManagementVersion = "1.1.1"

[libraries]
springBootStarterWeb = { module = "org.springframework.boot:spring-boot-starter-web", version.ref = "springboot" }
springBootStarterTest = { module = "org.springframework.boot:spring-boot-starter-test", version.ref = "springboot" }
jacksonKotlinModule = { module = "com.fasterxml.jackson.module:jackson-module-kotlin", version.ref = "jacksonKotlin" }
kotlinReflect = { module = "org.jetbrains.kotlin:kotlin-reflect", version.ref = "kotlinReflect" }
lombok = { module = "org.projectlombok:lombok", version.ref = "lombok" }

[plugins]
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
kotlin-spring = {id ="org.jetbrains.kotlin.plugin.spring", version.ref = "kotlin"}
springBoot = { id = "org.springframework.boot", version.ref = "springboot" }
springDependencyManagement = { id = "io.spring.dependency-management", version.ref = "springDependencyManagementVersion" }
----

Die bis dato noch vorhandenen plugins, schreiben wir auch in das `.toml` File, die `build.gradle.kts` verweist dann, ähnlich wie die Dependencies nur noch auf die `libs.plugins.*`. xref:#listing.plugins_in_build_gradle[]


[[listing.plugins_in_build_gradle]]
[source,text]
.So sieht jetzt die Plugin Sektion in der build.gradle.kts aus
----
plugins {
alias(libs.plugins.kotlin.jvm)
alias(libs.plugins.kotlin.spring)
alias(libs.plugins.springDependencyManagement)
alias(libs.plugins.springBoot)
}
----
Wenn wir jetzt, wie vorher beschrieben, den Dependabotprozess manuell anstoßen, wird ein PR für die veraltete `lombok` Dependency erstellt.

== Die PRs automatisch mergen
Alle Änderungen findet ihr hierzu im *_step_4_* Branch.

Die PRs werden jetzt alle automatisch erstellt, wie wir uns das vorgestellt haben. Jetzt fehlt nur noch der entscheidende Schritt: Das automatische Mergen der Dependency Pull-Requests. Um dies vorzubereiten, müssen wir noch 2 Sachen in den General-Settings ändern: xref:#bild.five[]

<img src="../images/screenshot_settings_autotmerge_delete.jpg" alt="Allow auto-merge und Automatically delete head branches sollte an sein([Screenshot GitHub])"/>

Wir erstellen eine neue Datei `enable-auto-merge.yml`, diese wird ebenfall in `workflows` abgespeichert, siehe xref:#listing.enable_automerge[]

[[listing.enable_automerge]]
[source,text]
.Der enable-auto-merge Workflow
----
name: Enables auto-merge on dependency pull requests
"on":
pull_request:
branches:
- main
paths:
- 'demo1/**'
workflow_dispatch: {}

jobs:
enable-automerge:
if: |
contains(github.event.pull_request.labels.*.name, 'automatic-merge')
&& (
contains(github.event.pull_request.labels.*.name, 'minor')
|| contains(github.event.pull_request.labels.*.name, 'patch')
)
runs-on: ubuntu-latest
permissions:
contents: write
pull-requests: write
steps:
- name: Enable auto-merge
run: |
gh pr merge --merge --auto "$PR_URL"
env:
PR_URL: ${{github.event.pull_request.html_url}}
GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
----
Um den Workflow in den Branch-Protection-Rules auswählen zu können müssen wir ihn jetzt noch manuell auslösen.

Findet Dependabot mehr als nur einen PR, würde beim 2. PR, nachdem der 1. gemerged wurde folgendes stehen: _This branch is out-of-date with the base branch_. Aus diesem Grund macht Dependabot ein force-push von `main` in den jeweiligen PR. xref:#bild.six[] Danach laufen dann die Tests von vorne los, und der Code wird bei Erfolg automatisch gemerged.

<img src="../images/screenshot_automatic_pr.jpg" alt="So sieht ein automatischer Ablauf aus ([Screenshot GitHub])"/>

Manchmal braucht ein Force-Push etwas Zeit, man kann dann das ganze auch per Button selber erledigen.

== Besonderheiten in einem Monorepo
Und jetzt noch ein Monorepo Beispiel, das fertige Monorepoprojekt findet ihr hier.<<BeispielProjectMonorepo>>

Wenn Teams nicht ausschließlich mit _pairing_ und _trunk-based_, also nur auf dem `main` branch arbeiten, sondern klassisch mit Branches und Pull-Requests, sollen nur die eigenen Tests laufen und nicht die Tests der anderen Projekte.

Um dies zu erreichen, habe ich alle Workflows dupliziert, einmal einen normalen Test, `testing-demo1.yml` und dann den gleiche noch mal mit `auto-merge-testing-demo1.yml`. Der Unterschied zwischen den Workflows ist nur, wann dieser laufen soll, und wann es übersprungen (_skipped_) werden soll. `testing-demo1.yml` soll nur bei Änderungen in Demo1 getriggert werden xref:#listing.normal_test_demo_1[]. `auto-merge-testing-demo1.yml` soll aber bei allen Projektänderungen getriggert werden.xref:#listing.auto-merge-test_demo_1[], Um das skipping zu erreichen, überprüfen wir in beiden Workflows noch, ob das Label `automatic-merge` gesetzt ist, oder ob es nicht gesetzt ist sowie ob es das richtige Projekt ist.

[[listing.normal_test_demo_1]]
[source,text]
.So schaut der Beginn von testing-demo1.yml aus

----
name: Test Demo 1

on:
pull_request:
paths:
- 'demo1/**'
- '.github/**'
  branches: [ main ]

  workflow_dispatch: {}

jobs:
test-demo-1:
if: |
!contains(github.event.pull_request.labels.*.name, 'automatic-merge')
runs-on: ubuntu-latest
----



[[listing.auto-merge-test_demo_1]]
[source,text]
.So schaut der Beginn von auto-merge-testing-demo1.yml aus

----
name: Auto-Merge Test Demo 1

on:
pull_request:
paths:
- 'demo1/**'
- 'demo2/**'
- 'demo3/**'
- '.github/**'
  branches: [ main ]
  workflow_dispatch: {}

jobs:
auto-merge-test-demo-1:
if: |
contains(github.event.pull_request.labels.*.name, 'demo1') ||
contains(github.event.pull_request.labels.*.name, 'automatic-merge')
runs-on: ubuntu-latest

----

Wenn wir jetzt testweise in jedem Projekt eine Dependency downgraden, sehen wir schön in der xref:#bild.eight[], dass alle 3 Dependencies in den Projekten geupdated werden.

<img src="../images/screenshot_all_3_projects_run.jpg" alt="In allen 3 Pojekten wurde ein PR getriggert ([Screenshot GitHub])"/>

Erstellt man selber einen PR, mit einer kleinen Änderung in einem Projekt, so sieht man zwar, dass alle _auto-merge_ Tests gestartet werden, aber da dann die Bestimmungen nicht erfüllt sind, diese geskipped werden. xref:#bild.nine[],

.Nur die benötigten Tests laufen, der Rest wird geskipped ([Screenshot GitHub])
[id="bild.nine"]
image::screenshot_only_code_test_running.jpg[]

== Und was ist mit Maven?
In einem Maven Projekt funktioniert das ganze exakt genauso. Der einzige Unterschied ist, dass ihr in eurem Testworkflow statt `./gradlew clean build` nur `mvn clean test` verwendet. Ein fertiges Beispielprojekt könnt ihr euch hier anschauen <<BeispielProjektMaven>>

== Probleme & Tricks
Ab und an kann es passieren, dass es zu einem _merge-conflict_ kommt. xref:#bild.eleven[]. Manchmal löst sich so ein Problem von selbst, ähnlich wie bei _This branch is out-of-date with the base branch_, da Dependabot die offenen PRs regelmäßig force-pushed, wenn es etwas neueres gibt. Ansonsten reicht bei einem Konflict oft ein `@dependabot recreate` in das Kommentarfeld des PRs und Dependabot erstellt das ganze neu.

.Leider gab es einen merge-conflict ([Screenshot GitHub])
[id="bild.eleven"]
image::screenshot_conflicts.jpg[]

Manchmal benutzen Projekte private oder öffentliche, aber Non-Standard Registries (wie z.B. die confluent Registry), die müsst ihr im `dependabot.yml` .xref:#listing.dependabot_registry[],  angeben. Die Secrets hierfür speichert ihr in GitHub in den Secrets ab.

[[listing.dependabot_registry]]
[source,text]
.Das Dependabot file mit einer privaten und einer öffentlichen Registry
----
version: 2
registries:
secret-private-registry:
type: maven-repository
url: https://joergi.io/registry/libs-release/
username: ${{ secrets.MY_REGISTRY_USERNAME }}
password: ${{ secrets.MY_REGISTRY_PASSWORD }}
confluent-repo:
type: maven-repository
url: https://packages.confluent.io/maven/
updates:
- package-ecosystem: "gradle"
  directory: "/my-project/"
    - secret-private-registry
    - confluent-repo
      labels:
      ......
----

Solltet ihr in einem Monorepo arbeiten, empfehle ich, dass ihr die Dependabot Zeiten der einzelnen Projekte auf jeden Fall versetzt, dadurch entstehen weniger Merge Konflikte.

Wie oben schon erwähnt, wäre es schön, wenn Dependabot die `settings.gradle.kts` auswerten würde. Dann liese sich in einem Monorepo mit 3 Projekten statt 3 `.toml` Files das ganze auf ein File reduzieren. Hierfür gibt es schon Dependabot Tickets <<DependabotTicket1>><<DependabotTicket2>> auf GitHub.

[bibliography]
== Quellen
- [[[BeispielProject,1]]] Beispiel Projekt: link:https://github.com/joergi/dependencies[]
- [[[SpringInitializer,2]]] Spring initializer: link:https://start.spring.io/[]
- [[[DependabotEcoystem,3]]] Dependabot Ecosystem: link:https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#package-ecosystem[]
- [[[GithubCli,4]]] Github CLI https://cli.github.com/[]
- [[[BeispielProjectMonorepo,5]]] Monorepo Beispiel: link:https://github.com/joergi/dependencies-monorepo[]
- [[[BeispielProjektMaven,6]]] Mavenbeispielprojekt link:https://github.com/joergi/dependencies-with-maven[]
- [[[DependabotTicket1,7]]] Dependabot Ticket: link:https://github.com/dependabot/dependabot-core/issues/6831[]
- [[[DependabotTicket2,8]]] Dependabot Ticket: link:https://github.com/dependabot/dependabot-core/issues/1164[]

== Über den Autoren
Jörg Liedl arbeitet als Senior Software Entwickler bei dem Wissenschaftsverlag SpringerNature AG, Das Java-Ökosystem ist seine Heimat, wo mittlerweile aber Kotlin das gute alte Java verdrängt hat. Er bevorzugt es Sachen zu automatisieren, um den eigenen Arbeitsalltag angenehmer zu gestalten.

image::ja_bild_joerg-liedl.jpg[]
