# Carnet de migration — spécification de construction

Construis une application web locale : **un seul fichier HTML autonome**, ouvrable
par double-clic, sans serveur, sans build, sans dépendance npm.

C'est un outil de travail personnel qui a deux usages successifs :
d'abord apprendre le métier de la migration sur des systèmes libres, ensuite
conduire de vraies missions de conseil avec la même méthode. La grille de
diagnostic et le registre de décisions sont l'actif ; le reste est du support.

---

## Contraintes techniques

- Un fichier `.html`, tout en ligne : CSS dans un `<style>`, JS dans un `<script>`.
- Aucun framework. JS natif, DOM direct.
- Seule ressource externe autorisée : Google Fonts (IBM Plex Sans + IBM Plex Serif).
  Prévois une pile de repli système si les polices ne chargent pas.
- Persistance : `localStorage`, une seule clé (`carnet.v1`), tout dans un objet JSON.
  Chaque lecture et chaque écriture dans un `try/catch` ; si le stockage échoue,
  l'application continue de fonctionner sans persister.
- Responsive jusqu'à 360 px. Respecte `prefers-color-scheme` **et** un bouton de
  bascule manuel à trois états (auto, clair, sombre) dont le choix est persisté.
- Accessibilité : onglets en `role="tab"` / `aria-selected`, boutons de notation
  en `aria-pressed` avec `aria-label`, focus visible partout,
  `prefers-reduced-motion` respecté.
- Aucun texte en dur dans le HTML pour les contenus listés plus bas : ils vivent
  dans des constantes JS en haut du script, pour être modifiables facilement.

## Modèle de données

```js
{
  missions: {
    [id]: {
      id, name, terrain,            // terrain = id d'un terrain, ou null
      created,                      // "AAAA-MM-JJ"
      answers: { [questionId]: { txt, flag } },   // flag: "ok" | "warn" | "risk" | null
      decisions: [ { id, title, q, ctx, opts, dec, cons, status, date } ],
      phases:  { [index]: { done, note, date } }
    }
  },
  current,        // id de la mission affichée
  skills: { [index]: 0|1|2|3 },   // hors mission : c'est une évaluation de la personne
  theme           // null | "light" | "dark"
}
```

`questionId` = `passe.id + "." + index`, stable si on réordonne l'affichage.

## Barre de mission (au-dessus des onglets)

Sélecteur des missions existantes, plus trois boutons : Nouvelle, Renommer,
Supprimer (avec confirmation). En dessous, une ligne discrète qui rappelle le
terrain rattaché, ou invite à en choisir un.

Si aucune mission n'existe au premier lancement, en créer une automatiquement :
« Palier 1 — Snipe-IT », terrain `snipeit`.

---

## Onglet 1 — Terrains

Cinq systèmes libres classés par difficulté croissante. Chacun s'affiche en carte
avec : nom, pile technique, une jauge de difficulté (◆ pleins / ◇ vides sur 5),
ce que le terrain force à apprendre, la décision emblématique (mise en valeur par
un filet vertical), l'angle IA produit, des pastilles d'axes couverts, et un
bouton pour le rattacher à la mission courante.

Quand on rattache un terrain à une mission dont le nom est encore générique,
renomme-la automatiquement « Palier N — Nom du terrain ».

### Données

**1 — Snipe-IT** · PHP / Laravel · gestion de parc informatique · difficulté 1
Axes : architecture générale.
*Ce qu'il force :* un périmètre qu'on tient dans la tête en une semaine. On y
apprend la mécanique complète — poser une façade, écrire des tests de
caractérisation sur du code qu'on ne comprend pas, extraire, basculer, retirer —
sans jamais être noyé par le domaine.
*Décision emblématique :* qu'est-ce qui est vraiment le cœur ici, et qu'est-ce
qui est un CRUD qu'on remplacerait par un produit du marché ? Sur un petit
système la réponse est gênante, et c'est l'exercice.
*IA produit :* assistant de saisie et classification automatique des actifs.
Domaine simple, donc on se concentre sur la seule question qui compte — où passe
la frontière entre déterministe et probabiliste.

