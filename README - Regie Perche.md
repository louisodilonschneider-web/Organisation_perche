# Régie Perche — README

Outil pour préparer et suivre l'intervention "initiation au saut à la perche" (6ème) et la sortie associée (3ème) : organisation du jour J (classes / créneaux / ateliers), suivi des autorisations et du meeting post-initiation, invités hors-classe, export PDF de chaque page.

C'est un fichier HTML unique (`Regie Perche - Organisation intervention.html`), sans installation : il s'ouvre directement dans un navigateur (double-clic, ou "Ouvrir avec" votre navigateur).

## 1. Fonctionnement général

- Toutes les données (classes, élèves, créneaux, meeting, suivi 3ème, invités) sont stockées dans une base **Firebase Realtime Database** et se synchronisent automatiquement, en temps réel, entre tous les appareils qui ouvrent ce fichier avec une connexion internet. Fermer l'onglet ou éteindre l'ordinateur ne fait donc rien perdre.
- La configuration Firebase (clés du projet `eps-pasteur`) est intégrée directement dans le fichier HTML, comme demandé. Ces clés identifient le projet mais ne sont pas un mot de passe : ce qui protège réellement les données, ce sont les **règles de sécurité** configurées côté console Firebase (voir section 3, importante).
- En bas de la barre latérale, un indicateur affiche l'état de la connexion :
  - **"Synchronisé (Firebase, temps réel)"** (vert) : tout fonctionne normalement.
  - **"Connexion à Firebase…"** ou **"Connexion Firebase perdue — nouvelle tentative…"** : problème réseau temporaire, l'app réessaie seule.
  - **"Accès refusé par Firebase (vérifiez les règles de sécurité)"** : les règles de sécurité de la base bloquent la lecture/écriture (voir section 3).
  - **"Mode fichier local (Firebase indisponible)"** : la connexion à Firebase a échoué (pas d'internet, SDK bloqué, projet supprimé…). L'app continue de fonctionner mais uniquement dans cet onglet, et **rien n'est sauvegardé automatiquement** — utilisez alors les boutons "Exporter (.json)" / "Importer (.json)" qui apparaissent dans ce mode pour ne rien perdre.

## 2. Structure des données dans Firebase

Tout est rangé sous un seul nœud racine `regiePerche` dans la Realtime Database, pour ne pas interférer avec d'éventuels autres projets sur le même compte Firebase :

```
regiePerche/
  config/main       → capacité meeting, ateliers, créneaux horaires
  schedule/main      → affectations classe ↔ créneau, groupes ↔ ateliers
  classes/<id>       → { niveau, nom, students: [ ... ] }
  suivi3e/<id>        → un élève de 3ème suivi individuellement
  invites/<id>        → un invité hors-classe (collègue, contributeur…)
```

Les champs "meeting" (feuille rendue, réponse, places souhaitées/attribuées, transport) vivent directement dans chaque élève, à l'intérieur de `classes/<id>/students`.

## 3. Règles de sécurité Firebase — À VÉRIFIER AVANT UTILISATION

**Ceci est important : ce fichier contient des données concernant des élèves mineurs** (noms, prénoms, autorisations de sortie, informations de transport). Comme la configuration Firebase est intégrée dans le fichier HTML, toute personne qui obtiendrait une copie de ce fichier a techniquement les moyens d'accéder à votre base de données Firebase — **la seule barrière réelle, ce sont les règles de sécurité** que vous définissez dans la console Firebase (Build → Realtime Database → Rules), pas la clé elle-même.

Deux niveaux possibles, du plus simple au plus sûr :

**a) Fonctionnement immédiat (le plus simple, à réserver à un usage strictement personnel)**

```json
{
  "rules": {
    "regiePerche": {
      ".read": true,
      ".write": true
    }
  }
}
```

Cela suffit pour que l'application fonctionne tout de suite. En contrepartie, **ne partagez ce fichier HTML avec personne d'autre que vous-même**, et ne le publiez jamais en ligne (site public, pièce jointe largement diffusée, etc.) : quiconque l'obtient peut lire et modifier toutes les données.

**b) Fonctionnement protégé par authentification (recommandé si vous envisagez de partager l'outil avec des collègues)**

Activez l'authentification Firebase (Authentication → Sign-in method → Email/Password, par exemple), créez un compte pour vous (et vos collègues si besoin), puis restreignez les règles :

```json
{
  "rules": {
    "regiePerche": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```

Cette option nécessite d'ajouter un écran de connexion à l'application, ce qui n'a pas été fait ici pour rester simple — dites-le-moi si vous voulez que je l'ajoute.

Sans configuration explicite des règles, une base Firebase Realtime Database toute neuve est **verrouillée par défaut** (aucune lecture/écriture) : c'est ce qui provoque le message "Accès refusé" dans l'indicateur de synchronisation. Il faut donc obligatoirement appliquer l'option (a) ou (b) ci-dessus au moins une fois dans la console Firebase pour que l'outil fonctionne.

## 4. Utilisation

- **Classes & élèves** : ajoutez vos classes de 6ème et leurs élèves, un par un ou par import CSV (bouton dédié — colonnes Nom / Prénom, avec détection automatique du séparateur `;`, `,` ou tabulation, et choix de l'ordre des colonnes).
- **Organisation du jour J** : définissez les créneaux horaires et les ateliers, affectez chaque classe à un créneau, répartissez les élèves entre ateliers pour la rotation, puis imprimez (bouton "Exporter en PDF" de la page, via l'impression du navigateur).
- **Meeting — 6ème** : liste de chaque classe avec, par élève, la feuille d'information rendue ou non, la réponse au meeting, le nombre de places souhaitées, le transport (personnel / besoin de covoiturage avec précision de la personne / propose du covoiturage avec le nombre de places et les élèves emmenés / autre), le nombre de places attribuées et leur numéro. Les lignes sont colorées (blanc/bleu/jaune/vert) selon que l'élève ne demande rien, attend une attribution, a une attribution partielle ou complète.
- **Sortie & meeting — 3ème** : suivi individuel (autorisation de sortie, argent reçu, souhait de meeting, places, transport).
- **Invités hors-classe** : attribution de places à des collègues ou contributeurs au projet, en dehors des classes.
- Chaque page dispose de son propre bouton d'export PDF (utilise l'impression du navigateur en mise en page paysage).

## 5. Sauvegarde manuelle

Même avec Firebase actif, vous pouvez à tout moment garder une copie locale : en mode "fichier local" (Firebase indisponible), les boutons "Exporter (.json)" / "Importer (.json)" apparaissent automatiquement dans la barre latérale et permettent de sauvegarder/recharger l'intégralité des données depuis un fichier `.json`.
