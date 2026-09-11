# Livret 21 jours — Jean-Philippe Ackermann

Application post-conférence AcéKom déployée pour **Jean-Philippe Ackermann** (JPA), conférencier sur l'optimisme managérial. Premier client AcéKom.

- **En ligne :** https://jpa.acekom.fr
- **Objectif prioritaire du client :** référencement Google et visibilité publique (ça explique le bouton d'avis Google et le choix du sous-domaine public).
- **Propriété :** toute l'infra reste chez AcéKom (GitHub, Supabase, Google Cloud). JPA n'a accès ni au repo ni à la base. Seul son compte Brevo lui appartient.

---

## ⚠️ À lire avant de toucher au code

- **Aucune modification de code avant une conférence de JPA.** La stabilité avant l'événement passe devant toute amélioration de dernière minute.
- **Ne jamais renommer la clé interne `donner`.** Elle mappe la progression stockée dans Supabase pour tous les utilisateurs existants. La renommer casse les parcours en cours.
- **GitHub Pages est sensible à la casse.** Tous les noms de fichiers (slides, images) doivent être en minuscules exactes. Une majuscule passe en local et casse en ligne.
- **Aucune URL en dur.** L'app construit `CLEAN_URL` à partir de `window.location.origin + window.location.pathname`. Un changement de domaine ne demande donc aucune modif de code.

---

## Le parcours

21 jours répartis en 3 semaines :

| Semaine | Thème |
|---|---|
| 1 | Décider |
| 2 | Renforcer |
| 3 | Réussir |

Une journée type enchaîne :

1. **Le pas du jour** — le texte de l'exercice
2. **L'image du jour** — la slide associée
3. **Votre réponse** — la saisie de l'utilisateur, enregistrée dans Supabase

> **À COMPLÉTER :** mécanique de déblocage des jours (déblocage par date ? au clic ? à la validation du jour précédent ?) et comportement pour quelqu'un qui arrive en retard ou saute des jours.

---

## La stack

| Brique | Détail |
|---|---|
| Frontend | `index.html` — fichier unique |
| Hébergement | GitHub Pages, compte `acekom84-commits` |
| Base de données | Supabase, projet `bqklygwhqkhdegdbilfm`, région Paris (eu-west-3) |
| Auth | Google OAuth uniquement (le magic link a été retiré) |
| Google Cloud | Console sous compte AcéKom — scopes non sensibles (`email`, `profile`, `openid`) |
| Domaine | `acekom.fr` chez IONOS, sous-domaine `jpa` créé par enregistrement CNAME |
| Emails / newsletter | Brevo — compte propre à JPA |

**Note OAuth :** le compteur « 3 utilisateurs / plafond 100 » de la Google Cloud Console ne s'applique pas ici, puisque l'app ne demande que des scopes non sensibles. Le statut **In Production + External** est correct et suffisant. Pas de vérification de marque à déclencher.

---

## Réglages propres à JPA

### Bouton « Noter la conférence »

Redirection directe vers la page d'avis Google Business de JPA (le système de notation interne a été retiré) :

```
https://search.google.com/local/writereview?placeid=ChIJwTO0jO7DzRIRsmn8l-i8gr4
```

- Libellé : **Noter la conférence**
- Icône : le « G » officiel Google en SVG, aligné à gauche
- **Les couleurs du G ne doivent pas être modifiées** — c'est une obligation des guidelines de marque Google.

### Liens externes suivis

Site web et LinkedIn de JPA — leurs clics sont tracés (voir section Données).

---

## Modifier le contenu

### Ajouter ou remplacer une slide

1. Uploader l'image dans le dossier `slides/` — **nom en minuscules, extension `.jpg`**
2. Dans le fichier, ajouter la ligne suivante juste avant le `}` de fermeture de l'objet du jour concerné :

```js
slide:"slides/nom-du-fichier.jpg"
```

### Modifier un texte de jour

Les textes vivent dans le même objet de données que les slides. Voir les repères ci-dessous pour retrouver le bloc.

---

## Repères Ctrl+F

Le fichier étant unique et long, chercher plutôt que scroller :

| Ce que je cherche | Chaîne à chercher |
|---|---|
| Client Supabase | `createClient` |
| Données des 21 jours | *À COMPLÉTER — nom de la constante* |
| Bouton avis Google | `writereview` |
| Tracking des clics | `trackClic` |
| CSS par semaine | *À COMPLÉTER* |

---

## Données (Supabase)

### Table `progression`

Stocke la progression et les réponses des utilisateurs.

> **À COMPLÉTER :** colonnes exactes.

Une colonne `conference` (type `text`) a été ajoutée par `ALTER TABLE` pour l'attribution par événement.

### Table `clics`

Suivi des clics sur les liens externes de JPA.

| Colonne |
|---|
| `id` |
| `user_id` |
| `bouton` |
| `cree_le` |

- **RLS :** insert autorisé pour les utilisateurs authentifiés, aucune lecture côté app.
- **Côté JS :** fonction `trackClic(bouton)`, encapsulée dans un `try/catch` pour qu'un échec de tracking ne casse jamais la navigation.
- **RGPD :** stocker `user_id` est un choix assumé. L'alternative est le comptage anonyme (`user_id` à `null`).

### Attribution des inscriptions à une conférence

Manuelle, après chaque événement, par plage de dates au format `AAAA-MM-JJ` — **borne haute = le jour suivant** :

```sql
UPDATE progression
SET conference = 'nom-du-label'
WHERE cree_le >= '2026-09-15' AND cree_le < '2026-09-16';
```

Les inscrits tardifs isolés se corrigent un par un dans le Table Editor.

Garder la liste des labels déjà utilisés pour rester cohérente d'un événement à l'autre :

> **À COMPLÉTER :** liste des labels de conférences utilisés à ce jour.

### Sécurité SQL

`SELECT` = lecture seule, sans risque, à utiliser librement pour le suivi. `ALTER`, `UPDATE`, `DELETE` = à manier volontairement et en sachant ce qu'on fait.

---

## Qui fait quoi

- **Florence (AcéKom)** — contenu, textes, slides, édition directe de l'`index.html`, monitoring Supabase.
- **Morgan** — modifications techniques. Il travaille via l'éditeur inline de GitHub ou en télécharge/re-upload : il lui faut des **numéros de ligne exacts ou des chaînes Ctrl+F**, jamais un fichier complet réécrit. Ses instructions lui sont transmises en `.txt`, étape par étape.

---

## En cours

Trois modifications en attente (état : septembre 2026) :

- [ ] **Redirection avis Google** — remplacer le système de notation interne par la redirection décrite plus haut.
- [ ] **Repositionnement de « L'image du jour »** — la placer entre « Le pas du jour » et « Votre réponse ». Un ajustement de `margin` en CSS sera probablement nécessaire.
- [ ] **Tracking des clics** — implémentation de la table `clics` et de `trackClic()` par Morgan.

*(Une fois faites, basculer ces lignes dans un historique en bas de fichier.)*

---

## Ce qui n'a rien à faire ici

Pas de clé `service_role`, pas de mot de passe, pas d'accès Brevo client dans ce repo. La clé `anon` Supabase est publique par nature et présente dans l'`index.html` : c'est normal.
