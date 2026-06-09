# Moniteur — Allocation d'imprimantes

Visualisation interactive du moniteur de synchronisation pour l'allocation de ressources partagées (imprimantes), avec priorité aux processus système sur les processus utilisateur.

## Utilisation

Ouvrir `moniteur_imprimante.html` dans un navigateur. Aucune installation requise.

> Une connexion internet est nécessaire au premier chargement pour les icônes (CDN jsDelivr).

## Paramètres

| Paramètre | Valeur | Description |
|-----------|--------|-------------|
| N | 5 | Nombre total d'imprimantes |
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

## Scénario de test suggéré

1. Demander 3 (système) → Dispo passe à 2
2. Demander 3 (système) → bloqué, besoin résiduel = 1 en file
3. Demander 2 (utilisateur) → bloqué en file
4. Libérer 2 → étape 1 débloque le processus système, étape 3 débloque l'utilisateur
