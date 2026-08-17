# Moniteur — Allocation d'imprimantes

Visualisation interactive du moniteur de synchronisation pour l'allocation de ressources partagées (imprimantes), avec priorité aux processus système sur les processus utilisateur.

## Utilisation

Ouvrir `main.html` dans un navigateur. Aucune installation ni connexion internet requise — la page est 100% autonome (aucune dépendance externe, plus de CDN).

## Paramètres

| Paramètre | Valeur par défaut | Description |
|-----------|--------|-------------|
| N | 5 (configurable de 1 à 20) | Nombre total d'imprimantes |
| Dispo | 0 … N | Imprimantes actuellement libres |
| FileSystème | file FIFO | Processus système bloqués + besoin résiduel |
| FileUtilisateur | file FIFO | Processus utilisateur bloqués |

## Procédures

**Demander(K, type)** — un processus réclame K imprimantes.
- Si `Dispo >= K` : allocation immédiate, `Dispo -= K`.
- Si `Dispo < K` et type = système : enfiler le besoin résiduel `(K - Dispo)`, `Dispo = 0`, bloquer.
- Si `Dispo < K` et type = utilisateur : enfiler K, bloquer.

**Libérer(m)** — un processus rend m imprimantes. Quatre étapes dans l'ordre :

| Étape | Condition | Action |
|-------|-----------|--------|
| 0 | toujours | `Dispo += m` |
| 1 | file système non vide **et** `Dispo >= Tête` | débloquer le processus système en tête, répéter |
| 2 | file système non vide **et** `Dispo < Tête` | réserver : `Tête -= Dispo`, `Dispo = 0` |
| 3 | file système vide **et** `Dispo >= Tête` utilisateur | débloquer les processus utilisateur, répéter |

**Garde de sécurité** : `Libérer(m)` est refusé si `m` dépasse le nombre d'imprimantes réellement occupées (`N - Dispo`), ce qui garantit l'invariant `0 ≤ Dispo ≤ N` à tout instant.

## Fonctionnalités

- **Animation pas-à-pas** : chaque étape de `Libérer` s'exécute avec un court délai visible (imprimantes, files et badges se mettent à jour en direct), pas juste le résultat final. Vitesse réglable (Instantanée / Rapide / Normale / Lente).
- **Nombre d'imprimantes configurable** (`N`, de 1 à 20) sans recharger la page.
- **Détecteur de famine (starvation)** : bannière d'alerte si la file utilisateur reste non servie sur plusieurs libérations consécutives à cause de la priorité système — illustre un concept classique d'ordonnancement.
- **Jauge d'occupation** visuelle (imprimantes libres vs occupées).
- **Statistiques cumulées** : demandes totales, allocations immédiates, déblocages après attente, pic d'occupation.
- **Scénario de démonstration** en un clic (rejoue automatiquement le scénario de test ci-dessous).
- **Simulation automatique** : génère des demandes/libérations aléatoires en continu pour observer le système sous charge.
- **Export du journal** en fichier `.txt` (utile pour un rapport de TP).
- **Thème clair/sombre**, mémorisé d'une visite à l'autre.
- **Accessibilité** : journal en zone `aria-live`, libellés `aria-label` sur les imprimantes, raccourci clavier `Entrée` sur les champs K/m.

## Scénario de test suggéré

1. Demander 3 (système) → Dispo passe à 2
2. Demander 3 (système) → bloqué, besoin résiduel = 1 en file
3. Demander 2 (utilisateur) → bloqué en file
4. Libérer 2 → étape 1 débloque le processus système, étape 3 débloque l'utilisateur

Ce scénario peut être rejoué automatiquement via le bouton **▶ Scénario de démo**.