**2 — Mautic** · PHP / Symfony · marketing automation · difficulté 2
Axes : architecture générale, event-driven.
*Ce qu'il force :* campagnes, segments, scoring, déclencheurs. Le métier est
nativement événementiel, donc les concepts d'EDA arrivent parce que le domaine
les impose et non parce qu'ils sont au programme. Le projet conduit lui-même une
modernisation de fond : on observe une vraie migration en cours plutôt que de
l'imaginer.
*Décision emblématique :* un contact traverse une campagne, un segment et un
scoring, chacun avec sa propre idée de ce qu'est un contact. Lequel est maître,
et que fait-on des trois autres ?
*IA produit :* scoring de leads et construction de segments. Le cas le plus
instructif pour l'évaluation, parce qu'il n'y a pas de vérité terrain — il faut
inventer sa mesure de qualité, ce qui est exactement le travail réel.

**3 — Dolibarr** · PHP · ERP et CRM · difficulté 3
Axes : architecture générale, DDD, décisions organisationnelles.
*Ce qu'il force :* le premier système trop gros pour tenir dans la tête. Des
dizaines de modules, une base partagée, aucune frontière explicite, et du code où
la logique métier vit dans les écrans. C'est ici que le diagnostic devient une
compétence et non une lecture.
*Décision emblématique :* le tiers — client, fournisseur, prospect, contact — est
la même table pour tout le monde. Où couper, et qu'est-ce qui casse dans l'heure ?
*IA produit :* extraction de données depuis des factures fournisseurs. Documents
réels, coût et latence mesurables, évaluation possible sur vérité terrain. Le bon
terrain pour apprendre à chiffrer une fonctionnalité IA avant de l'écrire.

**4 — Apache OFBiz** · Java · ERP complet, modèle de données universel · difficulté 4
Axes : architecture générale, DDD, event-driven, décisions organisationnelles.
*Ce qu'il force :* plus de mille entités dérivées d'un modèle de données
universel, et des milliers de services qui opèrent tous dessus. C'est l'anti-DDD
par construction : le système est conçu pour n'avoir aucune frontière de
contexte. Le travail devient trouver des contextes là où le design affirme qu'il
n'y en a pas.
*Décision emblématique :* qui possède le stock ? La commande, l'entrepôt et la
comptabilité y écrivent tous les trois aujourd'hui, et la partie double ne
pardonne rien.
*IA produit :* un agent qui navigue et interroge le modèle. La question n'est pas
ce que l'agent sait faire, mais ce qu'il a le droit de faire.

**5 — OpenEMR** · PHP · dossier patient informatisé · difficulté 5
Axes : architecture générale, DDD, event-driven, décisions organisationnelles, IA produit.
*Ce qu'il force :* le seul terrain où une erreur de migration est irréversible et
grave. Domaine clinique dense, contraintes réglementaires, données qu'on ne peut
ni perdre ni corriger silencieusement, et un système qu'on ne peut pas arrêter.
Les décisions y coûtent leur vrai prix.
*Décision emblématique :* qu'est-ce qu'on ne migre pas du tout, et qui accepte
formellement de porter l'ancien tel quel ? C'est la décision que les consultants
ratent le plus, et ici elle ne peut pas être esquivée.
*IA produit :* aide à la décision clinique — c'est-à-dire le cas où l'IA ne doit
surtout pas décider. La meilleure école possible sur les garde-fous,
l'explicabilité et le refus d'automatiser.

### Note à afficher en bas de l'onglet, sous le titre « L'angle mort »

Aucun de ces terrains n'apprend l'IA dans l'organisation. Un dépôt n'a pas
d'équipe : pas de revue à repenser, pas d'estimation à refaire, personne à qui
demander qui est responsable d'un bug que personne n'a tapé. Cet axe se travaille
sur une vraie équipe. C'est aussi celui qui différencie le plus d'un cabinet
d'architecture, donc il ne se remet pas à plus tard.

### Note à afficher en haut, sous le chapô

Ne pas sauter les deux premiers paliers parce qu'ils paraissent faciles. Ce qu'on
y apprend est la mécanique — interception, caractérisation, retrait — et c'est
elle qui manque quand on attaque un gros système directement.

---

## Onglet 2 — Diagnostic

