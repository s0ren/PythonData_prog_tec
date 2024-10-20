---
title: "Readme"
author: Søren Magnusson

lang: da
format:
  html:
    toc: true
    code-fold: true
  # pdf:
  #   toc: true
  #   number-sections: true
  #   colorlinks: true
  typst:
    toc: true
    number-sections: false
    pagesize: a4
  docx:
    toc: true
    pagesize: a4
---

# Min opsætning af quarto miljø

## Motivation

Quarto skal bruges til at håndtere dokumenter, vejledninger og slides.

Dokumenterne i denne sammenhæng indeholder ofte en masse kode eksempler, og skal gerne kunne versionsstyres, med *git*.

Derfor er markdown velegnet som format, men der er em masse add-ons, og forskellige måder at rendre *markdown*. Her kommer *pandoc*, mv ind i billedet. Endelig er der også *jupyter notebooks*, som kombinerersektioner med kode, der afvikles inde i dokumentet, sammen andre sektioner i markdown. Jeg vil også gerne have diagrammer i *PlantUml* eller *Mermaid*.

Alle disse features, og sikkert mange flere, vil jeg gerne med i en samlet pakke, og her tilbyder ***quarto*** noget.

Da *Quarto* har plugins til *VS Code* med visuelle værktøjer, batch rendering, tænker jeg at bygge op med dette.

Jeg vil gerne have en devcontainer, (et slags projekt *VSCode* kører i en (*Docker-*)Container)

## Komponenter

### Kærne image

En docker container skal køre med et image, en nogen man laver fra bunden, eller et man bygger videre på.

Man kan lave udvidelserne i VSCodes `dencontainer.json`, men så skal alle udvidelserne installeres hver gang man genstarter containeren. Hvis man laver et "sub-image, defineret i `Dockerfile`, så caches alle tilføjelserne.

Jeg har kigget lidt rund, og for at slippe for *R*[^1] og andet der er overflødigt for mig, har jeg valgt at køre med *quartos* eget image, som basis. Images ligger godt nok under deres dev sektion, men jeg tænker det er ok.

[^1]: Sådan nogen som Rocker har også en løsning med meget R

Se <https://github.com/quarto-dev/quarto-cli/releases/tag/v1.5.57>

I [`./.devcontainer/Dockerfile`](./.devcontainer/Dockerfile) bygges quarto-dev imaget, som skulle være det seneste (man kan måske slutte `release`).

Jeg har lavet en `Dockerfile`, der bygger videre på *quarto-dev*. Jeg har hentet inspiration fra 
- <https://www.avonture.be/blog/docker-quarto/#create-your-own-docker-image> og 
- <https://github.com/analythium/quarto-docker-examples>

Desuden installeres: 
- python3 
- pip 
- git og quarto udvidelserne 
- tinytex 
- chromium

#### Grafer

Jeg vil gerne bruge *plantUml* til nogle grafer, men også gerne *Mermaide*, som er indbygget.

Derfor har jeg tilføjet `plantuml` til `apt install`. PlantUml har afhængighed til Java Runtime. Den installeres med automatisk hvis ikke den er der i forvejen. 

Jeg har haft lidt bøvl med at få qaurto til generere grafer korrekt i `pdf`(`latex`). Så jeg har eksperimenteret med at udskifte kilden i `FROM `-linien øverst i `Dockerfile`.
Dels har jeg prøvet at bruge den nyeste, version 1.6.10, i stedet for release v. 1.5.57.

Umiddelbart så vejledningerne ud til at jeg skulle installere en extension til qaurto, som installeres med 

  quarto install extension pandoc-ext/diagram

og tilføjede

  RUN quarto install --no-prompt --update-path extension pandoc-ext/diagram

Men det virkede ikke rigtigt, men var tilsyneladende unødvendigt, måske fordi jeg bruger `quarto-full` image'et.  
Det gør en *forskel* at tilføje

```
---
...

filters:
  - diagram
diagram:
  cache: false
  ---
```
i dokumentets _yaml_-header

Så, pt. virker både *plantUml* og *Mermaid* i *html*, *revealJS*, *typst* (*pdf*), og *docx* (*ms-wor*d).
Det viser sig at hvis dokumenterne ligger i undermapper, skal digrams installeres igen i undermappen,

    quarto add pandoc-ext/diagram

