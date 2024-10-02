---
title: Dependabot Konfiguration
dates: 2024-10-03 00:24
categories: 
tags:
  - github
  - dependabot
---

In diesem Artikel möchte ich kurz meine Dependabot (DB) Konfiguration vorstellen.

>[!info]  Dependabot
>Dependabot ist eine Funktion in Github, welche dafür da ist, Abhängigkeiten in einem Projekt zu überwachen und diese notfalls zu Updaten.

Ich nutze Dependabot für folgende 2 Punkte:
- npm dependencies
- git submodules

Damit das ganze sauber und ohne manuelle eingreifen Funktioniert, sind 2 Konfigurationsdateien notwendig. 
Zu erst haben wir die Konfiguration des Bot's an sich. Diese wird in YAML geschrieben. 

```yaml
version: 2
updates:
- package-ecosystem: npm # hier überwachen wir mit npm
  directory: "/"
  schedule:
    interval: daily
  open-pull-requests-limit: 20
- package-ecosystem: "gitsubmodule" # und hier gucken wir uns die Submodule an
  directory: "/"
  schedule:
    interval: "daily"
```

Dann gehen wir das mal durch. 😁

### Was soll der Dependabot überwachen?

Hiermit wird angegeben, welche Version von DB wir haben möchten.
```yaml
version: 2
```

In dem Array "Updates" geben wir an, was genau wir alles durch den Bot überwacht haben wollen.
```yaml
updates: 
```

Der erste Eintrag in "Updates" ist für die npm dependencies. Dabei geben wir mit dem Keyword "directory" an, von wo aus wir die abhängigkeiten überwachen wollen. Mit "schedule.intervall" sagen wir, wie häufig diese Updates angeschaut werden sollen. Zu guter letzt geben wir mit "open-pull-request-limit" noch an, dass wir bis zu 20 pull-requests, also Paketupdates, gleichzeitig haben wollen. (Der default Wert liegt bei 5)
```yaml
- package-ecosystem: npm # hier überwachen wir mit npm
  directory: "/"
  schedule:
    interval: daily
  open-pull-requests-limit: 20
```

Mit dem Wissen, lässt sich dann auch der 2. Eintrag relativ schnell entschlüsseln. Das ganze noch einmal für git submodule.
```yaml
- package-ecosystem: "gitsubmodule" # und hier gucken wir uns die Submodule an
  directory: "/"
  schedule:
    interval: "daily"
```

### Ich hab da ein Update! Darf ich das einspielen?

Nun guckt der Bot regelmäßig nach Updates. Für die git submodule, muss er allerdings einen Pull Request (PR) auf machen, damit die Änderungen im submodule im main branch übernommen werden. Jetzt möchte ich aber nicht jedes mal wenn ich einen neuen Post veröffentliche mich bei GitHub anmelden und den PR freigeben, damit die Änderungen gemerged werden...

Deswegen nutzen wir hier GitHub Actions um alle Änderungen vom Bot frei zu geben.
```yaml
name: Dependabot auto-merge
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]' && github.repository == 'owner/my_repo'
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      - name: Enable auto-merge for Dependabot PRs
        if: contains(steps.metadata.outputs.dependency-names, 'my-dependency') && steps.metadata.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --merge "$PR_URL"
        env:
          PR_URL: ${{github.event.pull_request.html_url}}
          GH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

>[!danger]
>Hier ist Vorsicht geboten! Durch das automatische freigeben von Änderungen in einem submodule kann man sich schnell auch das Projekt zerschießen, da es evtl. irgendwelche breaking changes in dem Modul gibt. Da das submodule in diesem Projekt auf mein eigenes Repo verweist und hier "nur" Markdown-Content zu erwarten ist, nehme ich hier keine weiteren Änderungen vor.

Auf die GitHub Action gehe ich jetzt nicht weiter ein. Das kommt ein anderes mal. Bei mir ist's auch schon Spät und es wird Zeit den Rechner einmal ruhen zu lassen. 😅

Daher, liebe Grüße und bis zum nächsten mal, 
euer Appa