Trente-et-une questions réparties en cinq passes. Pour chacune : le texte de la
question, trois boutons de signal exclusifs (OK « je sais » / ? « je crois
savoir » / ! « je ne sais pas », cliquer à nouveau désélectionne), et une zone de
texte libre pour la réponse.

En haut de l'onglet, un compteur permanent : combien de vertes, d'oranges, de
rouges, de non posées. Quand il y a des rouges, ajoute la phrase : « Les N rouges
sont le périmètre réel de ta première phase. »

Chaque passe affiche son titre, puis une ligne grise expliquant pourquoi elle
existe.

### Passe 1 — Le terrain
*Pourquoi :* avant le système, la demande. La moitié des migrations échouent
parce que personne n'a demandé pourquoi maintenant.
1. Qui souffre aujourd'hui, et de quoi exactement ? Un symptôme mesurable, pas « c'est du legacy ».
2. Qu'est-ce qui déclenche la demande maintenant plutôt qu'il y a deux ans ?
3. Que coûte l'inaction, chiffré par le client lui-même et pas par toi ?
4. Qui a déjà essayé, jusqu'où, et pourquoi ça s'est arrêté ?
5. Qui perd du pouvoir, du budget ou du périmètre si la migration réussit ?
6. Qui signe, et qu'est-ce qui lui vaudrait des ennuis si ça dérape ?

### Passe 2 — Le système tel qu'il est
*Pourquoi :* pas tel qu'il est documenté ni tel que l'équipe le décrit. Ce qui
tourne réellement en production.
1. Quels sont tous les points d'entrée réels : interface, API, batchs, files, intégrations, accès direct en base ?
2. Qui écrit dans les tables centrales ? Combien de composants distincts, et lesquels en dehors de l'application ?
3. Quelles zones changent chaque semaine, et lesquelles n'ont pas bougé depuis trois ans ?
4. Qu'est-ce qui tourne en production que plus personne ne revendique ?
5. Quels invariants métier le système protège-t-il réellement, par opposition à ceux qu'on croit protégés ?
6. Quelles données seraient irrécupérables en cas d'erreur de migration ?

### Passe 3 — Les frontières
*Pourquoi :* le travail de fond — trouver des contextes dans un système conçu
sans aucun, et décider de ce qu'on abandonne.
1. Quels termes du métier ont plusieurs sens selon l'endroit du système ? C'est le signal de frontière le plus fiable.
2. Quelles données sont maîtresses ici, et lesquelles ne sont qu'une copie d'ailleurs ?
3. Qu'est-ce qui fait gagner de l'argent au client, et qu'est-ce qui est générique et remplaçable ?
4. Si on coupait ici, qu'est-ce qui casserait dans l'heure ? Dans le mois ?
5. Qu'est-ce qu'on ne migre pas du tout, et qui accepte de le porter tel quel ?
6. Quelle frontière proposée traverse une équipe, un contrat ou un service externe ?

### Passe 4 — L'organisation
*Pourquoi :* une frontière technique que l'organisation ne porte pas se referme
en dix-huit mois.
1. Qui possède quoi aujourd'hui, officiellement, et qui le possède en réalité ?
2. Combien d'équipes doivent se coordonner pour livrer un changement de bout en bout ?
3. La structure d'équipe correspond-elle aux frontières visées ? Sinon, laquelle bouge, et qui décide ?
4. Qui maintient l'ancien système pendant la transition, et cette personne le sait-elle ?
5. Qu'est-ce qui empêche l'équipe de livrer en continu aujourd'hui ?
6. Où l'IA est-elle déjà utilisée dans cette équipe, et quelle pratique n'a pas été adaptée en conséquence ?
7. Qui a la connaissance que personne n'a écrite, et que se passe-t-il si cette personne part ?

### Passe 5 — La conduite
*Pourquoi :* une migration qui ne produit rien de visible avant six mois est
annulée au cinquième.
1. Quelle première tranche prouve la valeur en moins de huit semaines ?
2. Comment saura-t-on qu'on recule ? Le signal d'échec se définit maintenant, pas au moment où il se produit.
3. Quelle est la stratégie de retour arrière à chaque étape, et a-t-elle été testée ?
4. Combien de temps les deux systèmes coexistent, et qui paie la double maintenance ?
5. Qu'est-ce qui doit être vrai pour qu'on arrête délibérément la migration ?
6. Quel est le rythme de décision : qui tranche, en combien de jours, quand on découvre un obstacle ?

