# Reverse-Engineer-Anything

Guide pédagogique **non officiel**, en français, pour comprendre et utiliser
**REA** (*Reverse Engineer Anything*), un outil open source qui permet à un
agent de code d'analyser des applications et des binaires, puis d'expliquer
leur fonctionnement en montrant les preuves sur lesquelles il s'appuie.

> Ce dépôt **ne contient pas le code de REA**. Le projet amont est
> [morluto/rea](https://github.com/morluto/rea), publié sous licence MIT. Ce
> guide est une documentation indépendante, rédigée en français, qui n'est ni
> affiliée ni approuvée par les auteurs de REA.

## Sommaire

- [Objectif de ce dépôt](#objectif-de-ce-dépôt)
- [Statut et avertissements](#statut-et-avertissements)
- [La rétro-ingénierie en bref](#la-rétro-ingénierie-en-bref)
- [Présentation de REA](#présentation-de-rea)
- [Concepts clés](#concepts-clés)
- [Le modèle en trois temps](#le-modèle-en-trois-temps)
- [Plateformes et prérequis](#plateformes-et-prérequis)
- [Installation et démarrage rapide](#installation-et-démarrage-rapide)
- [Premier exercice guidé](#premier-exercice-guidé)
- [Déroulé type d'une enquête](#déroulé-type-dune-enquête)
- [Fournisseurs d'analyse](#fournisseurs-danalyse)
- [Agents compatibles et configuration MCP](#agents-compatibles-et-configuration-mcp)
- [Observation web et Electron](#observation-web-et-electron)
- [Sécurité et confidentialité](#sécurité-et-confidentialité)
- [Cadre légal et éthique](#cadre-légal-et-éthique)
- [Bonnes pratiques](#bonnes-pratiques)
- [Limites et idées reçues](#limites-et-idées-reçues)
- [Glossaire](#glossaire)
- [FAQ](#faq)
- [Ressources](#ressources)
- [Contribuer](#contribuer)
- [Licence](#licence)

## Objectif de ce dépôt

Ce dépôt a un but simple : **expliquer, pas à pas et en français**, ce
qu'est la rétro-ingénierie assistée par agent, ce que fait REA, comment
l'installer, comment l'utiliser sans se tromper et dans quel cadre légal
et éthique il est raisonnable de le faire.

Il s'adresse à trois profils :

- les **débutants**, qui découvrent la rétro-ingénierie et veulent des
  définitions claires avant de toucher à un outil ;
- les **développeurs**, qui veulent comprendre le comportement d'une
  application dont ils n'ont pas le code source ;
- les **curieux de l'IA agentique**, qui s'intéressent à la façon dont un
  agent de code peut piloter des outils d'analyse via le protocole MCP.

## Statut et avertissements

- **Projet non officiel.** Les auteurs de REA ne sont pas impliqués dans
  ce guide.
- **État vérifié le 9 octobre 2026** à partir du dépôt amont. Le projet
  évolue vite : en cas de divergence, la documentation amont fait foi.
- **Aucune information inventée.** Ce guide s'appuie sur la documentation
  amont. Ce qui n'y figure pas est signalé comme non documenté.
- **Ce guide n'est pas un avis juridique.** Voir la section
  [Cadre légal et éthique](#cadre-légal-et-éthique).

## La rétro-ingénierie en bref

### Le chemin normal : du source au binaire

Un développeur écrit du **code source** lisible. Un **compilateur** le
transforme en **binaire** : une suite d'instructions que le processeur sait
exécuter, mais qu'un humain lit très difficilement. Au passage, une partie
de l'information disparaît : commentaires, noms de variables locales,
organisation d'origine du code.

### Le chemin inverse : du binaire à la compréhension

La **rétro-ingénierie** (ou rétroconception) consiste à remonter ce chemin
pour comprendre comment un logiciel fonctionne. Deux techniques de base :

- le **désassemblage** transforme les octets en instructions machine
  lisibles (l'assembleur) ;
- la **décompilation** tente de reconstruire un **pseudo-code** de plus
  haut niveau à partir de ces instructions.

Le pseudo-code produit est une **reconstruction plausible**, pas le code
d'origine. Les noms sont souvent génériques, la structure peut différer et
certaines parties restent ambiguës.

### Ce qu'on cherche concrètement

- des **chaînes de caractères** (messages, URL, noms de fichiers) ;
- des **symboles** (noms de fonctions, quand ils sont conservés) ;
- des **références croisées** : qui appelle quoi, qui lit quelle donnée ;
- un **graphe d'appels** et un **graphe de flot de contrôle** pour suivre
  l'enchaînement logique d'une fonctionnalité.

## Présentation de REA

REA donne à un agent de code **une manière cohérente d'enquêter sur un
logiciel**. Sans outil de ce type, l'opérateur doit choisir un outil,
apprendre son interface, transférer les preuves d'un programme à l'autre
et décider lui-même de la suite. REA confie ce travail à l'agent via des
commandes, des compétences (*skills*), des résultats structurés et des
workflows d'enquête reproductibles.

D'après la documentation amont, REA couvre notamment :

- l'analyse native approfondie (via Hopper ou Ghidra), avec des
  « dossiers de fonction » ;
- le tri de binaires gérés PE/CLI (.NET) sans exécution ;
- des enregistrements de preuves reproductibles (*Evidence v2*) ;
- la capture contrôlée du comportement d'un processus ;
- l'observation passive de sites web, de pages Electron et d'inspecteurs
  V8 Node/Electron ;
- la reconstruction bornée de code JavaScript et de *source maps*.

Les mêmes capacités sont accessibles de deux façons : en **ligne de
commande** (CLI) et via un **serveur MCP** que votre agent appelle.

## Concepts clés

- **Agent de code** : un assistant IA capable de lire des fichiers, lancer
  des commandes et appeler des outils externes.
- **MCP (Model Context Protocol)** : un protocole standard qui permet à un
  agent de découvrir et d'utiliser des outils fournis par un serveur local.
- **Skill** : un ensemble d'instructions que l'agent charge pour savoir
  quand et comment utiliser un outil. REA installe un skill de routage avec
  son enregistrement MCP.
- **Fournisseur (*provider*)** : le moteur d'analyse qui répond réellement
  aux questions (Hopper, Ghidra, observateur web, etc.). REA fournit
  l'interface commune ; le fournisseur fait le travail.
- **Evidence v2** : le format de preuve de REA. Chaque résultat réussi est
  enregistré avec l'identité de l'artefact et du fournisseur, un niveau de
  confiance, des limites et des emplacements.
- **Inconnu explicite** : quand REA ne sait pas, il le dit. Un résultat
  tronqué ou ambigu n'est jamais présenté comme complet.

## Le modèle en trois temps

La documentation amont résume la démarche en trois verbes :

1. **Décompiler** : ouvrir une application et récupérer du code lisible,
   des chaînes, des noms et d'autres indices sur son fonctionnement.
2. **Comprendre** : suivre le code d'une partie de l'application à une
   autre jusqu'à pouvoir expliquer comment une fonctionnalité marche.
3. **Recréer** : utiliser ce qui a été compris pour écrire une
   fonctionnalité dans votre propre produit, adaptée à votre pile
   technique.

REA montre **comment il est arrivé à ses conclusions**. Il ne prétend ni
retrouver le code source d'origine, ni cloner automatiquement une
application. La recréation (troisième temps) est faite par l'agent avec ses
outils habituels d'édition et de test.

## Plateformes et prérequis

D'après la documentation amont (vérifiée le 9 octobre 2026) :

- macOS 12 ou plus récent ;
- Ubuntu 24.04+, Fedora 41+ ou Arch Linux 64 bits ;
- Windows x64, uniquement pour une prise en charge **expérimentale**,
  limitée à Ghidra et aux applications PE natives x86-64 ;
- Node.js 22.19+ ou 24.11+, et npm.

> **Attention pour Termux/Android.** Android n'apparaît pas dans la liste
> des plateformes prises en charge. Termux convient très bien pour
> **rédiger et publier ce guide**, mais il ne faut pas supposer que REA
> s'y exécute. Utilisez une machine listée ci-dessus.

Les analyses natives profondes exigent un moteur séparé : **Hopper** ou
**Ghidra**. L'inspection statique de JavaScript et de .NET se fait sans eux.

## Installation et démarrage rapide

### Méthode recommandée : l'assistant de configuration

```bash
npx --yes rea-agents@latest setup
```

L'assistant détecte les agents installés, affiche son plan complet,
n'active rien par défaut et demande une approbation finale avec **Non**
comme réponse par défaut. Les enregistrements sont additifs, précédés
d'une sauvegarde, puis relus après écriture. Relancer la commande est sûr.

Pour examiner le plan sans rien modifier :

```bash
npx --yes rea-agents@latest setup --dry-run
```

Après la configuration, **redémarrez votre agent** pour qu'il charge REA.

### Vérifier l'installation

```bash
npx -y rea-agents@latest doctor
```

La commande `doctor` est en lecture seule. Elle distingue les hôtes non
pris en charge, les dépendances manquantes, l'absence de moteur d'analyse,
la dérive de configuration et les contrôles réussis.

### Installer la commande rea

Si vous préférez une commande disponible en permanence :

```bash
npm install --global rea-agents
rea setup
rea doctor
```

Mise à jour d'une installation globale :

```bash
rea upgrade
```

### Désinstaller

```bash
rea uninstall
```

Cette commande retire uniquement les enregistrements MCP et le skill gérés
par REA. Hopper, Node.js, vos preuves et vos autres serveurs MCP sont
conservés. L'option `--purge-data` supprime en plus le cache et l'état
locaux de REA.

## Premier exercice guidé

Objectif : obtenir une première explication sourcée, sur un logiciel dont
vous avez le droit de faire l'analyse.

1. Vérifiez l'installation avec `rea doctor`.
2. Choisissez une application **que vous possédez** ou que vous avez écrite.
3. Demandez à votre agent, par exemple :

   ```text
   Analyse l'application située à /chemin/vers/mon-app. Explique comment
   fonctionne sa recherche hors ligne, cite les preuves utilisées et indique
   clairement ce qui reste inconnu. Ne modifie aucun fichier.
   ```

4. Relisez la réponse en vérifiant que chaque affirmation renvoie à une
   preuve. Si une affirmation n'est pas sourcée, demandez à l'agent de la
   justifier ou de la retirer.

Pour une analyse ponctuelle depuis le terminal :

```bash
npx -y rea-agents@latest analyze /chemin/vers/mon-app
```

Pour une application JavaScript ou Electron (dossier ou archive `.asar`),
la documentation amont exige l'option `--approved` et une racine d'entrée
approuvée par l'opérateur :

```bash
export REA_INVESTIGATION_INPUT_ROOTS_JSON='["/chemin/vers/mes-apps"]'
rea analyze-javascript-application /chemin/vers/mes-apps/app.asar \
  --approved --json
```

## Déroulé type d'une enquête

La documentation amont décrit un enchaînement en six étapes pour la
demande « comprendre une fonctionnalité puis la recréer » :

1. **Ouvrir et identifier le binaire** : `open_binary`, `binary_overview`.
2. **Trouver des indices** : `search_strings`, `search_procedures`,
   `list_names`.
3. **Relier les indices au code exécutable** : `find_xrefs_to_name`,
   `xrefs`, `procedure_callers`.
4. **Reconstruire le flot de contrôle** : `get_call_graph`,
   `procedure_callees`, `procedure_info`.
5. **Décompiler les routines utiles** : `procedure_pseudo_code`,
   `procedure_assembly`, `batch_decompile`.
6. **Construire la fonctionnalité** dans votre projet.

REA assure les étapes 1 à 5. L'étape 6 est réalisée par l'agent avec ses
outils d'édition de fichiers et de test.

## Fournisseurs d'analyse

### Hopper

Hopper est une application de bureau **distincte**, avec sa propre licence.
REA ne l'inclut pas, mais l'assistant peut l'installer après approbation.
Un mode démonstration gratuit existe, avec des limites fixées par l'éditeur.
Une licence payante est facultative.

### Ghidra

La prise en charge de Ghidra est « apportez le vôtre ». Elle vise la
version exacte **12.1.2** avec un **JDK 21 complet de 64 bits** sous Linux
x64. REA ne télécharge jamais Ghidra et n'installe jamais Java. Vous
indiquez les chemins :

```bash
export GHIDRA_INSTALL_DIR=/chemin/vers/ghidra_12.1.2_PUBLIC
export JAVA_HOME=/chemin/vers/jdk-21
rea doctor --json
rea setup
rea providers --json
```

Le fournisseur Ghidra est en **lecture seule** : l'état de l'interface
graphique et les modifications restent indisponibles.

### Et IDA ?

IDA/Hex-Rays n'est **pas pris en charge** aujourd'hui. Il figure
seulement dans la feuille de route amont comme fournisseur à évaluer.

### Choisir un fournisseur

```bash
rea providers --json
rea analyze /chemin/vers/programme --provider hopper
```

Il n'y a **jamais de bascule silencieuse** vers un autre fournisseur en
cas d'échec : l'erreur est remontée avec des diagnostics.

## Agents compatibles et configuration MCP

L'assistant détecte Claude Code, Claude Desktop, Codex, Cursor, Gemini CLI,
Windsurf et Devin. Il configure automatiquement les six premiers. Devin est
signalé mais laissé inchangé, faute de frontière de configuration MCP locale
documentée.

Tout agent capable de lancer un serveur MCP local peut utiliser REA avec la
configuration manuelle suivante, où la version est épinglée :

```json
{
  "mcpServers": {
    "rea": {
      "command": "npx",
      "args": ["-y", "rea-agents@3.1.0", "mcp"]
    }
  }
}
```

La version `3.1.0` est celle affichée dans la documentation amont à la date
de vérification. Consultez la documentation amont pour la version actuelle.

## Observation web et Electron

L'observation de sites est **désactivée par défaut**. Elle exige un point
d'accès CDP en boucle locale et des origines explicitement approuvées :

```bash
export REA_BROWSER_OBSERVE_ENABLED=true
export REA_BROWSER_CDP_ENDPOINTS_JSON='["http://127.0.0.1:9222"]'
export REA_BROWSER_ALLOWED_ORIGINS_JSON='["http://127.0.0.1:3000"]'
```

D'après la documentation amont, l'inspection est **passive** : REA
n'évalue pas le JavaScript de la page, ne navigue pas et ne clique pas.
Les identifiants, cookies, en-têtes d'autorisation et valeurs de stockage
ne sont pas conservés. Les scénarios contrôlés (Playwright) forment une
frontière distincte, explicitement permissionnée.

## Sécurité et confidentialité

- **Analyse locale.** REA n'envoie pas l'application à un service d'analyse
  hébergé.
- **Votre fournisseur de modèle compte.** Les résultats transmis à votre
  agent sont soumis à la politique de données de son fournisseur. Lisez-la
  avant d'analyser un logiciel confidentiel.
- **Capacités dynamiques désactivées par défaut.** Elles demandent une
  politique d'opérateur **et** une approbation explicite à chaque appel.
- **Pas de bac à sable.** La documentation amont précise que les
  fournisseurs et les cibles lancées s'exécutent avec les permissions de
  l'utilisateur courant.
- **Recommandation de ce guide** (non issue de la documentation amont) :
  analysez les binaires non fiables dans une machine virtuelle dédiée.
- **Signaler une faille de REA** : suivre la procédure privée du fichier
  `SECURITY.md` du dépôt amont.

## Cadre légal et éthique

> Cette section est informative. **Ce n'est pas un avis juridique.** Les
> règles varient selon le pays et le contexte. Vérifiez les textes en
> vigueur et consultez un juriste pour un cas précis.

### En France et dans l'Union européenne

L'article L122-6-1 du Code de la propriété intellectuelle, qui transpose
la directive européenne sur la protection des programmes d'ordinateur,
prévoit plusieurs exceptions au droit d'auteur sur un logiciel :

- le **fonctionnement normal** : l'utilisateur légitime peut, dans certaines
  limites, traduire, reproduire ou adapter le logiciel pour l'utiliser
  conformément à sa destination, y compris pour corriger des erreurs
  (alinéa I) ;
- l'**observation et l'étude** : l'utilisateur légitime peut observer,
  étudier ou tester le fonctionnement ou la sécurité du logiciel afin de
  déterminer les idées et principes qui sont à la base de ses éléments
  (alinéa III), dans le cadre des opérations qu'il est en droit d'effectuer ;
- la **décompilation pour l'interopérabilité** : elle est possible sans
  autorisation de l'auteur lorsqu'elle est **indispensable** pour obtenir
  les informations nécessaires à l'interopérabilité d'un logiciel créé de
  façon indépendante, et sous conditions strictes (alinéa IV).

Parmi ces conditions, les informations obtenues ne peuvent pas être
utilisées à d'autres fins que l'interopérabilité, ni communiquées à des
tiers hors du besoin d'interopérabilité.

### Licences logicielles et contrats

- Lisez le **contrat de licence** (CLUF) du logiciel étudié : il peut
  encadrer certains usages. La portée de ce qu'il peut valablement
  interdire dépend de la loi applicable.
- Une licence **open source** n'autorise pas tout pour autant : les
  conditions de redistribution et de réutilisation du code restent
  applicables.
- Les **mesures techniques de protection** (DRM) font l'objet de règles
  propres. Ne les contournez pas sans avoir vérifié ce qui est permis.

### Comprendre n'est pas copier

Le droit d'auteur protège l'**expression** d'un programme, pas ses
**idées et principes**. La Cour de justice de l'Union européenne a jugé,
dans l'affaire SAS Institute contre World Programming (C-406/10), que la
fonctionnalité d'un programme n'est pas protégée en tant que telle. À
retenir : **recréer un comportement observé** n'est pas **recopier du code
décompilé**. Ne redistribuez pas de code reconstitué à partir d'un logiciel
propriétaire.

### Hors de l'Union européenne

D'autres pays ont d'autres règles. Aux États-Unis, par exemple, l'article
1201 du DMCA encadre le contournement des mesures de protection, avec des
exceptions limitées. Vérifiez le droit applicable à votre situation.

### Accès aux systèmes et vulnérabilités

- Le Code pénal sanctionne l'accès ou le maintien frauduleux dans un
  système de traitement automatisé de données (article 323-1 et suivants).
  N'observez que des systèmes qui vous appartiennent ou pour lesquels vous
  disposez d'une **autorisation écrite**.
- Si vous découvrez une vulnérabilité, privilégiez la **divulgation
  responsable** auprès de l'éditeur. L'article L2321-4 du Code de la
  défense prévoit un cadre pour signaler de bonne foi une vulnérabilité à
  l'autorité nationale de sécurité des systèmes d'information (ANSSI).

### Périmètre autorisé : liste de contrôle

- [ ] Je possède le logiciel, ou j'en ai une licence légitime.
- [ ] J'ai lu le contrat de licence.
- [ ] Mon objectif est clair : étude, sécurité, interopérabilité.
- [ ] Je n'ai pas l'intention de redistribuer du code décompilé.
- [ ] Je n'analyse aucun système tiers sans autorisation écrite.
- [ ] J'ai vérifié la politique de données de mon fournisseur de modèle.

## Bonnes pratiques

- **Partez d'une question précise** : « comment fonctionne la recherche
  hors ligne ? » donne de meilleurs résultats que « analyse tout ».
- **Exigez les preuves.** Une conclusion sans preuve est une hypothèse.
- **Distinguez l'observé de l'inféré.** Un code présent dans un paquet n'est
  pas la preuve qu'il s'exécute.
- **Travaillez sur une copie** du logiciel étudié.
- **Isolez** les binaires non fiables (machine virtuelle).
- **Versionnez vos notes** : conservez vos questions, vos preuves et vos
  conclusions dans un dépôt dédié.
- **Relisez les plans d'installation** avant de les approuver.

## Limites et idées reçues

- **« REA retrouve le code source d'origine. »** Faux. Aucun décompilateur
  ne peut le garantir. Vous obtenez du pseudo-code, de l'assembleur, des
  symboles, des chaînes et des relations.
- **« REA clone une application toute seule. »** Faux. La recréation est un
  travail de l'agent, guidé par vos exigences.
- **« Un résultat vide prouve l'absence. »** Faux. Les données manquantes ou
  tronquées restent des inconnues, jamais des preuves d'absence.
- **« Tout fonctionne partout. »** Faux. La prise en charge dépend de la
  plateforme, du fournisseur et du type de cible.

## Glossaire

- **Assembleur** : représentation lisible des instructions machine.
- **Binaire** : programme compilé, exécutable par la machine.
- **CDP** : *Chrome DevTools Protocol*, utilisé pour observer un navigateur.
- **CFG** : graphe de flot de contrôle, c'est-à-dire l'enchaînement des
  blocs d'instructions d'une fonction.
- **Compilation** : traduction du code source en binaire.
- **Décompilation** : reconstruction d'un pseudo-code depuis un binaire.
- **Désassemblage** : traduction des octets en instructions lisibles.
- **Dossier de fonction** : synthèse d'une fonction (pseudo-code,
  assembleur, appels, références, chaînes).
- **MCP** : *Model Context Protocol*.
- **Pseudo-code** : code reconstruit, proche d'un langage de haut niveau.
- **Symbole** : nom associé à une fonction ou à une donnée.
- **Xref** : référence croisée entre deux éléments du programme.

## FAQ

### Hopper doit-il être lancé avant REA ?

Non. D'après la documentation amont, REA le démarre quand une opération en
a besoin. Une instance déjà ouverte est aussi prise en charge.

### REA envoie-t-il mon application sur Internet ?

REA n'a pas de service d'analyse hébergé. Cependant, votre agent et son
fournisseur de modèle ont leur propre politique de données.

### Puis-je l'utiliser sous Windows ?

Partiellement : la prise en charge est expérimentale et limitée à Ghidra
et aux applications PE natives x86-64.

### Puis-je utiliser REA depuis Termux ?

Ce n'est pas documenté. Android ne figure pas parmi les plateformes
prises en charge listées par le projet amont.

### Où signaler une erreur dans ce guide ?

Ouvrez une *issue* sur ce dépôt en indiquant la section concernée et, si
possible, le lien vers la documentation amont qui contredit le guide.

## Ressources

- [Dépôt amont morluto/rea](https://github.com/morluto/rea)
- [Paquet npm rea-agents](https://www.npmjs.com/package/rea-agents)
- [Documentation d'installation](https://github.com/morluto/rea/blob/main/docs/installation.md)
- [Observation du navigateur](https://github.com/morluto/rea/blob/main/docs/browser-observation.md)
- [Prompts MCP guidés](https://github.com/morluto/rea/blob/main/docs/mcp-prompts.md)
- [Politique de sécurité de REA](https://github.com/morluto/rea/blob/main/SECURITY.md)
- [Hopper](https://www.hopperapp.com/)
- [Ghidra](https://github.com/NationalSecurityAgency/ghidra)

## Contribuer

Les contributions sont bienvenues : corrections, clarifications, exemples
et traductions. Avant de proposer une modification :

1. ouvrez une *issue* pour en discuter ;
2. vérifiez toute affirmation technique dans la documentation amont ;
3. respectez les règles `markdownlint` par défaut, y compris la limite de
   80 colonnes ;
4. rédigez en français, avec un ton clair et pédagogique.

## Licence

Ce guide est publié sous licence [MIT](LICENSE). REA est un projet distinct,
également sous licence MIT. Hopper et Ghidra ont leurs propres licences.