eller 

    quarto add pandoc-ext/diagram --no-prompt

hvis du ikke gider svare på noget...

### Devcontainer config

VS Codes devcontainer definerer en container som et udviklingsmiljø, som kan køres fra VS Code. Afviklingen foregår inde i containeren og de filer man arbejder med mappes til værts-systemet. Her på min maskine, windows. Devcontainere kan også køres via web, f.eks. fra github workspaces, eller self-hosted.

I dette projekt snakker vi primært om filen [`.devcontainer/devcontainer.json`](./.devcontainer/devcontainer.json)

### Pakker på containeren

Efter image-filen til contaneren er bulded, kan *dev*-contaneren, installere noget ting *post build*. f.eks python pip eller node js pakker.

Disse indlæses eftercontaneren er startet, men gøres hver gang containere genstartes.\
Fordele og ulemper, gynger og karuseller.

Det kan være mere flektibelt at gøre det her, men og langssommere...

### VSCode udviddelser

Når man kører en devcontainer på VSCode, kan

Min `devcontainer.json` er inspireret af <!-- TODO: genfind link -->

Jeg har tilføjet følgende:

-   ...

-   Jeg går efter streetside softwares udgave,"streetsidesoftware.code-spell-checker",

    -   også dansk sprog\
        "streetsidesoftware.code-spell-checker-danish"

    Spell Right, har jeg også kigget på, men den kræver tilsyneladende mere installation på linux og ældre windows

    Der er nu response på disse: jeg fåår response på stavefleg...?

    Virker endnu ikke i Quarto visuel editor. Den bruger et helt andet komponent, som jeg ikke har fundet frem til. Ej heller hvordan det ændres, så dansk er en mulighed.

### Quarto project opsætning

Lige nu lægger jeg en minimalt projekt opsætning med i repoet.  
I filen `./_qaurto.yml`, har jeg defineret:  

- `execute-dir`  til `project`, 
  altså at render prosesserne udføres i projektets root. Det betyder at extensions _ikke_ behøver at installeres i undermapper.
- `output-dir` er sat til `out`, så alle filer der produceres af render (og midlertidige filer), lande under `out`.
- `diagrams` defineres i `filters:`, så det fungerer globalt.
- `render` (er listen af de filtyper der skal rendres) er sat til `*.qmd` og `readme.md`

Lav meget gerne selv et quarto projekt med kommandoerne

Interaktivt:

    quarto create project

Alt på en gang:

    quarto create default ny_projekt_mappe

Du kan også trykke {{< kbd F1>}}, og vælge `Quarto: Create project`.

<!-- TODO: lav kapitel om brugervejledning -->

## Forudsætninger

For at bruge denne pakke skal man have \* en computer der lever op til nogle hardwarekrav, \* nogle features installeret på systemet \* viden om f.eks markdown og lignende

### Systemkrav

#### Hardware

En computer med

-   nok ram
-   rimelig processor
-   fornuftig harddisk

#### System software

Installeret på sytemet

- *Docker* eller lignende f.eks *PodMan*.
- *WLS*, når det er på windows
- *VS Code*

### Nyttig viden

Du bør vide noget om 
- *MarkDown* 
- web og html 
- Adobes *Portable Document Format* 
- *Git* 
- måske kommandoer, *powershell*, *bash* og *make*

## Historik

Projektet er lige nu meget eksperimentelt, så en egentlig historik kommer nok senere \[TODO: Lav historik\]

For mit eget overbliksskylde laver jeg en separet TODO.md. Se [TODO.md](./TODO.md)

## Referencer

-   <https://quarto.org/>
-   <https://github.com/quarto-dev/quarto-cli/discussions/5129>
-   <https://github.com/quarto-dev/quarto-cli/releases/tag/v1.5.57>
-   <https://www.avonture.be/blog/docker-quarto/>
-   <https://github.com/analythium/quarto-docker-examples/tree/main>
-   [https://medium.com/\@rami.krispin/setting-a-dockerized-python-development-environment-template-de2400c4812b](https://medium.com/@rami.krispin/setting-a-dockerized-python-development-environment-template-de2400c4812b){.uri}
-   <https://rocker-project.org/>