---

## Onglet 3 — Décisions

Registre d'entrées créables et supprimables. Chaque entrée a un titre éditable
et cinq zones de texte étiquetées, dans cet ordre : **La question à trancher**,
**Le contexte**, **Les options écartées**, **La décision**, **Ce qu'on assume de
perdre**. Plus un statut : ouverte / prise / écartée, matérialisé par la couleur
du filet gauche de la carte.

L'ordre des champs compte : la question avant la décision, et les options
écartées avant la décision. C'est ce qui empêche de justifier après coup.

État vide : « Aucune décision. Les rouges du diagnostic sont en général les
premières à instruire. »

---

## Onglet 4 — Étranglement

Six phases repliables, identiques quel que soit le système. Chacune affiche un
objectif, un critère de sortie, et un piège (mis en valeur par un filet rouge).
Chacune a une zone de notes et un bouton « Marquer le critère atteint » qui
horodate.

1. **Interception** — *Objectif :* poser une façade devant l'existant pour que
   tout trafic passe par un point qu'on contrôle. *Critère de sortie :* aucun
   appelant n'atteint plus directement l'ancien système, vérifié par les logs et
   non par déclaration. *Piège :* les accès directs en base et les batchs de
   nuit, qu'on découvre trois mois plus tard.
2. **Caractérisation** — *Objectif :* capturer le comportement réel à préserver,
   y compris les bugs dont dépendent des utilisateurs. *Critère de sortie :* une
   suite de tests qui décrit l'existant et qui échoue si on le change par
   accident. *Piège :* corriger les bugs qu'on découvre — c'est une autre
   mission, et elle rend la migration invérifiable.
3. **Extraction** — *Objectif :* sortir un seul contexte borné, le plus petit
   qui ait une valeur autonome. *Critère de sortie :* le nouveau contexte tourne
   avec ses propres données et passe la suite de caractérisation. *Piège :* en
   prendre deux parce que « c'est lié » — ils le sont toujours, c'est précisément
   le problème à résoudre.
4. **Bascule** — *Objectif :* router progressivement le trafic vers le nouveau,
   avec comparaison des deux sorties. *Critère de sortie :* tout le trafic sur le
   nouveau, retour arrière possible et déjà testé en conditions réelles.
   *Piège :* basculer d'un coup un vendredi — un retour arrière non répété
   n'existe pas.
5. **Découplage** — *Objectif :* supprimer la synchronisation entre ancien et
   nouveau : couche anticorruption, puis événements. *Critère de sortie :* les
   deux systèmes ne partagent plus aucune écriture, la cohérence à terme est
   assumée et documentée. *Piège :* laisser la synchronisation bidirectionnelle
   « en attendant » — elle devient permanente.
6. **Retrait** — *Objectif :* éteindre l'ancien code et l'ancienne donnée, la
   phase que presque tout le monde saute. *Critère de sortie :* le code est
   supprimé, pas désactivé ; les tables sont archivées puis retirées. *Piège :*
   s'arrêter à la bascule — un système à moitié migré coûte plus cher que
   l'ancien seul.

---

## Onglet 5 — Mes compétences

Hors mission. Vingt-quatre affirmations notées de 0 à 3, groupées par axe avec un
titre d'axe au-dessus de chaque groupe. Barème affiché : 0 jamais fait, 1 des
notions, 2 fait au moins une fois, 3 je peux l'enseigner.

En haut, une barre par axe qui se remplit selon le score, avec un verdict quand
l'axe est complet : **creuser** sous 42 %, **consolider** de 42 à 72 %,
**acquis** au-dessus. Tant que l'axe est incomplet, afficher « n/4 ».

**Architecture générale**
- J'écris huit scénarios de qualité chiffrés pour un système inconnu et je dis lesquels s'opposent.
- Je documente une architecture pour qu'un nouvel arrivant décide seul.
- Je repère les points de rupture avant qu'ils cèdent : charge, couplage, volumétrie.
- J'anime une revue d'architecture qui tranche au lieu de recenser les options.

