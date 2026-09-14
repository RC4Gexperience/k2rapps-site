# Déploiement public historique — K2R Apps

> **Archive technique.** Cette procédure documente l’installation initiale et ne doit plus être appliquée. L’état actif de LK2X est décrit dans `EXPLOITATION_LK2X.md`.

## État préparé

Le site est statique. Le dépôt Git doit avoir `site-k2rapps` comme racine. Dans Cloudflare Pages, utiliser :

| Champ | Valeur |
|---|---|
| Framework preset | None |
| Production branch | `main` |
| Build command | `exit 0` |
| Build output directory | `.` |
| Root directory | laisser vide si le dépôt contient directement les fichiers du site |

La redirection permanente de `www.k2rapps.fr` vers `k2rapps.fr` sera créée dans les règles de redirection Cloudflare après le raccordement du domaine. Elle n’est pas stockée dans le projet, car le flux de déploiement Cloudflare actuel traite le site comme un Worker avec ressources statiques.

## Audit DNS public réalisé le 9 septembre 2026

Les serveurs DNS actuellement publiés sont OVH : `dns111.ovh.net` et `ns111.ovh.net`.

### Enregistrements de messagerie à conserver lors de l’import Cloudflare

| Type | Nom | Valeur | Priorité |
|---|---|---|---:|
| MX | `@` | `mx0.mail.ovh.net` | 1 |
| MX | `@` | `mx1.mail.ovh.net` | 5 |
| MX | `@` | `mx2.mail.ovh.net` | 50 |
| MX | `@` | `mx3.mail.ovh.net` | 100 |
| TXT | `@` | `v=spf1 include:mx.ovh.com ~all` | — |

Le DNS public contient aussi le TXT `1|www.k2rapps.fr`, utilisé par l’ancien service Web OVH : le conserver lors de l’import initial. Les éventuels DKIM, DMARC, autoconfig/autodiscover ou enregistrements Zimbra non exposés par les requêtes publiques ci-dessus doivent être vérifiés et conservés depuis la zone DNS OVH avant toute bascule.

## Déploiement et raccordement — actions à effectuer dans les comptes

1. Créer un dépôt GitHub, par exemple `k2rapps-site`, et y pousser le contenu de ce dossier.
2. Dans Cloudflare : **Workers & Pages → Create application → Pages → Import an existing Git repository** ; sélectionner ce dépôt et utiliser exactement les réglages ci-dessus.
3. Attendre la publication réussie sur l’adresse temporaire `*.pages.dev` et vérifier les quatre pages.
4. Dans Cloudflare : **Websites → Add a site** ; ajouter `k2rapps.fr`, choisir le plan Free et laisser Cloudflare importer la zone DNS.
5. Avant de confirmer la bascule des serveurs DNS chez OVH, comparer l’import aux enregistrements de messagerie du tableau ci-dessus et aux éventuels DKIM/DMARC/Zimbra visibles dans OVH. Ne supprimer aucun enregistrement de messagerie.
6. Dans OVH : **Domaines → k2rapps.fr → Serveurs DNS** ; remplacer uniquement les deux serveurs OVH par les deux serveurs Cloudflare fournis pour cette zone. Cette étape ne change pas les MX si la zone importée est complète.
7. Une fois la zone Cloudflare active, dans le projet Pages : **Custom domains → Set up a domain** ; ajouter d’abord `k2rapps.fr`, puis `www.k2rapps.fr`.
8. Lorsque Cloudflare le demande, remplacer uniquement les anciens enregistrements Web de parking : `A @ → 213.186.33.5`, `AAAA @ → 64:ff9b::d5ba:2105` et `A www → 213.186.33.5`. Ne pas modifier les MX, SPF, DKIM, DMARC ou entrées mail.
9. Attendre l’activation HTTPS et vérifier que `www.k2rapps.fr` redirige vers `https://k2rapps.fr`.

## Sous-domaine futur

Ne créer aucun enregistrement pour `app.k2rapps.fr` maintenant. Plus tard, il pourra être relié à un projet Cloudflare distinct, sans modifier le site vitrine ni les enregistrements de messagerie.
