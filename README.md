
# Application-de-gestion-d-un-magasin-de-fournitures-scolaires-en-Tunisie

# Cahier des charges — Application de gestion d'un magasin de fournitures scolaires en Tunisie

**Version : 7.0 — modèle métier clarifié, application desktop locale (Flutter + SQLite), mono-utilisateur, magasin unique, caisse unique, sans dépôt**

## 1. Présentation du projet

Cette application a pour objectif de gérer simplement et complètement un magasin de fournitures scolaires, papeterie et articles de bureau en Tunisie.

L'application est destinée à des utilisateurs **non développeurs** et peu spécialisés en informatique. L'interface doit donc être simple, claire, rapide et adaptée à une utilisation quotidienne en magasin.

### Mode d’utilisation

L'application fonctionne en **mono-utilisateur** : le propriétaire ouvre directement l'application, sans écran de connexion, identifiant, mot de passe, rôles ou comptes utilisateurs.

### Architecture technique

- application **desktop** développée avec **Flutter** (Windows en priorité) ;
- application **100 % locale** : aucun backend, aucun serveur, aucune API, fonctionnement sans connexion Internet ;
- base de données **SQLite** locale (un seul fichier) ;
- chaque opération (vente, achat, avoir, règlement, clôture, inventaire) est exécutée dans **une transaction unique** : stock, caisse, soldes et audit sont enregistrés ensemble, ou pas du tout ;
- schéma de base de données versionné, avec migrations, pour permettre les mises à jour sans perte de données ;
- lecteur de code-barres utilisé en mode clavier (champ de recherche toujours actif sur l'écran de vente) ;
- impression des tickets sur imprimante thermique 58/80 mm (pilote système ou ESC/POS), à tester dès le début avec l'imprimante réelle du magasin.

### Conventions de montants, quantités et arrondis

- Devise : dinar tunisien (TND), avec trois décimales (millimes).
- Tous les montants monétaires sont stockés en entiers de millimes : 12,500 TND = 12500. Aucun montant métier n'est stocké en `float` ou `double`.
- Les quantités sont stockées en entiers à échelle 1000 : 1 unité = 1000 unités internes ; par exemple, 1,250 unité = 1250. Pour les articles dont `quantiteDecimale = false`, seules les quantités entières sont autorisées.
- Les calculs utilisent une arithmétique entière. Pour un montant positif, tout résultat intermédiaire monétaire non entier en millimes est arrondi au millime le plus proche, les demis étant arrondis au millime supérieur. Le brut d'une ligne est calculé à partir du prix unitaire et de la quantité, puis la remise est calculée et arrondie au millime ; le net de ligne vaut brut moins remise. Le total du document est la somme des lignes nettes. La règle exacte est centralisée dans un service commun et testée.
- Les prix de vente saisis sont TTC. Toute ventilation HT/taxe dépend de la configuration fiscale validée pour le magasin. Les taux et mentions applicables doivent être confirmés avant mise en production.
- Une remise en pourcentage est stockée en points de base (100 = 1,00 % ; 1250 = 12,50 %) ; une remise fixe est stockée en millimes. La ligne conserve le type, la valeur saisie et le montant effectivement appliqué afin que l'historique reste inchangé.

### Modules obligatoires

- gestion de caisse ;
- gestion des ventes ;
- gestion des ventes à crédit ;
- gestion des tickets ;
- gestion des achats ;
- gestion des articles et du stock ;
- gestion des familles ;
- gestion des sous-familles ;
- localisation simple des articles dans les rayons et étagères du magasin ;
- gestion des clients ;
- gestion des fournisseurs ;
- gestion des avoirs de vente ;
- gestion des avoirs d'achat ;
- rapports ;
- tableau de bord ;
- paramètres ;
- inventaire et mouvements de stock.

### Périmètre d'exploitation — un seul magasin, aucun dépôt

L'application gère **un seul établissement commercial**. Elle ne doit pas proposer de module de gestion de plusieurs magasins, de création de magasins supplémentaires, ni de gestion de dépôts, entrepôts ou stocks séparés.

La localisation des articles reste disponible sous une forme simple, à l'intérieur du magasin uniquement : **rayon, étagère ou vitrine**. Cette localisation n'est pas un dépôt et ne crée pas un stock indépendant. Chaque article possède un stock global unique.

### Règle fondamentale — paiements en espèces uniquement

> **Tous les règlements monétaires gérés par l'application sont effectués uniquement en espèces (cash).**

L'application ne doit pas proposer comme moyen de paiement :

- carte bancaire ;
- chèque ;
- virement ;
- paiement en ligne ;
- portefeuille électronique ;
- autre moyen électronique.

Une **vente à crédit** n'est pas un moyen de paiement : elle crée une créance client. Le règlement ultérieur de cette créance est effectué uniquement en espèces.

Un **achat à crédit fournisseur** n'est pas un moyen de paiement : il crée une dette fournisseur. Le règlement ultérieur de cette dette est effectué uniquement en espèces ou compensé avec un avoir fournisseur.

Un **avoir** est un solde commercial et non un moyen de paiement. Lorsqu'un remboursement monétaire est effectué, il se fait uniquement en espèces.

---

# 2. Objectifs

L'application doit permettre au gérant de :

1. connaître les ventes réalisées sur une période ;
2. connaître les encaissements réellement reçus en espèces ;
3. connaître la situation réelle de la caisse ;
4. suivre les ventes à crédit et les créances clients ;
5. suivre les achats et les dettes fournisseurs ;
6. connaître le stock disponible en temps réel ;
7. localiser physiquement un article ;
8. identifier les articles en rupture ou proches du stock minimum ;
9. imprimer rapidement un ticket ;
10. gérer les retours et les avoirs ;
11. suivre les marges commerciales ;
12. consulter des rapports simples sans compétence technique.

---

# 3. Fonctionnement mono-utilisateur

L'application est destinée à **un seul utilisateur : le propriétaire / gérant du magasin**.

- aucun écran de connexion ;
- aucun identifiant ni mot de passe applicatif ;
- aucun module de création ou de gestion de comptes ;
- aucun rôle ni système de permissions par utilisateur ;
- toutes les fonctions métier sont accessibles au propriétaire.

Les actions sensibles (annulation, correction de stock, remboursement et restauration de sauvegarde) doivent demander une confirmation et, si nécessaire, un motif. L'historique conserve automatiquement la date, l'heure, le type d'action et le document concerné, sans demander d'identifier l'utilisateur.

La protection de l'accès à l'application repose sur la sécurité du poste ou du compte système qui l'héberge. Aucun login intégré à l'application n'est prévu.

---

# 4. Menu principal

```text
Tableau de bord
├── Caisse
├── Ventes
├── Tickets
├── Avoirs de vente
├── Achats
├── Avoirs d'achat
├── Articles / Stock
├── Familles
├── Sous-familles
├── Localisation des articles (rayons/étagères)
├── Clients
├── Fournisseurs
├── Inventaire
├── Rapports
└── Paramètres
```

Une recherche globale peut permettre de retrouver un article, client, fournisseur ou document.

---

# 5. Module Gestion de caisse

## 5.1 Ouverture de caisse

Au début de la journée ou de la session :

- date et heure automatiques ;
- montant initial en espèces ;
- remarque facultative.

La plateforme possède **une seule caisse fixe**, intégrée à l’application. Il n’est pas possible de créer, renommer, sélectionner ou supprimer une caisse. Une seule session peut être ouverte à la fois.

**Aucune vente ni opération impliquant un mouvement d'espèces ne peut être validée sans session de caisse ouverte.** Le message affiché est simple : « Ouvrez la caisse pour continuer ». Une session peut rester ouverte sur plusieurs jours ; un rappel est affiché si elle dépasse 24 heures.

## 5.2 Mouvements de caisse

Le système doit enregistrer les **mouvements réels d'espèces** :

- encaissements des ventes comptant ;
- paiements immédiats partiels de ventes à crédit ;
- règlements de dettes clients ;
- remboursements clients en espèces ;
- paiements d'achats comptant ;
- règlements de dettes fournisseurs ;
- remboursements reçus des fournisseurs ;
- dépenses de caisse ;
- autres entrées cash ;
- autres sorties cash.

Une opération sans mouvement réel d'espèces ne doit pas modifier la caisse.

Chaque mouvement contient au minimum :

- numéro ;
- date/heure ;
- type entrée/sortie ;
- montant ;
- motif ;
- référence du document ;

## 5.3 Calcul de caisse

Le solde théorique est calculé à partir du fonds initial et des mouvements nets d'espèces de la session :

```text
Solde théorique
= Fonds initial
+ Parts d'encaissement cash conservées sur les ventes
+ Règlements clients en espèces
+ Autres entrées d'espèces
+ Remboursements reçus des fournisseurs
- Paiements d'achats en espèces
- Règlements fournisseurs en espèces
- Dépenses en espèces
- Remboursements clients en espèces
- Autres sorties d'espèces
```

Les espèces présentées au guichet ne doivent pas être confondues avec le montant net conservé par le magasin : si une monnaie est rendue, le mouvement de vente correspond à la part conservée après monnaie. Le ticket garde séparément les espèces présentées et la monnaie rendue.

Une vente totalement à crédit n'ajoute rien à la caisse. Un achat totalement à crédit ne retire rien de la caisse. L'utilisation ou la compensation d'un avoir, sans échange d'espèces, n'entraîne aucun mouvement de caisse.

## 5.4 Clôture de caisse

À la fin de la session :

1. afficher le solde théorique ;
2. saisir le montant réellement compté en espèces ;
3. calculer l'écart ;
4. demander une remarque si l'écart est différent de zéro ;
5. enregistrer automatiquement la date et l’heure de clôture ;
6. verrouiller la clôture ;
7. imprimer ou exporter l'état de caisse.

```text
Écart = Espèces réellement comptées - Solde théorique
```

Une clôture validée ne doit pas être supprimée et la session clôturée n'est **jamais modifiée**. Toute correction doit être tracée.

Si un document d'une session déjà clôturée est annulé ensuite (vente, avoir, achat, règlement), le mouvement de caisse inverse est enregistré **dans la session actuellement ouverte**, avec la référence du document d'origine.

---

# 6. Module Gestion des ventes

## 6.1 Vente comptant

Le propriétaire doit pouvoir scanner/rechercher un article, l'ajouter au panier, modifier la quantité, appliquer une remise autorisée, consulter le total, saisir les espèces présentées, visualiser la monnaie à rendre, valider la vente et imprimer le ticket.

Règles :

- Une vente validée exige une session de caisse ouverte, même si la vente est intégralement à crédit. Sans session ouverte, afficher : « Ouvrez la caisse pour continuer ».
- Le client est facultatif pour une vente réglée immédiatement sans utilisation d'un avoir client. Il devient obligatoire si la vente crée une créance ou utilise un avoir/solde commercial client.
- Une remise de ligne peut être exprimée en pourcentage ou en montant fixe. Le type et la valeur de la remise sont enregistrés. Le montant de remise ne peut pas dépasser le montant brut de la ligne. La remise maximale de l'article est contrôlée sur l'équivalent en pourcentage du montant brut ; toute dérogation exige une confirmation explicite et une trace d'audit.
- Une vente peut être mise en attente puis reprise. Une vente en attente n'affecte ni stock, ni caisse, ni créances, ni avoirs.
- Le ticket est le document de vente remis au client ; l'application ne produit pas de facture de vente.
- La validation est atomique : lignes, numérotation, stock, éventuel mouvement de caisse et audit sont enregistrés ensemble, ou aucun de ces effets n'est enregistré.

### Espèces présentées, monnaie et caisse

Distinguer obligatoirement :

- **Espèces présentées** : billets/pièces remis par le client ;
- **Monnaie rendue** : espèces restituées au client ;
- **Part réglée en espèces** : montant réellement conservé pour régler la vente, après déduction de la monnaie rendue.

Le mouvement de caisse de la vente enregistre la part réglée en espèces, et non les espèces présentées si une monnaie doit être rendue. Le ticket affiche les trois valeurs. La monnaie ne peut être négative.

## 6.2 Vente à crédit ou partiellement réglée

Une vente qui laisse un reste dû exige un client identifié. Le système doit gérer une vente totalement à crédit, partiellement réglée en espèces, réglée en espèces, ou dont une partie est couverte par un avoir client valide.

Le type de vente est une classification du règlement, pas un moyen de paiement : `COMPTANT` signifie que la vente est soldée à la validation (en espèces et/ou par avoir client), `CREDIT` signifie qu'un reste dû est créé sans paiement cash immédiat, et `MIXTE` signifie qu'une partie est réglée en espèces et qu'un reste dû subsiste. Les montants cash, les avoirs utilisés et le reste dû restent toujours affichés séparément.

Au moment de la validation :

```text
Reste à régler à la validation
= Total net de la vente
- Part réglée en espèces
- Avoir client appliqué à cette vente
```

Le solde dû courant du client est ensuite diminué par les règlements clients affectés à cette vente et par les compensations d'avoirs autorisées. Le solde dû est calculé à partir des documents et affectations valides ; il ne doit pas être modifié manuellement.

Exemple sans avoir :

```text
Total net de la vente        : 120,000 TND
Part réglée en espèces       :  50,000 TND
Reste dû à la validation     :  70,000 TND
```

Effets :

```text
Stock       → - quantité vendue
Caisse      → + 50,000 TND
Créance     → + 70,000 TND
```

La vente conserve les lignes, la quantité, le prix et la remise appliqués, le coût moyen au moment de la vente, le total net, la part réglée en espèces, les opérations d'avoirs liées, l'échéance facultative et le statut.

## 6.3 Règlement d'une dette client

Depuis la fiche client, le propriétaire sélectionne une ou plusieurs ventes restant dues, saisit le montant reçu en espèces et confirme le règlement. L'interface peut proposer l'affectation automatique aux dettes les plus anciennes, tout en permettant de vérifier les affectations avant validation.

- Un règlement client est toujours en espèces et exige une session de caisse ouverte.
- Le montant du règlement ne peut pas dépasser le solde dû affectable du client.
- Chaque règlement est relié à une ou plusieurs affectations qui indiquent la vente concernée et le montant affecté. La somme des affectations doit être égale au montant du règlement.
- Un règlement crée un mouvement de caisse positif, diminue les soldes des ventes affectées et produit une référence/reçu.
- Une annulation de règlement conserve le document original et crée les opérations inverses traçables ; elle ne supprime pas l'historique.

## 6.4 Annulation d'une vente

Une vente validée n'est jamais supprimée physiquement. L'annulation demande une confirmation et un motif, conserve le numéro et inverse les effets sur le stock, la créance et uniquement les espèces réellement encaissées.

Une vente ayant déjà reçu des règlements, des compensations ou des avoirs liés ne peut être annulée qu'après inversion ou annulation des documents dépendants, dans un ordre autorisé. La totalité de l'opération d'annulation est atomique.

Si le mouvement de caisse d'origine appartient à une session clôturée, la session clôturée reste inchangée et le mouvement inverse est enregistré dans la session actuellement ouverte. Une opération qui nécessite un mouvement inverse d'espèces est bloquée tant qu'aucune session n'est ouverte.

## 6.5 Retours

Un retour physique est normalement traité par un avoir de vente. Chaque ligne retournée indique l'état constaté :

```text
Intact / revendable → stock disponible augmenté
Endommagé           → stock endommagé/bloqué augmenté
Défectueux          → stock bloqué, puis mise au rebut selon la procédure prévue
```

La quantité retournée cumulée ne peut pas dépasser la quantité vendue sur la ligne d'origine, déduction faite des retours non annulés précédents.

# 7. Module Avoirs de vente

Un avoir de vente représente un retour client ou une correction financière justifiée. Il est rattaché à la vente d'origine lorsque celle-ci est connue. Une correction purement financière sans retour physique peut être créée sans ligne article, avec motif obligatoire.

## 7.1 Contenu et création

Chaque avoir contient un numéro unique, une date/heure, la vente d'origine si connue, le client si nécessaire, ses lignes éventuelles, le montant total, le motif et le statut. Une ligne de retour référence la ligne de vente d'origine lorsque possible et conserve le prix/remise de cette vente ; le prix actuel de l'article ne s'applique jamais rétroactivement.

## 7.2 Effet sur le stock

Pour un retour physique, le mouvement de stock est créé dans la même transaction que l'avoir :

```text
Article revendable → stock disponible + quantité retournée
Article endommagé  → stock endommagé/bloqué + quantité retournée
```

Les retours cumulés non annulés ne dépassent pas la quantité de la ligne d'origine. Un avoir de correction financière sans retour physique ne crée aucun mouvement de stock.

## 7.3 Traitements financiers

Un avoir peut être traité en totalité ou en plusieurs opérations cumulables :

- conservé comme solde client disponible ;
- utilisé sur une future vente du même client ;
- compensé avec une créance existante du même client ;
- remboursé en espèces.

Chaque utilisation, compensation ou remboursement est enregistré comme une opération distincte liée à l'avoir. Le cumul des opérations valides ne peut jamais dépasser le montant de l'avoir. Une opération qui implique un remboursement en espèces exige une session de caisse ouverte et crée une sortie de caisse égale au montant réellement remboursé. Une conservation, une utilisation d'avoir ou une compensation sans espèces ne modifie pas la caisse.

Si la vente d'origine était anonyme et qu'aucun client n'est affecté à l'avoir, seul le remboursement en espèces est autorisé. Un avoir conservé, utilisé sur une vente future ou compensé exige un client identifié.

## 7.4 Utilisation et solde de l'avoir

Lorsqu'un avoir est utilisé sur une nouvelle vente, cette vente doit être liée au même client. Le montant appliqué ne peut pas dépasser le solde disponible de l'avoir ni le montant restant à régler sur la vente.

```text
Solde disponible de l'avoir
= Montant total de l'avoir
- Utilisations sur ventes
- Compensations de créances
- Remboursements en espèces
```

Le solde restant peut demeurer disponible pour une utilisation ultérieure ; il n'a pas de date d'expiration. Les montants utilisés, compensés et remboursés sont calculés à partir des opérations enregistrées, et non modifiés manuellement.

## 7.5 Annulation d'un avoir

Un avoir validé n'est jamais supprimé. Son annulation exige une confirmation et un motif, conserve le numéro, inverse les mouvements de stock et les opérations financières associées et inscrit un audit. Si l'avoir a déjà été utilisé, compensé ou remboursé, les opérations dépendantes doivent d'abord être annulées ou inversées dans un ordre autorisé. Toute sortie de caisse inverse est passée dans la session actuellement ouverte ; une session clôturée n'est jamais modifiée.

# 8. Module Gestion des tickets

Le ticket est généré automatiquement après validation d'une vente. Il porte le numéro de la vente et constitue le seul document remis au client : il n'existe pas de facture de vente dans l'application.

## Contenu du ticket

- logo facultatif, nom commercial, adresse et téléphone ;
- informations fiscales, selon la configuration validée ;
- numéro du ticket (identique au numéro de vente) et date/heure ;
- client si connu ;
- articles, quantités, prix unitaires, remises et totaux ;
- total TTC ;
- montant d'avoir utilisé sur la vente, le cas échéant ;
- espèces présentées par le client ;
- part réglée en espèces et monnaie rendue ;
- mention comptant/crédit et reste dû éventuel.

Un avoir validé doit disposer d'un document d'avoir imprimable ou exportable, avec son numéro, son origine si connue, les articles retournés ou la justification financière, son montant, le mode et le montant du traitement financier, et le motif.

## Impression

Compatibilité avec imprimante thermique :

- 58 mm ;
- 80 mm.

Possibilité de :

- prévisualiser ;
- imprimer ;
- réimprimer le document existant sans créer une nouvelle opération ;
- exporter en PDF si nécessaire.

Une réimpression ne doit pas créer une nouvelle vente.

---

# 9. Module Gestion des achats

## 9.1 Création d'un achat

L'achat contient un fournisseur, la date/heure, la référence de facture papier facultative, les lignes, quantités, prix d'achat, remises, taxes si applicables, le total net et une remarque éventuelle. L'application enregistre la référence de la facture fournisseur mais ne génère pas cette facture.

## 9.2 Achat payé, partiellement payé ou à crédit

Le système accepte un achat réglé entièrement en espèces, partiellement en espèces, entièrement à crédit fournisseur, ou partiellement réglé au moyen d'un avoir fournisseur existant.

```text
Dette créée à la validation
= Total net de l'achat
- Part réglée en espèces
- Avoir fournisseur appliqué à cet achat
```

L'application doit empêcher les montants négatifs et l'utilisation d'un avoir supérieur à son solde disponible. L'utilisation d'un avoir fournisseur ne crée pas de mouvement de caisse. Seule la part effectivement payée en espèces diminue la caisse.

À la validation d'un achat, le stock disponible augmente des quantités reçues, les coûts et prix utilisés sont conservés sur les lignes d'achat et le coût moyen pondéré courant est mis à jour.

## 9.3 Règlement d'une dette fournisseur

Un règlement fournisseur est toujours effectué en espèces et exige une session de caisse ouverte. Il est relié au fournisseur et à une ou plusieurs affectations vers les achats qui restent dus. Le montant ne peut pas dépasser la dette affectable ; la somme des affectations est égale au montant du règlement. Le règlement diminue la dette et la caisse, et une référence/reçu est conservé.

Une annulation de règlement n'efface pas le document d'origine : elle inverse les affectations et crée le mouvement de caisse inverse traçable, sans modifier une session déjà clôturée.

## 9.4 Effet sur le stock et le coût moyen

Après validation, les quantités achetées sont ajoutées au stock disponible. Le prix d'achat net utilisé et le coût moyen antérieur sont historisés sur les documents nécessaires à l'audit.

## 9.5 Annulation d'un achat

Un achat validé n'est jamais supprimé. L'annulation exige un motif et une confirmation. Elle est bloquée si le stock disponible ne permet pas de retirer les quantités concernées ou si des documents dépendants ne sont pas préalablement inversés. L'annulation inverse le stock et les effets financiers réellement constatés, dont uniquement le paiement cash effectivement réalisé. Le coût moyen courant est recalculé selon l'historique valide, sans modifier le coût enregistré sur les anciennes lignes de vente. Les règlements ou avoirs dépendants doivent être inversés avant l'annulation.

# 10. Module Avoirs d'achat

Un avoir d'achat représente un retour de marchandises ou une correction financière accordée par le fournisseur. Il est rattaché à l'achat d'origine lorsque possible ; une correction purement financière peut être saisie sans lignes physiques, avec motif obligatoire.

## 10.1 Contenu

L'avoir contient son numéro, le fournisseur, l'achat d'origine si connu, la date/heure, les lignes éventuelles, le prix d'achat historique, les remises, les taxes applicables, le montant total, le motif et le statut.

## 10.2 Effet sur le stock

Pour un retour physique, les quantités sont retirées du stock disponible dans la même transaction que l'avoir. Les quantités cumulées retournées ne dépassent pas les quantités réceptionnées sur les lignes d'achat concernées ; elles ne dépassent pas non plus les quantités physiquement disponibles à retirer. Une correction financière sans retour physique ne modifie pas le stock.

## 10.3 Traitements financiers

Un avoir fournisseur peut être imputé sur une dette existante, utilisé sur un prochain achat, conservé comme solde disponible ou remboursé en espèces. Plusieurs traitements peuvent être combinés, au moyen d'opérations distinctes, dans la limite du montant total de l'avoir.

- Une imputation sur dette diminue la dette fournisseur sans modifier la caisse.
- L'utilisation sur un prochain achat diminue le montant à régler de cet achat sans mouvement de caisse.
- Un remboursement reçu en espèces augmente la caisse et exige une session ouverte.

```text
Solde disponible de l'avoir fournisseur
= Montant de l'avoir
- Montants imputés sur dettes
- Montants utilisés sur achats
- Remboursements reçus en espèces
```

## 10.4 Annulation d'un avoir d'achat

Un avoir validé n'est jamais supprimé. L'annulation exige une confirmation et un motif, conserve le numéro et inverse ses effets sur le stock et les soldes. Les utilisations, imputations et remboursements dépendants doivent d'abord être annulés ou inversés. Toute entrée de caisse inverse est enregistrée dans la session ouverte, sans modifier une session clôturée.

# 11. Module Gestion des articles et du stock

Chaque article comprend un identifiant interne, un code généré automatiquement, une désignation, une unité, un prix de vente TTC et un statut. Les champs suivants sont facultatifs à la création rapide et peuvent être complétés plus tard : code-barres, description, famille, sous-famille, marque, prix d'achat, emplacement, stock minimum, catégorie fiscale et photo. Le stock initial est nul par défaut ; toute quantité initiale doit être enregistrée par une opération d'ouverture de stock/inventaire traçable, jamais saisie silencieusement.

L'article contient aussi :

- quantité décimale autorisée (non par défaut) ;
- dernier prix d'achat et coût moyen pondéré courant ;
- remise maximale autorisée exprimée en pourcentage ;
- stock disponible, stock endommagé/bloqué et stock minimum ;
- emplacement principal facultatif dans le magasin unique.

## 11.1 Exemples d'articles

```text
Stylo bleu BIC
Cahier 96 pages
Cahier 200 pages
Ramette papier A4
Crayon HB
Gomme blanche
Taille-crayon
Cartable
Sac à dos
Feutres
Peinture
Colle
Règle 30 cm
Calculatrice
```

## 11.2 Recherche

Recherche par code-barres, code article, désignation, famille, sous-famille, marque ou emplacement. Un scan doit pouvoir ajouter directement l'article au panier de vente. Si l'article n'existe pas, un bouton permet de créer rapidement une fiche minimale.

## 11.3 Prix, remises et marge

```text
Marge unitaire estimée = Prix de vente net - Coût moyen pondéré
Taux de marge estimé = (Marge unitaire / Coût moyen pondéré) × 100
```

Si le coût moyen est nul, le taux de marge n'est pas calculé et l'interface affiche une information appropriée. Chaque ligne de vente conserve le prix, la remise appliquée et le coût moyen pondéré au moment de la vente ; modifier le prix courant ne change aucun ancien document.

### Valorisation du stock — coût moyen pondéré

```text
Nouveau coût moyen pondéré
= arrondi au millime de :
  (Stock disponible actuel × Coût moyen actuel
   + Quantité reçue × Prix d'achat net)
  / (Stock disponible actuel + Quantité reçue)
```

Si le stock disponible avant réception est nul, le nouveau coût moyen prend le prix d'achat net de la réception. Les calculs utilisent les quantités à échelle 1000 et les montants en millimes, sans `double` pour les valeurs métier.

Règles :

- le coût moyen est recalculé à chaque achat validé, correction ou annulation qui modifie la valorisation ;
- les ventes, retours clients et retours fournisseurs ne recalculent pas le coût moyen courant ;
- les lignes de vente conservent le coût moyen historique, même lorsqu'un achat ultérieur est annulé ;
- la valeur du stock disponible = quantité disponible × coût moyen ; le stock endommagé/bloqué est suivi séparément ;
- le traitement des taxes incluses dans le coût d'achat est appliqué seulement après validation du régime fiscal réel.

## 11.4 Mouvements de stock

Types minimaux : entrée par achat, sortie par vente, retour client revendable, retour client endommagé/bloqué, retour fournisseur, correction positive/négative, inventaire, mise au rebut et remise en stock disponible depuis le stock bloqué.

Chaque mouvement contient l'article, le type, le compartiment concerné (disponible ou endommagé/bloqué), la quantité à échelle 1000, le stock avant et après, la date/heure, la référence source et un motif si nécessaire. Le mouvement et le solde courant de l'article sont modifiés dans une transaction unique.

## 11.5 Inventaire

Le système permet de créer une session, d'afficher le stock théorique par article et compartiment, de saisir le stock compté, de calculer l'écart, de valider la session et de produire les mouvements de correction correspondants. Une session validée est conservée dans l'historique et n'est pas réécrite ; une correction ultérieure crée un nouveau mouvement traçable.

## 11.6 Alertes et règles de stock

```text
Stock disponible = 0              → Rupture
Stock disponible <= stock minimum → Réapprovisionnement conseillé
```

Une vente qui ferait passer le stock disponible sous zéro est bloquée. Les quantités sont positives dans chaque opération de mouvement ; le sens entrée/sortie est porté par le type de mouvement. Les corrections passent par l'inventaire ou une correction explicitement motivée.

# 12. Module Familles

Permet de classer les produits par grandes catégories.

Exemples :

```text
Écriture
Papier
Cahiers
Dessin & Arts plastiques
Fournitures de bureau
Classement
Scolaire
Cartables & Sacs
Informatique
Accessoires
```

Fonctions :

- ajouter ;
- modifier ;
- désactiver ;
- rechercher ;
- afficher le nombre d'articles ;
- consulter les ventes par famille.

---

# 13. Module Sous-familles

Chaque sous-famille appartient à une famille.

Exemple :

```text
Famille : Écriture
  ├── Stylos
  ├── Crayons
  ├── Feutres
  ├── Surligneurs
  └── Marqueurs

Famille : Cahiers
  ├── Cahiers scolaires
  ├── Blocs-notes
  ├── Carnets
  └── Registres
```

Une sous-famille utilisée par des articles ne doit pas être supprimée. Elle doit être désactivée ou les articles doivent être réaffectés.

---

# 14. Localisation des articles dans le magasin

Cette fonction sert uniquement à retrouver rapidement un article dans le **magasin unique**. Il ne s'agit pas d'une gestion de dépôt ou d'entrepôt.

Exemples de localisations autorisées :

```text
Rayon A - Écriture
Rayon B - Cahiers
Rayon C - Papier
Rayon D - Arts plastiques
Vitrine 1
Étagère 2 - Fournitures de bureau
```

Chaque article peut avoir une seule localisation principale. Les informations de localisation peuvent comprendre :

- code facultatif ;
- nom court ;
- rayon ;
- étagère ;
- description facultative.

Règles :

- aucun dépôt, entrepôt ou réserve n'est créé ou géré ;
- aucun stock séparé par magasin ou dépôt ;
- aucun transfert de stock entre dépôts ;
- la localisation ne modifie pas la quantité globale en stock ;
- le stock est suivi au niveau de l'article, dans le magasin unique.

---

# 15. Module Gestion des clients

## 15.1 Fiche client

- code client généré ;
- nom et prénom / raison sociale (obligatoire) ;
- téléphone et adresse (facultatifs) ;
- identifiant fiscal facultatif ;
- autorisation de crédit ;
- plafond de crédit facultatif ;
- solde dû calculé à partir des ventes, règlements et compensations validés ;
- avoir disponible calculé à partir des opérations d’avoirs valides ;
- statut et remarque.

Les soldes et avoirs affichés ne sont pas des champs librement modifiables. Toute correction passe par un document ou une opération traçable.

## 15.2 Historique client

Afficher chronologiquement les ventes comptant et à crédit, règlements en espèces, retours, avoirs créés/utilisés/compensés/remboursés, solde courant et échéances éventuelles.

## 15.3 Règles de crédit

Le système permet d'autoriser/interdire le crédit, de définir un plafond, d'alerter en cas de dépassement, de bloquer par défaut une vente qui dépasserait le plafond et, si le propriétaire confirme explicitement, d'enregistrer la dérogation dans le journal d'audit. Les dettes peuvent être affichées par ancienneté si des échéances sont renseignées.

# 16. Module Gestion des fournisseurs

## 16.1 Fiche fournisseur

- code fournisseur généré ;
- raison sociale / nom (obligatoire) ;
- contact, téléphone, adresse, identifiant fiscal et email (facultatifs selon le besoin) ;
- dette calculée à partir des achats, règlements et imputations validés ;
- avoir disponible calculé à partir des opérations d'avoirs valides ;
- statut et remarque.

Les soldes affichés sont dérivés des documents et affectations valides ; ils ne sont pas modifiés manuellement.

## 16.2 Historique fournisseur

Afficher les achats, retours, avoirs d'achat, règlements en espèces, remboursements reçus en espèces, dette courante, avoirs disponibles et historique chronologique.

# 17. Tableau de bord

Le tableau de bord doit être compréhensible rapidement.

## 17.1 Indicateurs principaux

Afficher notamment :

- chiffre d'affaires ;
- nombre de ventes ;
- ventes comptant ;
- ventes à crédit ;
- encaissements clients ;
- avoirs de vente ;
- remboursements clients ;
- total achats ;
- avoirs d'achat ;
- règlements fournisseurs ;
- remboursements fournisseurs ;
- dépenses caisse ;
- solde théorique de caisse ;
- créances clients ;
- dettes fournisseurs ;
- avoirs clients disponibles ;
- avoirs fournisseurs disponibles ;
- valeur du stock ;
- marge commerciale estimée ;
- articles en rupture ;
- articles sous stock minimum.

## 17.2 Filtres

- aujourd'hui ;
- hier ;
- semaine ;
- mois ;
- année ;
- période personnalisée.

## 17.3 Graphiques

- ventes par jour ;
- ventes par semaine ;
- ventes par mois ;
- achats par période ;
- top articles ;
- ventes par famille ;
- créances clients ;
- évolution de la marge.

---

# 18. Rapports

Tous les rapports doivent pouvoir être filtrés par période et, selon le cas, article, famille, client ou fournisseur.

## 18.1 Rapports ventes

- ventes par période ;
- détail des tickets ;
- ventes comptant ;
- ventes crédit ;
- paiements immédiats cash sur ventes crédit ;
- règlements de crédits ;
- retours ;
- avoirs de vente ;
- remboursements clients ;
- remises ;
- top articles ;
- marge estimée.

## 18.2 Rapports caisse

- journal de caisse ;
- ouvertures ;
- clôtures ;
- encaissements ;
- sorties ;
- dépenses ;
- autres entrées/sorties ;
- règlements clients ;
- remboursements clients ;
- règlements fournisseurs ;
- remboursements fournisseurs ;
- écarts de caisse.

## 18.3 Rapports stock

- stock actuel ;
- stock disponible ;
- stock endommagé/bloqué ;
- articles en rupture ;
- articles sous minimum ;
- mouvements ;
- inventaires ;
- valorisation du stock ;
- articles par rayon/étagère (localisation simple dans le magasin).

## 18.4 Rapports achats

- achats par période ;
- achats par fournisseur ;
- détail des achats ;
- retours fournisseurs ;
- avoirs d'achat ;
- remboursements fournisseurs ;
- dettes fournisseurs.

## 18.5 Rapports clients

- clients débiteurs ;
- créances totales ;
- règlements ;
- avoirs disponibles ;
- historique d'un client.

## 18.6 Rapports fournisseurs

- fournisseurs débiteurs ;
- dettes totales ;
- règlements ;
- avoirs disponibles ;
- historique d'un fournisseur.

## 18.7 Export

Prévoir :

- PDF ;
- Excel/CSV selon besoin.

---

# 19. Paramètres de la plateforme — configuration très simple

L’application doit fonctionner avec **presque aucune configuration**. Les valeurs techniques et les règles métier sont prédéfinies. Le propriétaire ne configure que les informations réellement nécessaires.

## 19.1 Informations à saisir au premier lancement

Seuls ces champs sont nécessaires pour démarrer :

- nom commercial du magasin ;
- téléphone ou adresse, facultatifs ;
- logo, facultatif.

Le matricule fiscal et les informations fiscales sont renseignés uniquement s’ils sont applicables au magasin.

## 19.2 Paramètres simples, accessibles plus tard

Le menu `Paramètres` doit rester limité à ces rubriques :

```text
PARAMÈTRES

[ Informations du magasin ]
[ Ticket et impression ]
[ Fiscalité (si nécessaire) ]
[ Sauvegarde ]
```

Il n’existe **aucun écran de configuration de caisse** : la caisse est unique et fixe.

### Ticket et impression

- largeur du ticket : 80 mm par défaut ; choix 58 mm si nécessaire ;
- imprimante : choix facultatif ;
- impression automatique : activée par défaut si une imprimante est configurée ;
- message de pied de ticket : facultatif.

Les numéros de ventes, tickets, achats, avoirs et règlements sont générés automatiquement.

### Fiscalité

Les informations fiscales ne doivent pas bloquer le démarrage. Elles sont saisies uniquement si elles s’appliquent au régime du magasin :

- identifiant fiscal ;
- catégories et taux applicables ;
- mentions à imprimer sur les documents.

Elles doivent être validées selon le régime réel du magasin. Les taux ne sont pas demandés à chaque vente : ils proviennent de catégories fiscales configurées. Chaque article peut référencer une catégorie fiscale ; les lignes de vente conservent une copie des informations fiscales utilisées au moment de la validation afin qu'une modification ultérieure n'altère pas les anciens tickets.

Par défaut, tous les prix de vente sont **TTC** et le ticket affiche le total TTC. Si la ventilation HT/taxe est requise, les montants TTC sont regroupés par catégorie/taux, la composante fiscale est calculée à partir du TTC et arrondie une seule fois par catégorie ; le HT est le TTC moins la taxe calculée. Les taux, mentions obligatoires et règles exactes doivent être confirmés pour le régime réel du magasin. Le traitement des taxes sur les achats et dans le coût moyen pondéré doit être validé avec le comptable avant la mise en production.

### Sauvegarde

- sauvegarde automatique activée par défaut ;
- sauvegarde locale automatique (à la clôture de caisse et à la fermeture de l'application) ;
- copie vers un dossier externe au choix (clé USB, disque externe ou dossier synchronisé) : une alerte est affichée tant qu'aucune copie externe n'est configurée ou si elle est ancienne ;
- bouton `Sauvegarder maintenant` ;
- bouton `Restaurer une sauvegarde`, avec confirmation.

Aucune fréquence technique ni paramètre de base de données ne doit être demandé à l’utilisateur ordinaire.

## 19.3 Valeurs et règles fixes

```text
Pays                    = Tunisie
Devise                  = TND
Langue                  = Français
Fuseau horaire          = Africa/Tunis
Paiement                = Espèces uniquement
Nombre de magasins      = 1
Nombre de caisses       = 1
Dépôt / entrepôt        = Aucun
Session de caisse       = Une seule ouverte à la fois
Numérotation documents  = Automatique
Stock et totaux         = Calculés automatiquement
Sauvegarde automatique  = Activée
Vente avec stock négatif= Interdite
Vente sans caisse ouverte= Interdite
Valorisation du stock   = Coût moyen pondéré
Précision des montants  = millimes stockés en entiers
Précision des quantités = milli-unités (échelle 1000) stockées en entiers
Facture de vente        = Aucune (ticket uniquement)
Architecture            = Flutter desktop, SQLite local, sans backend
```

Ces éléments ne sont pas présentés comme paramètres modifiables.

## 19.4 Assistant de démarrage

L’installation ne doit pas nécessiter un assistant complexe. Au premier lancement, un petit écran demande uniquement :

```text
1. Nom du magasin (obligatoire)
2. Téléphone / adresse (facultatifs)
3. Configurer l’imprimante maintenant ou plus tard

[ Commencer à utiliser l’application ]
```

L’utilisateur peut ignorer les champs facultatifs et commencer immédiatement.

## 19.5 Configuration technique invisible

Les paramètres de base de données, numérotation, caisse, stock, crédit, avoirs et calculs sont gérés par l’application et ne sont pas exposés dans l’interface métier.

---

# 20. Règles métier obligatoires

## Règle 1 — Espèces uniquement

Le seul moyen de règlement monétaire est l'espèce. Les crédits et les avoirs sont des soldes commerciaux, pas des moyens de paiement. Aucun paiement par carte, chèque, virement ou moyen électronique n'est prévu.

## Règle 2 — Vente et caisse

Toute vente validée exige une session de caisse ouverte. Le stock diminue à la validation. Le mouvement de caisse est égal à la part de la vente réellement réglée et conservée en espèces, après monnaie rendue. Les espèces présentées et la monnaie sont conservées séparément sur le ticket.

## Règle 3 — Calcul de la dette client

À la validation : `reste dû initial = total net - part cash affectée à la vente - avoir client appliqué`. Le solde courant est ensuite diminué par les règlements affectés et les compensations valides. Aucun solde dû n'est corrigé directement dans une fiche.

## Règle 4 — Achat et dette fournisseur

À la validation : `dette créée = total net - part payée cash - avoir fournisseur appliqué à cet achat`. Le stock augmente de la quantité reçue. Seul le montant effectivement payé en espèces diminue la caisse.

## Règle 5 — Affectation des règlements

Un règlement client ou fournisseur peut être affecté à une ou plusieurs dettes. Chaque affectation référence le document concerné. La somme des affectations égale le montant du règlement et aucune affectation ne peut dépasser le solde dû du document. Les règlements dépassant le solde total affectable sont bloqués.

## Règle 6 — Caisse

Toute opération entraînant une entrée ou une sortie d'espèces exige une session ouverte. Les opérations sans échange réel d'espèces n'affectent pas la caisse. Une session clôturée conserve son solde théorique, les espèces comptées, l'écart et l'heure de clôture ; ces valeurs ne sont jamais modifiées.

## Règle 7 — Annulation après clôture

Toute annulation postérieure à une clôture crée, si nécessaire, un mouvement inverse dans la session actuellement ouverte. La session d'origine reste inchangée. Si aucun mouvement inverse cash n'est requis, aucune opération de caisse n'est créée.

## Règle 8 — Avoir de vente

Un avoir peut être partiel ou total et comprendre un retour physique et/ou une correction financière motivée. Les utilisations sur ventes, compensations de créances et remboursements cash sont enregistrés comme opérations distinctes. Leur cumul ne dépasse jamais le montant de l'avoir. Le solde non consommé reste disponible pour le client.

## Règle 9 — Avoir d'achat

Un avoir fournisseur peut être imputé sur une dette, appliqué à un nouvel achat, conservé comme solde ou remboursé en espèces. Les opérations sont distinctes et leur cumul ne dépasse pas le montant de l'avoir. Seul un remboursement en espèces reçu crée une entrée de caisse.

## Règle 10 — Client obligatoire pour utiliser un avoir client

Une vente comptant sans crédit peut rester anonyme. Une vente qui crée une créance ou utilise un avoir client nécessite un client. Un avoir issu d'une vente anonyme, sans client associé, ne peut être remboursé qu'en espèces ; il doit être affecté à un client pour être conservé, utilisé ou compensé.

## Règle 11 — Aucun impact artificiel sur la caisse

Un achat à crédit, une vente à crédit sans versement immédiat, une imputation ou une utilisation d'avoir sans espèces n'entraîne aucun mouvement de caisse.

## Règle 12 — Documents historiques immuables

Le changement d'un prix ou d'une fiche article n'altère pas les documents existants. Les lignes conservent le prix, la remise, le total, les informations fiscales utiles et le coût moyen historique utilisé. Les documents validés ne sont jamais supprimés physiquement.

## Règle 13 — Numérotation

Chaque type de document possède un numéro unique par année civile, attribué atomiquement dans la base. Exemple : `VTE-2026-000001`, `AVV-2026-000001`, `ACH-2026-000001`, `AVA-2026-000001`, `REGC-2026-000001` et `REGF-2026-000001`. L'unicité est garantie par une contrainte de base de données. Une annulation conserve le numéro d'origine.

## Règle 14 — Remises

Une remise de ligne indique le type (pourcentage ou montant fixe), la valeur saisie et le montant effectivement appliqué. Elle ne peut pas être négative ni dépasser le montant brut de la ligne. Le plafond article est contrôlé en équivalent pourcentage ; toute dérogation est confirmée et auditée.

## Règle 15 — Plafond de crédit

Si un plafond est activé, l'application vérifie le solde dû actuel du client et l'impact de la vente avant validation. Un dépassement est bloqué par défaut ; une confirmation explicite peut être autorisée et doit être tracée.

## Règle 16 — Quantités retournées et disponibles

La quantité cumulée retournée sur une ligne de vente ou d'achat ne peut pas dépasser la quantité d'origine, tous documents non annulés confondus. Un retour physique fournisseur doit en plus être couvert par le stock disponible à retirer.

## Règle 17 — Traçabilité

Les opérations sensibles gardent la date/heure, l'action, le type et la référence du document, l'ancienne et la nouvelle valeur si nécessaire, ainsi que le motif. L'application est mono-utilisateur et ne demande aucun identifiant utilisateur.

## Règle 18 — Transactions atomiques

La validation, l'annulation, les règlements, les opérations d'avoirs, la clôture et l'inventaire s'exécutent dans une transaction SQLite unique. En cas d'erreur, aucun effet partiel ne subsiste sur les documents, le stock, la caisse, les soldes ou le journal d'audit.

## Règle 19 — Quantités et montants exacts

Les montants monétaires sont des entiers en millimes. Les quantités utilisent des entiers à échelle 1000 ; les articles non décimaux n'acceptent que des quantités entières. Les conversions, remises et arrondis sont centralisés dans des services métier testés.

## Règle 20 — Soldes calculés

Les créances, dettes et soldes d'avoirs sont dérivés des documents validés et des affectations/opérations non annulées. Une éventuelle valeur de cache est mise à jour dans la même transaction que le document source et doit pouvoir être reconstruite depuis l'historique.

## Règle 21 — Cohérence du coût moyen

Chaque ligne de vente conserve le coût moyen pondéré utilisé lors de sa validation. Un achat annulé ou une correction de stock peut recalculer le coût moyen courant à partir de l'historique valide, mais ne réécrit jamais les coûts historiques des ventes.

# 21. Parcours utilisateur principaux

## 21.1 Ouverture journée

```text
Lancement de l’application
 ↓
Ouverture caisse
 ↓
Fonds initial
 ↓
Tableau de bord
```

## 21.2 Vente comptant

```text
Nouvelle vente (session de caisse ouverte obligatoire)
 ↓
Scan/recherche article
 ↓
Panier et remises validées
 ↓
Total net
 ↓
Espèces présentées
 ↓
Monnaie rendue = espèces présentées - part réglée en espèces
 ↓
Validation transactionnelle
 ↓
Stock - quantité vendue
Caisse + part nette conservée en espèces
Ticket (espèces présentées et monnaie séparées)
```

## 21.3 Vente à crédit totale

```text
Nouvelle vente
 ↓
Choisir client
 ↓
Ajouter articles
 ↓
Paiement immédiat = 0
 ↓
Validation
 ↓
Stock -
Créance +
Ticket crédit
```

## 21.4 Vente partiellement payée

```text
Total : 100,000 TND
 ↓
Part réglée en espèces (après monnaie) : 30,000 TND
 ↓
Reste dû : 70,000 TND
 ↓
Caisse +30,000 TND
Créance +70,000 TND
```

## 21.5 Règlement client

```text
Client
 ↓
Voir solde dû
 ↓
Nouveau règlement
 ↓
Espèces
 ↓
Validation
 ↓
Caisse +
Créance -
Reçu
```

## 21.6 Achat comptant

```text
Nouvel achat
 ↓
Choisir fournisseur
 ↓
Articles / quantités
 ↓
Paiement cash
 ↓
Validation
 ↓
Stock +
Caisse -
```

## 21.7 Achat à crédit / paiement partiel

```text
Achat : 1 000,000 TND
 ↓
Paiement cash : 400,000 TND
 ↓
Dette fournisseur : 600,000 TND
 ↓
Stock +
Caisse -400,000 TND
Dette +600,000 TND
```

## 21.8 Avoir de vente — remboursement cash

```text
Vente existante
 ↓
Retour client
 ↓
Créer avoir
 ↓
Définir état des articles
 ↓
Stock réintégré selon état
 ↓
Remboursement cash
 ↓
Caisse -
Avoir soldé
 ↓
Document d'avoir
```

## 21.9 Avoir de vente — solde client

```text
Vente existante
 ↓
Créer avoir
 ↓
Aucune sortie de caisse
 ↓
Avoir disponible client
 ↓
Utilisation sur prochaine vente
 ↓
Complément éventuel en espèces
```

## 21.10 Avoir d'achat

```text
Achat existant
 ↓
Retour fournisseur
 ↓
Créer avoir d'achat
 ↓
Stock -
 ↓
Imputation dette OU avoir disponible OU remboursement cash
 ↓
Mise à jour fournisseur/caisse
```

## 21.11 Clôture caisse

```text
Clôture
 ↓
Solde théorique
 ↓
Comptage réel
 ↓
Écart
 ↓
Commentaire si nécessaire
 ↓
Validation
 ↓
Rapport caisse
```

---

# 22. Modèle de données fonctionnel corrigé

Ce modèle fonctionnel décrit les principales tables et relations attendues. Les montants sont des entiers en millimes ; les quantités sont des entiers à échelle 1000. Les valeurs de solde sont calculées depuis les documents et opérations valides, et ne constituent pas des saisies manuelles.

## 22.1 Paramètres et numérotation

**PARAMETRES_MAGASIN** : id singleton, nom commercial, téléphone/adresse/logo facultatifs, identifiant fiscal facultatif, largeur de ticket, imprimante facultative, impression automatique, pied de ticket et informations fiscales générales.

**CATEGORIE_FISCALE** : id, code unique, libellé, taux_basis_points si applicable, mention de ticket facultative, actif. Les taux et les catégories sont saisis uniquement après validation du régime fiscal du magasin.

**COMPTEUR_DOCUMENT** : type de document, année, dernier numéro attribué. La combinaison type/année est unique ; l'attribution du prochain numéro est transactionnelle.

**SAUVEGARDE** : id, date/heure, chemin, type (automatique/manuelle/externe), statut, résultat et message d'erreur éventuel. Cette table conserve un historique ; les sauvegardes elles-mêmes sont des fichiers.

## 22.2 Catalogue et stock

**FAMILLE** : id, nom unique, description facultative, actif.

**SOUS_FAMILLE** : id, famille_id obligatoire, nom unique dans sa famille, description facultative, actif.

**EMPLACEMENT** : id, code facultatif, nom, rayon, étagère, description facultative, actif. Ce sont uniquement des repères dans le magasin ; aucun stock n'est porté par un emplacement.

**ARTICLE** : id, code unique généré, code-barres facultatif et unique s'il est renseigné, désignation obligatoire, description, famille_id facultatif, sous_famille_id facultatif, marque, unité, quantite_decimale, cout_moyen_millimes, dernier_prix_achat_millimes, prix_vente_ttc_millimes, remise_max_basis_points, stock_disponible_milliunites, stock_endommage_milliunites, stock_minimum_milliunites, emplacement_id facultatif, categorie_fiscale_id facultatif, photo facultative, actif.

**MOUVEMENT_STOCK** : id, article_id, type, compartiment (disponible/endommagé-bloqué), quantite_milliunites, stock_avant_milliunites, stock_apres_milliunites, date/heure, type/document source, motif facultatif, mouvement_inverse_id facultatif. Une correction directe est représentée par un document de correction et ses lignes, pas par une modification silencieuse de l’article.

**CORRECTION_STOCK** : id, numéro facultatif, date/heure, motif obligatoire, statut et lignes de correction.

**LIGNE_CORRECTION_STOCK** : id, correction_stock_id, article_id, compartiment, sens, quantite_milliunites ; chaque ligne génère un mouvement de stock.

**INVENTAIRE** : id, date de début, date de validation facultative, statut, remarque facultative.

**LIGNE_INVENTAIRE** : id, inventaire_id, article_id, compartiment, stock_theorique_milliunites, stock_compte_milliunites, ecart_milliunites. Une ligne est unique par inventaire, article et compartiment.

## 22.3 Clients et fournisseurs

**CLIENT** : id, code unique généré, nom obligatoire, téléphone/adresse/identifiant fiscal facultatifs, credit_autorise, plafond_credit_millimes facultatif, actif, remarque. Le solde dû et l'avoir disponible sont calculés depuis les documents et opérations valides.

**FOURNISSEUR** : id, code unique généré, raison_sociale obligatoire, contact/téléphone/adresse/identifiant fiscal/email facultatifs, actif, remarque. La dette et l'avoir disponible sont calculés depuis les documents et opérations valides.

## 22.4 Ventes

**VENTE** : id, numéro unique, date/heure, client_id facultatif seulement si la vente est réglée sans crédit ni avoir client, session_validation_id obligatoire, total_net_millimes, montant_cash_applique_millimes, especes_presentees_millimes, monnaie_rendue_millimes, statut, échéance facultative, motif_annulation facultatif. Le reste dû initial est calculé à partir du total, du cash appliqué et des opérations d'avoirs appliquées à la vente ; le reste dû courant tient aussi compte des règlements et compensations ultérieurs.

**LIGNE_VENTE** : id, vente_id, article_id, quantite_milliunites, prix_unitaire_millimes, type_remise, valeur_remise, remise_appliquee_millimes, montant_net_ligne_millimes, cout_moyen_moment_vente_millimes, code/libellé/taux de catégorie fiscale utilisés au moment de la vente si applicables.

**REGLEMENT_CLIENT** : id, numéro unique, client_id, montant_millimes, date/heure, référence/commentaire, statut. Tout règlement validé est en espèces et crée un mouvement de caisse.

**AFFECTATION_REGLEMENT_CLIENT** : id, reglement_client_id, vente_id, montant_millimes. Chaque règlement validé possède une ou plusieurs affectations ; la somme des affectations égale son montant et aucune affectation ne dépasse le solde dû de la vente concernée.

**AVOIR_VENTE** : id, numéro unique, vente_origine_id facultatif, client_id facultatif uniquement pour remboursement en espèces d'un retour anonyme, date/heure, montant_total_millimes, motif, statut. Au moins une ligne ou une justification de correction financière est nécessaire.

**LIGNE_AVOIR_VENTE** : id, avoir_vente_id, ligne_vente_origine_id facultatif, article_id facultatif si correction purement financière, quantite_milliunites, prix_unitaire_origine_millimes, remise_origine_millimes, montant_ligne_millimes, etat_retour. Une ligne sans retour physique ne génère pas de mouvement de stock.

**OPERATION_AVOIR_VENTE** : id, avoir_vente_id, type (utilisation_sur_vente/compensation_creance/remboursement_especes), vente_cible_id facultatif selon le type, montant_millimes, date/heure, statut, motif facultatif. Une opération de remboursement référence le mouvement de caisse correspondant. La somme des opérations valides ne dépasse jamais le montant de l'avoir ; le solde disponible est calculé.

## 22.5 Achats et fournisseurs

**ACHAT** : id, numéro unique, date/heure, fournisseur_id, référence facture fournisseur facultative, total_net_millimes, montant_cash_applique_millimes, statut, remarque, motif_annulation facultatif. Le reste dû initial tient compte des paiements en espèces et des avoirs fournisseurs appliqués à cet achat ; le solde courant tient compte des règlements et imputations ultérieurs.

**LIGNE_ACHAT** : id, achat_id, article_id, quantite_milliunites, prix_achat_unitaire_millimes, type_remise, valeur_remise, remise_appliquee_millimes, montant_net_ligne_millimes et informations fiscales historiques si applicables.

**REGLEMENT_FOURNISSEUR** : id, numéro unique, fournisseur_id, montant_millimes, date/heure, référence/commentaire, statut. Tout règlement validé est en espèces et crée un mouvement de caisse.

**AFFECTATION_REGLEMENT_FOURNISSEUR** : id, reglement_fournisseur_id, achat_id, montant_millimes. La somme des affectations égale le règlement ; aucune affectation ne dépasse le solde dû du document concerné.

**AVOIR_ACHAT** : id, numéro unique, achat_origine_id facultatif, fournisseur_id, date/heure, montant_total_millimes, motif, statut. Au moins une ligne ou une justification de correction financière est nécessaire.

**LIGNE_AVOIR_ACHAT** : id, avoir_achat_id, ligne_achat_origine_id facultatif, article_id facultatif pour correction financière, quantite_milliunites, prix_achat_origine_millimes, remise_origine_millimes, montant_ligne_millimes.

**OPERATION_AVOIR_ACHAT** : id, avoir_achat_id, type (imputation_dette/utilisation_sur_achat/remboursement_especes), achat_cible_id facultatif selon le type, montant_millimes, date/heure, statut, motif facultatif. Un remboursement reçu référence le mouvement de caisse. Le cumul des opérations valides ne dépasse pas le montant de l'avoir.

## 22.6 Caisse

**SESSION_CAISSE** : id, date_heure_ouverture, fonds_initial_millimes, date_heure_cloture facultative, solde_theorique_cloture_millimes facultatif, especes_comptees_millimes facultatif, ecart_millimes facultatif, remarque_ecart facultative, statut. Une contrainte de base de données impose au maximum une session ouverte. Les données de clôture sont enregistrées une seule fois et restent immuables.

**OPERATION_CAISSE_MANUELLE** : id, date/heure, catégorie (dépense/autre entrée/autre sortie), montant_millimes, motif obligatoire, statut ; elle référence le ou les mouvements de caisse créés, y compris toute inversion ultérieure.

**MOUVEMENT_CAISSE** : id, numéro unique, session_id, sens (entrée/sortie), catégorie, montant_millimes positif, date/heure, motif, type_source, id_source, mouvement_origine_id facultatif pour une inversion. Le montant est le montant net d'espèces réellement conservé ou déboursé. Une opération sans échange d'espèces n'a pas de mouvement de caisse.

Les mouvements de caisse issus d'une vente, d'un achat, d'un règlement ou d'une opération d'avoir sont rattachés à leur source. Les entrées/sorties manuelles passent par un document **OPERATION_CAISSE_MANUELLE** contenant le type (dépense/autre entrée/autre sortie), le montant, la date/heure et un motif obligatoire ; ce document génère le mouvement de caisse. Les références sources doivent être validées par la couche métier.

## 22.7 Traçabilité

**JOURNAL_AUDIT** : id, date/heure, action, module, type_document, document_id, ancienne_valeur facultative, nouvelle_valeur facultative, motif facultatif. Aucune identité utilisateur n'est demandée.

## 22.8 Contraintes relationnelles et d'intégrité

- Les lignes de vente/achat appartiennent à un seul document et référencent un article existant.
- Une sous-famille appartient à une famille ; un article peut rester sans famille pour permettre la création rapide.
- Les paiements et les avoirs doivent rester dans les limites des soldes ouverts ; une somme affectée supérieure au reste dû est interdite.
- Tout avoir qui utilise, compense ou rembourse des montants en plusieurs fois possède un historique d'opérations, pas seulement des champs de total modifiables.
- Les documents validés ne sont jamais supprimés physiquement ; l'annulation et les inversions laissent des traces.
- Les numéros sont uniques en base et attribués dans la transaction qui crée le document.
- Les effets d'une opération sur document, stock, caisse, soldes et audit sont enregistrés dans la même transaction SQLite.
- Les opérations cash inverses liées à une session clôturée sont enregistrées dans une session ouverte ultérieure, jamais dans la session clôturée.

# 23. Interface utilisateur

L'application doit être conçue pour un utilisateur non développeur.

## Principes

- boutons clairement nommés ;
- navigation simple ;
- recherche rapide ;
- clavier et souris ;
- écran de vente optimisé ;
- messages d'erreur simples ;
- confirmations avant actions sensibles ;
- affichage clair des montants en TND ;
- alertes visuelles sur dettes, stock et caisse ;
- historique facilement consultable.

## Saisie rapide — règle générale

La saisie doit être minimale. Le logiciel complète et calcule automatiquement tout ce qu’il peut.

### Champs obligatoires minimum

- **Article :** désignation et prix de vente. Le code article est généré automatiquement. Code-barres, famille, sous-famille, prix d’achat, emplacement et stock minimum peuvent être complétés plus tard. Le stock initial vaut 0 par défaut.
- **Client :** nom uniquement. Téléphone et adresse sont facultatifs. Un client est obligatoire si la vente crée un reste dû ou utilise un avoir client ; il reste facultatif pour une vente réglée immédiatement sans avoir client.
- **Fournisseur :** nom ou raison sociale. Téléphone, adresse, contact et identifiants sont facultatifs.
- **Vente comptant :** aucun client requis si aucun avoir client n’est utilisé ; quantité 1 par défaut ; le prix et le total sont calculés automatiquement.
- **Achat :** fournisseur, article, quantité et prix d’achat. La référence de facture est facultative si elle n’est pas disponible.
- **Avoir :** créer l’avoir directement depuis la vente ou l’achat d’origine ; les lignes, prix et montants sont préremplis.

### Fonctions destinées à éviter la double saisie

- date et heure remplies automatiquement ;
- codes et numéros de documents générés automatiquement ;
- sous-totaux, remises, total, monnaie à rendre, solde client/fournisseur et marge calculés automatiquement ;
- ajout d’un article par scan de code-barres ou recherche par nom ;
- bouton `Créer article rapide` depuis la vente lorsque l’article n’existe pas ;
- mémorisation du dernier prix d’achat connu, avec confirmation lors de la réception suivante ;
- possibilité d’importer la liste initiale des articles par Excel/CSV ;
- familles et sous-familles courantes préchargées, modifiables ultérieurement ;
- aucune saisie manuelle du stock après une vente, un achat ou un retour : les mouvements sont générés automatiquement.

Le système ne doit pas demander deux fois une information déjà connue depuis le document d’origine.

## Écran de vente

Exemple :

```text
[ SCAN / RECHERCHE ARTICLE ]

Article              Qté      Prix       Total
------------------------------------------------
Stylo bleu            2       1,500       3,000
Cahier                 3       4,500      13,500
------------------------------------------------
                              TOTAL : 16,500 TND

Client : [ Facultatif / obligatoire pour crédit ]

[ COMPTANT ]   [ CRÉDIT ]

Espèces présentées : 20,000 TND
Monnaie rendue :       3,500 TND
Part cash conservée : 16,500 TND

          [ METTRE EN ATTENTE ]   [ VALIDER LA VENTE ]
```

Pour une vente à crédit :

```text
Total vente        : 100,000 TND
Payé cash          : 30,000 TND
Reste dû           : 70,000 TND
```

---

# 24. Notifications et alertes

Afficher des alertes pour :

- stock nul ;
- stock sous minimum ;
- client ayant une dette ;
- client proche/dépassant le plafond ;
- fournisseur avec solde dû ;
- caisse ouverte non clôturée ;
- écart de caisse ;
- sauvegarde absente ou ancienne ;
- opération bloquée par une règle métier ;
- article sans prix de vente ;
- article sans stock minimum si la règle l'exige.

---

# 25. Sécurité et traçabilité

L'application ne comporte **ni écran de connexion, ni compte utilisateur, ni mot de passe applicatif**. Elle est conçue pour le propriétaire unique du magasin.

Prévoir néanmoins :

- journal d'audit automatique ;
- confirmations avant les actions sensibles ;
- motif obligatoire pour les annulations et corrections importantes ;
- traçabilité des changements de prix ;
- traçabilité des corrections de stock ;
- traçabilité des mouvements de caisse ;
- historique des avoirs, remboursements et compensations ;
- sauvegardes régulières et restauration contrôlée.

Le journal d'audit conserve la date/heure, l'action, le module, le document concerné, l'ancienne et la nouvelle valeur lorsque cela est pertinent, ainsi que le motif. Il ne demande pas d'identifiant utilisateur.

La base de données et les fichiers de sauvegarde doivent être protégés par les droits d’accès du système d’exploitation sur le poste qui héberge l'application.

---

# 26. Critères d'acceptation principaux

## Scénario A — Vente cash et monnaie rendue

```text
Créer un article et ouvrir la caisse avec un fonds initial
→ vendre des articles pour un total de 16,500 TND
→ le client présente 20,000 TND
→ monnaie rendue : 3,500 TND
→ mouvement net de caisse : +16,500 TND
→ stock diminué des quantités vendues
→ ticket imprimé avec total, espèces présentées et monnaie rendue
```

## Scénario B — Vente totalement à crédit

```text
Choisir client
→ vendre article
→ payer 0 TND
→ stock diminué
→ caisse inchangée
→ créance client créée
```

## Scénario C — Vente partiellement payée

```text
Vente 100 TND
→ paiement cash 30 TND
→ reste crédit 70 TND
→ caisse +30
→ créance +70
```

## Scénario D — Règlement client

```text
Dette client 70 TND
→ règlement cash 50 TND
→ caisse +50
→ dette restante 20 TND
→ reçu généré
```

## Scénario E — Achat cash

```text
Achat 1 000 TND
→ paiement cash 1 000 TND
→ stock augmenté
→ caisse -1 000 TND
→ dette fournisseur 0
```

## Scénario F — Achat à crédit

```text
Achat 1 000 TND
→ paiement cash 0 TND
→ stock augmenté
→ caisse inchangée
→ dette fournisseur 1 000 TND
```

## Scénario G — Achat partiellement payé

```text
Achat 1 000 TND
→ paiement cash 400 TND
→ stock augmenté
→ caisse -400 TND
→ dette fournisseur 600 TND
```

## Scénario H — Avoir de vente avec remboursement cash

```text
Vente existante
→ retour article
→ créer avoir 20 TND
→ stock + selon état de l'article
→ remboursement cash 20 TND
→ caisse -20 TND
→ avoir soldé
```

## Scénario I — Avoir de vente conservé

```text
Vente existante
→ avoir 20 TND
→ aucune sortie de caisse
→ avoir client +20 TND
→ prochaine vente 35 TND
→ avoir utilisé 20 TND
→ espèces reçues 15 TND
→ caisse +15 TND
```

## Scénario J — Avoir d'achat

```text
Achat existant
→ retour fournisseur
→ avoir 100 TND
→ stock - selon quantité retournée
→ avoir imputé sur dette ou conservé
→ ou remboursement reçu en cash
```

## Scénario K — Clôture caisse

```text
Solde théorique
→ comptage réel
→ écart calculé
→ remarque si nécessaire
→ clôture
→ rapport
```

## Scénario L — Inventaire

```text
Stock théorique
→ comptage réel
→ écart
→ validation
→ correction de stock historisée
```

## Scénario M — Retour sur vente anonyme

```text
Vente comptant sans client : 20 TND
→ retour de l'article
→ création de l'avoir
→ seul le remboursement cash est proposé
→ caisse -20 TND
→ stock + selon l'état de l'article
```

## Scénario N — Vente sans caisse ouverte

```text
Aucune session ouverte
→ tentative de validation d'une vente
→ opération bloquée : « Ouvrez la caisse pour continuer »
→ stock et caisse inchangés
```

## Scénario O — Annulation après clôture

```text
Vente de 20 TND dans la session 1, puis session 1 clôturée
→ annulation de la vente le lendemain
→ mouvement de caisse -20 TND dans la session 2 (ouverte), avec référence à la vente
→ session 1 inchangée
→ stock réintégré
```

## Scénario P — Coût moyen pondéré

```text
Stock 10 unités à 2,000 TND
→ achat de 10 unités à 3,000 TND
→ coût moyen = 2,500 TND
→ vente d'une unité : marge calculée avec 2,500 TND
```

## Scénario Q — Erreur pendant une opération

```text
Vente en cours de validation
→ erreur technique avant la fin
→ aucun changement de stock, de caisse ni de solde
```

---

## Scénario R — Règlement affecté à plusieurs ventes

```text
Client possède deux ventes restant dues : 30,000 et 40,000 TND
→ il règle 50,000 TND en espèces
→ règlement affecté : 30,000 à la première vente et 20,000 à la seconde
→ caisse +50,000 TND
→ reste dû global : 20,000 TND
→ somme des affectations = montant du règlement
```

## Scénario S — Utilisation partielle d'un avoir

```text
Avoir client valide : 25,000 TND
→ application de 10,000 TND à une vente du même client
→ le solde d'avoir devient 15,000 TND
→ aucune opération de caisse due à l'utilisation de l'avoir
→ le montant utilisé ne peut pas dépasser le solde de l'avoir ni le reste à payer de la vente
```

---

# 27. Priorité des fonctionnalités

## Version 1 — Obligatoire

- dashboard ;
- caisse ouverture/clôture ;
- ventes comptant ;
- mise en attente d'une vente ;
- ventes à crédit ;
- paiements partiels cash ;
- règlements clients cash ;
- tickets ;
- avoirs de vente ;
- achats comptant ;
- achats à crédit ;
- paiements partiels fournisseurs ;
- règlements fournisseurs cash ;
- avoirs d'achat ;
- articles ;
- import initial d’articles depuis Excel/CSV ;
- familles ;
- sous-familles ;
- emplacements ;
- stock ;
- inventaire ;
- clients ;
- fournisseurs ;
- rapports ;
- paramètres ;
- sauvegarde/restauration ;
- audit des actions sensibles.

## Version 2 — Améliorations possibles

- import avancé depuis Excel/CSV ;
- import fournisseurs/clients ;
- impression d'étiquettes code-barres ;
- gestion multi-langue français/arabe ;
- application tablette/mobile ;
- statistiques avancées ;
- notifications automatiques de réapprovisionnement.

---

# 28. Fonctionnalités explicitement exclues

La version de base ne doit pas gérer :

- gestion de plusieurs magasins / établissements ;
- gestion de plusieurs caisses ou sélection d’une caisse ;
- gestion de dépôt, entrepôt ou réserve comme emplacement de stock indépendant ;
- stock distinct par magasin ou dépôt ;
- transfert de stock entre magasins ou dépôts ;

- paiement par carte ;
- paiement par chèque ;
- virement bancaire comme moyen de paiement ;
- paiement en ligne ;
- portefeuille électronique ;
- cryptomonnaie ;
- facture de vente (le ticket est le seul document remis au client) ;
- backend, serveur, API ou synchronisation cloud ;
- comptabilité générale complète ;
- paie ;
- ressources humaines avancées.

---

# 29. Contraintes et conventions

- Pays : **Tunisie** ;
- Devise : **Dinar tunisien (TND)** ;
- Langue principale : français ;
- Arabe : option possible ;
- Fuseau horaire : **Africa/Tunis** ;
- Paiement : **espèces uniquement** ;
- Type de magasin : fournitures scolaires / papeterie ;
- Mode : mono-utilisateur (propriétaire / gérant) ;
- Périmètre : un seul magasin commercial ;
- Aucun module de gestion de dépôts ou d'entrepôts ;
- Localisation article : rayon/étagère uniquement, sans stock distinct ;
- Plateforme : application desktop Flutter (Windows en priorité), 100 % locale, sans backend ;
- Base de données : SQLite locale ;
- Version tablette/mobile : hors périmètre de la V1 ;
- Ticket : imprimante thermique 58/80 mm ;
- Stock : mise à jour automatique à chaque opération validée.

---

# 30. Règles de calcul et cohérence financière

## 30.1 Solde client

Pour chaque vente active, le reste dû initial est calculé après paiement cash et avoir appliqué à la vente. Le solde courant est :

```text
Solde dû client
= Somme des restes dus initiaux des ventes actives
- Règlements clients affectés à ces ventes
- Compensations d'avoirs affectées à ces créances
```

Les paiements cash reçus dès la vente et les avoirs utilisés directement sur une vente sont déjà pris en compte dans le reste dû initial : ils ne doivent pas être soustraits une deuxième fois. Les avoirs clients conservés restent séparés de la dette client.

## 30.2 Solde fournisseur

```text
Dette fournisseur courante
= Somme des soldes dus initiaux des achats actifs
- Règlements fournisseurs affectés aux achats
- Opérations d'avoirs imputées sur des dettes existantes
```

Les paiements cash et avoirs fournisseurs appliqués dès la validation sont déjà intégrés au solde dû initial de l'achat. Les avoirs non utilisés restent séparés de la dette fournisseur.

## 30.3 Solde avoir client

```text
Avoir disponible
= Montant total de l'avoir
- Utilisations sur ventes
- Compensations de créances
- Remboursements en espèces
```

Seules les opérations valides et non annulées entrent dans ce calcul.

## 30.4 Solde avoir fournisseur

```text
Avoir disponible
= Montant total de l'avoir
- Imputations sur dettes
- Utilisations sur achats
- Remboursements reçus en espèces
```

Seules les opérations valides et non annulées entrent dans ce calcul.

## 30.5 Caisse

```text
Solde théorique de la session
= Fonds initial
+ Somme des mouvements d'entrée
- Somme des mouvements de sortie
```

Chaque mouvement est compté une seule fois dans la session à laquelle il est affecté. Une dette, une créance ou une utilisation d'avoir sans mouvement réel d'espèces ne modifie pas la caisse. Une inversion postérieure à la clôture est enregistrée dans une session ouverte ultérieure.

# 31. Résultat attendu

À la fin du projet, le gérant doit pouvoir gérer le magasin sans intervention d'un développeur pour les opérations quotidiennes.

Le système doit permettre de répondre immédiatement à ces questions :

```text
1. Qu'est-ce que j'ai vendu ?
2. Combien ai-je réellement encaissé en espèces ?
3. Combien dois-je à mes fournisseurs ?
4. Quel est mon stock disponible ?
5. Où se trouve chaque article ?
6. Qui me doit de l'argent ?
7. Quels avoirs clients sont disponibles ?
8. Quels avoirs fournisseurs sont disponibles ?
9. Quelle est ma marge commerciale ?
10. Y a-t-il un écart de caisse ?
```

Architecture fonctionnelle :

```text
                  ┌──────────────┐
                  │   ARTICLES   │
                  └──────┬───────┘
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
        ┌───────────┐         ┌───────────┐
        │  ACHATS   │         │  VENTES   │
        └─────┬─────┘         └─────┬─────┘
              │                     │
              ↓                     ↓
          STOCK +                 STOCK -
              │                     │
              └──────────┬──────────┘
                         ↓
                    ┌─────────┐
                    │  CAISSE │
                    │  CASH    │
                    └────┬────┘
                         │
              ┌──────────┼───────────┐
              ↓          ↓           ↓
           CLIENTS   FOURNISSEURS   AVOIRS
              │          │           │
              └──────────┼───────────┘
                         ↓
                 RAPPORTS / DASHBOARD
```

---

# 32. Points de validation avant mise en production

Les éléments suivants doivent être validés avant la mise en production :

- régime fiscal réel du magasin (assujettissement à la TVA, ventilation sur le ticket) ;
- taux/catégories fiscales applicables ;
- mentions obligatoires sur les tickets ;
- acceptation du ticket seul par la clientèle (clients professionnels, écoles) ;
- numérotation documentaire ;
- règles internes de crédit client ;
- règles internes de crédit fournisseur ;
- politique de retour et d'articles endommagés ;
- confirmation du coût moyen pondéré comme méthode de valorisation et d'analyse de marge ;
- test de l'imprimante thermique réelle et du lecteur de code-barres ;
- fonctionnement mono-utilisateur sans écran de connexion ;
- procédure de sauvegarde et restauration ;
- format d'impression utilisé par le magasin.

> Ce cahier des charges décrit les besoins fonctionnels de l'application. Les règles fiscales et réglementaires tunisiennes doivent être confirmées selon la situation réelle de l'entreprise avant la mise en production.

---

# 33. Checklist de livraison

- [ ] Dashboard
- [ ] Caisse ouverture/clôture
- [ ] Vente comptant
- [ ] Vente à crédit
- [ ] Paiement partiel cash
- [ ] Règlement client cash
- [ ] Tickets
- [ ] Avoirs de vente
- [ ] Remboursement client cash
- [ ] Utilisation des avoirs clients
- [ ] Achats cash
- [ ] Achats à crédit
- [ ] Paiement partiel fournisseur
- [ ] Règlement fournisseur cash
- [ ] Avoirs d'achat
- [ ] Remboursement fournisseur cash
- [ ] Articles
- [ ] Familles
- [ ] Sous-familles
- [ ] Localisation des articles par rayon/étagère
- [ ] Stock disponible
- [ ] Stock endommagé/bloqué
- [ ] Mouvements de stock
- [ ] Inventaire
- [ ] Clients
- [ ] Fournisseurs
- [ ] Rapports ventes
- [ ] Rapports achats
- [ ] Rapports caisse
- [ ] Rapports stock
- [ ] Rapports clients
- [ ] Rapports fournisseurs
- [ ] Marge commerciale
- [ ] Numérotation des documents
- [ ] Statuts et annulations
- [ ] Audit des actions sensibles
- [ ] Identité du commerce (un seul établissement)
- [ ] Paramètres fiscaux configurables
- [ ] Paramètres tickets
- [ ] Sauvegarde
- [ ] Restauration
- [ ] Mise en attente d'une vente
- [ ] Mise au rebut du stock endommagé
- [ ] Coût moyen pondéré
- [ ] Transactions atomiques (stock, caisse, soldes, audit)
- [ ] Test imprimante thermique et lecteur de code-barres
- [ ] Tests des scénarios métier

---

# 34. Définition finale du projet

**Nom du projet :** Gestion Magasin Fournitures Scolaires — Tunisie

**Objectif :** fournir une application simple de gestion commerciale pour une papeterie/magasin de fournitures scolaires.

**Périmètre fixe :** un seul établissement, un seul stock global et aucun dépôt/entrepôt. La localisation des articles se limite aux rayons, étagères et vitrines du magasin.

**Plateforme :** application desktop Flutter, 100 % locale, base SQLite, sans backend.

**Principe central :** tous les règlements monétaires effectués dans l'application sont exclusivement en espèces. Les ventes à crédit créent des créances clients, les achats non réglés créent des dettes fournisseurs, et les avoirs sont suivis comme des soldes commerciaux distincts de la caisse.

**Priorités :**

1. rapidité de vente ;
2. exactitude de la caisse espèces ;
3. exactitude du stock ;
4. suivi des crédits ;
5. gestion fiable des retours et avoirs ;
6. traçabilité ;
7. simplicité des rapports et du dashboard.

**État du cahier des charges :** version 7.0 clarifiée. Les points fiscaux restant spécifiques au régime réel du magasin doivent être validés avant la mise en production.

---

# 35. Journal des changements — version 7.0

- Clarification de la représentation exacte des montants (millimes) et quantités (échelle 1000), sans nombres flottants pour les valeurs métier.
- Définition de la différence entre espèces présentées, monnaie rendue et part réellement conservée en caisse.
- Clarification du calcul des restes dus en présence de règlements et d'avoirs appliqués.
- Ajout explicite des affectations de règlements clients/fournisseurs aux documents soldés.
- Clarification des remises : type, valeur saisie, montant réellement appliqué et contrôle du plafond.
- Autorisation de traiter un avoir en plusieurs opérations cumulées, dans la limite de son montant total ; ajout d'un historique distinct des utilisations, compensations et remboursements.
- Précision des conditions d'annulation des documents dépendants et des opérations après clôture de caisse.
- Révision du modèle de données fonctionnel pour qu'il corresponde aux relations et contraintes du diagramme de classes corrigé.
- Les champs solde dû et avoir disponible sont des soldes calculés et ne doivent pas être librement modifiés.
- Rappel du périmètre stable : un gérant, un magasin, une caisse, aucun dépôt, aucune authentification applicative, fonctionnement local sans backend.
>>>>>>> main