**DDD**
- Je trouve des contextes bornés dans un modèle de données conçu pour être universel.
- Je trace une carte de contextes et je dis ce que chaque relation interdira dans deux ans.
- Je pose une frontière d'agrégat à partir des invariants, pas des tables.
- J'anime un event storming à douze sans qu'il devienne une réunion de plus.

**Event-driven**
- Je distingue un événement interne d'un contrat public qu'on ne pourra plus changer.
- J'implémente idempotence, outbox et livraison au moins une fois, et je le prouve par un test.
- Je conçois un processus long avec compensation plutôt qu'une transaction distribuée.
- J'explique à un dirigeant ce que la cohérence à terme change pour ses utilisateurs.

**Décisions organisationnelles**
- Je dis quelle frontière technique l'organisation actuelle est incapable de tenir.
- Je propose un découpage d'équipes et j'assume qui perd du périmètre.
- Je fais accepter à l'avance le signal qui dira qu'on recule.
- Je dis non à un périmètre intenable sans perdre la mission.

**IA dans le produit**
- Je place la frontière entre la partie déterministe et la partie probabiliste d'une fonctionnalité.
- Je chiffre coût et latence d'une fonctionnalité IA avant de l'écrire.
- Je construis un jeu d'évaluation qui détecte une régression de qualité, pas seulement une panne.
- Je définis ce qu'un agent a le droit de faire, et je le vérifie automatiquement.

**IA dans l'organisation**
- Je sais ce que devient une revue de code quand le code est majoritairement généré.
- Je mesure l'effet réel de l'IA sur une équipe, au-delà du ressenti et du volume produit.
- Je réponds à « qui est responsable de ce bug que personne n'a tapé ».
- Je construis un plan d'adoption sur 90 jours pour une équipe qui a les outils mais pas les pratiques.

---

## Onglet 6 — Livrable

Génère le contenu de la mission courante en Markdown, affiché dans une zone de
texte en lecture seule, avec un bouton Copier. Si l'API presse-papier échoue,
bascule en sélection du texte et dis-le à l'utilisateur.

Structure du Markdown : titre de la mission, date d'ouverture, terrain et palier,
puis le diagnostic passe par passe (chaque question suivie de son marqueur
`[connu]` / `[à confirmer]` / `[inconnu]` / `[non posée]` et de la réponse, ou
`_Sans réponse._`), puis les décisions, puis les six phases avec leur état.

C'est le rapport qui sera remis à un client. Écris-le pour être lisible tel quel,
sans retouche.

---

## Direction visuelle

Carnet d'ingénierie, pas tableau de bord SaaS. Sobre, dense, imprimable de l'œil.

- Fond papier gris-vert froid, panneaux légèrement plus clairs, encre presque
  noire teintée de vert. Trois couleurs de signal seulement : vert profond
  (acquis), ocre (à confirmer), terre brûlée (risque). Version sombre cohérente.
- IBM Plex Serif pour les titres, IBM Plex Sans pour le texte. Pas de troisième
  famille.
- Structure par filets et par espacement, pas par cartes arrondies à ombre douce.
  Les filets colorés à gauche des blocs portent une information (statut d'une
  décision, nature d'un encart), jamais une décoration.
- Numérotation uniquement là où le contenu est réellement une séquence : les
  paliers de terrain et les six phases. Nulle part ailleurs.
- Longueur de ligne sous 68 caractères pour le texte courant.
- Une seule chose spectaculaire : les barres d'axes qui se remplissent. Tout le
  reste reste calme.

## Critères d'acceptation

- Rechargement de la page : diagnostic, décisions, phases, notes, compétences,
  mission courante et thème sont tous retrouvés.
- Plusieurs missions coexistent sans fuite de données de l'une vers l'autre.
- Supprimer la dernière mission en recrée une automatiquement plutôt que de
  laisser l'application dans un état vide inutilisable.
- Le Markdown généré s'ouvre correctement dans un éditeur Markdown standard.
- Tout est utilisable au clavier seul, focus visible à chaque étape.
- À 360 px de large, aucune barre de défilement horizontale sur le corps de page.
