# VanoAtlas Web — déploiement sur GitHub Pages

Version publique de VanoAtlas : un site statique (aucun serveur), où chaque personne se
connecte avec son propre compte Google et ses données sont stockées uniquement dans
**son propre** Google Drive (dossier caché réservé à l'app). Toi (l'opérateur du site) ne
vois jamais les données de personne — elles ne transitent par aucun serveur.

Ce dossier contient :
- `index.html` — l'application complète (HTML + CSS + JS, un seul fichier)
- `privacy.html` — la politique de confidentialité (obligatoire pour la vérification Google)
- `README-DEPLOY.md` — ce guide

Ce que **toi seul** peux faire (ça nécessite ton propre compte Google / GitHub, je ne peux
pas le faire à ta place) :

## 1. Créer le dépôt GitHub + activer Pages

1. Sur [github.com](https://github.com), crée un nouveau dépôt (ex. `vanoatlas`), public.
2. Mets `index.html` et `privacy.html` à la racine du dépôt, commit + push.
3. Dans **Settings → Pages** du dépôt, choisis la branche `main` et le dossier `/ (root)`,
   sauvegarde.
4. Après une minute ou deux, ton site est en ligne à
   `https://TON-PSEUDO.github.io/vanoatlas/` — gratuit, sans domaine à acheter.

## 2. Créer un Client OAuth Google (type "Web application")

Sur [console.cloud.google.com](https://console.cloud.google.com), dans le même projet
Google Cloud que celui déjà utilisé pour la sync Calendar de l'app Mac (ou un nouveau
projet dédié à la version web, au choix) :

1. **APIs & Services → Identifiants → Créer des identifiants → ID client OAuth**.
2. Type d'application : **Application Web**.
3. **Origines JavaScript autorisées** : ajoute `https://TON-PSEUDO.github.io`
   (sans slash final). Ajoute aussi `http://localhost:5500` (ou le port que tu utilises)
   si tu veux tester en local avant de publier.
4. Copie le **Client ID** généré (il ressemble à `123456-abc.apps.googleusercontent.com`).
5. Dans `index.html`, cherche la ligne :
   ```js
   const GOOGLE_CLIENT_ID = 'REPLACE_WITH_YOUR_CLIENT_ID.apps.googleusercontent.com';
   ```
   et remplace la valeur par ton vrai Client ID. Recommit + repush.
6. Active aussi, dans **APIs & Services → Bibliothèque**, l'API **Google Calendar API**
   (déjà fait pour l'app Mac) — c'est la même API, pas besoin de la réactiver si c'est le
   même projet.

## 3. Configurer l'écran de consentement OAuth

Dans **APIs & Services → Écran de consentement OAuth** :

1. Type d'utilisateur : **Externe** (obligatoire pour que n'importe qui puisse se
   connecter, pas seulement des comptes de ton organisation).
2. Renseigne : nom de l'app (VanoAtlas), email de support, logo (optionnel).
3. **URL de la politique de confidentialité** : `https://TON-PSEUDO.github.io/vanoatlas/privacy.html`
4. **Scopes** : ajoute `.../auth/drive.appdata` et `.../auth/calendar.readonly`. Google
   classera chacun comme scope "sensible" — normal, c'est attendu, ça ne bloque rien à ce
   stade.
5. Tant que l'app est en statut **Test**, tu peux ajouter jusqu'à 100 comptes Google
   autorisés manuellement (onglet "Utilisateurs test") — utile pour toi + tes proches sans
   attendre de vérification.

## 4. Passer en public ("En production") — vérification Google

Comme tu veux que **n'importe qui** puisse s'inscrire, il faut soumettre l'app à la
vérification de Google avant de passer de "Test" à "En production". Points à savoir :

- Les deux scopes utilisés (`drive.appdata`, `calendar.readonly`) sont classés "sensibles",
  pas "restreints" — ils ne déclenchent **pas** l'audit de sécurité payant (CASA) que
  Google impose pour les scopes les plus intrusifs. Cet audit ne s'applique de toute façon
  qu'aux apps qui font transiter les données par un serveur ; VanoAtlas n'en a pas.
- La vérification demande : confirmer la politique de confidentialité (le lien du point 3),
  justifier l'usage de chaque scope, et fournir une courte vidéo de démonstration montrant
  le flux de connexion et à quoi sert chaque permission demandée.
- Compte un délai de quelques jours à quelques semaines selon la charge de Google — soumets
  tôt si tu vises une date de lancement précise.
- Tant que la vérification n'est pas terminée, les visiteurs verront un écran "Google n'a
  pas vérifié cette application" avec un bouton "Continuer quand même" — ça fonctionne,
  mais ça fait moins sérieux ; à réserver aux tout premiers testeurs.

Documentation officielle à jour (à revérifier au moment de soumettre, ces règles évoluent) :
https://support.google.com/cloud/answer/13463073

## 5. Limites connues de cette version web (par rapport à l'app Mac/Windows)

- **Pas de compilation LaTeX → PDF.** GitHub Pages ne peut pas exécuter `pdflatex`/`xelatex`
  (pas de serveur). Le bouton "Compile PDF" est remplacé par "Download .tex" : la personne
  télécharge le fichier source et le compile ailleurs (Overleaf, ou l'app Mac/Windows).
- **Pas de sync iCloud Calendar.** Cette fonctionnalité dépendait d'un serveur Python avec
  la librairie `caldav` ; elle reste disponible dans l'app Mac, mais n'a pas de sens côté
  navigateur (identifiants Apple à exposer côté client = mauvaise idée de sécurité). Le
  Google Calendar sync, lui, fonctionne bien côté web — direct depuis le navigateur.
- **Pas de photos de fond personnelles.** L'app Mac affiche tes photos (Sherbrooke,
  Montmorency) en arrière-plan ; publier ça sur un dépôt GitHub public les rendrait visibles
  par n'importe qui. La version web utilise un dégradé de couleurs à la place.
- **Un seul appareil à la fois pour l'écriture.** Les données sont sauvegardées comme un
  seul fichier JSON dans le Drive de chaque utilisateur ; si la même personne modifie
  l'app sur deux appareils/onglets en même temps sans recharger, la dernière sauvegarde
  écrase l'autre (pas de fusion intelligente). Pour un usage normal (un utilisateur, un
  appareil à la fois), ça ne pose pas de problème.

## 6. Vérifier que ça marche

1. Ouvre `https://TON-PSEUDO.github.io/vanoatlas/`.
2. Connecte-toi avec un compte Google ajouté comme "utilisateur test" (étape 3.5).
3. Crée une tâche, actualise la page → elle doit toujours être là (preuve que
   l'écriture/lecture Drive fonctionne).
4. Clique "Sync Google Calendar" → tes événements à venir doivent apparaître dans l'Agenda.
5. Vérifie dans ton Drive (drive.google.com → aucune trace visible, normal : le fichier
   est dans le dossier caché "appData", invisible par design — c'est la preuve que
   personne d'autre, y compris toi en tant qu'opérateur du site, ne peut le parcourir
   par accident).
