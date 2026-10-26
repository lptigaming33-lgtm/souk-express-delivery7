# Souk Express — livraison de quartier à Agadir

Plateforme de livraison fonctionnelle (Next.js App Router + PostgreSQL + Drizzle ORM), en français et arabe RTL,
branding orange / vert / blanc, orientée mobile (Android) et installable en PWA.

## Comptes de démonstration (réels, stockés en base)

| Rôle | Téléphone | Mot de passe |
| --- | --- | --- |
| Client | `0611111111` (Amine Tazi) | `Client2026!` |
| Client | `0611111112` / `0611111113` | `Client2026!` |
| Admin | `0600000001` | `SoukAdmin2026!` |
| Livreurs (4) | `0600000011` → `0600000014` | `Driver2026!` |

Les livreurs sont Yassine El Amrani, Hamza Bouzid, Reda Ait Taleb et Soufiane Lahlou (motos, Agadir).

## Fonctionnalités

- **Client** : boutiques et produits lus en base, recherche FR/AR, panier signé (cookie http-only + HMAC),
  totaux recalculés côté serveur, checkout téléphone/adresse marocains validés, paiement à la livraison (COD),
  commandes persistées avec numéro unique, suivi réel (timeline), historique, formulaire de suivi public
  (n° + téléphone), WhatsApp boutique/livreur/support, Google Maps, tampons fidélité en base + remise réelle.
- **Admin** : session sécurisée + contrôle de rôle, dashboard du jour, file de commandes, accept/refus (motif),
  assignation et réassignation des 4 livreurs, annulation, encaissement cash et remise au souk, gestion du
  catalogue (boutiques, produits, prix, activation), zones et frais, seuil de livraison offerte, paramètres
  fidélité, rapports quotidiens (par livreur, par boutique, top produits, cash).
- **Livreur** : compte individuel, uniquement ses livraisons, contact/adresse client, WhatsApp, Google Maps,
  ramassage autorisé puis livraison avec montant cash contrôlé, cash en main / à remettre.
- **API JSON** (en plus des server actions) : `/api/auth/login`, `/api/auth/logout`, `/api/me`, `/api/zones`,
  `/api/orders` (GET/POST), `/api/orders/[orderNumber]`, `/api/admin/orders`, `/api/admin/orders/[id]`,
  `/api/driver/orders`, `/api/driver/orders/[id]`, `/api/health`.

## Base de données

11 tables : `users`, `sessions`, `zones`, `businesses`, `products`, `orders`, `order_items`, `order_events`,
`loyalty_stamps`, `cash_handovers`, `settings`. Contraintes SQL (unicité téléphone, n° de commande,
idempotence, index, CHECK sur prix/montants/quantités, FK ON DELETE).

Préparation locale (Windows PowerShell) :

1. Installez PostgreSQL et créez la base `app_db` si elle n'existe pas.
2. Copiez `.env.example` vers `.env.local`, puis remplacez `YOUR_POSTGRES_PASSWORD` par le mot de passe PostgreSQL local. L'URL doit être de la forme `postgresql://postgres:VOTRE_MOT_DE_PASSE@127.0.0.1:5432/app_db` (un seul `/` avant `app_db`).
3. Installez les dépendances avec `npm install`.
4. Vérifiez la connexion et les tables avec `npm run db:check`.
5. Appliquez les migrations avec `npm run db:migrate`.
6. Vérifiez de nouveau avec `npm run db:check`, puis lancez `npm run db:seed` si vous souhaitez les données de démonstration.
7. Lancez `npm run dev`.

Commandes Drizzle : `npm run db:generate` génère une migration à partir des changements de schéma, tandis que `npm run db:migrate` applique les migrations existantes. N'exécutez pas `db:generate` pour tenter de réparer une base déjà en panne sans examiner d'abord son état.

Si seule la table `products` est absente alors que `businesses` existe déjà et que la migration initiale est enregistrée comme appliquée, `npm run db:repair-products` crée la table manquante avec les colonnes, la clé étrangère, la contrainte de prix et l'index définis dans le schéma. La commande تعمل داخل transaction، ما كتمسحش الجداول أو الصفوف الموجودة، وكتفشل بلا تغييرات إذا كانت `businesses` ناقصة. لا تستعملها كبديل عن migration الأول على قاعدة فارغة أو ناقصة بالكامل.

```powershell
Copy-Item .env.example .env.local
npm install
npm run db:check
npm run db:migrate
npm run db:check
npm run db:seed
npm run dev
```

Données de démonstration : 6 boutiques, 26 produits, 8 zones, 8 comptes, 8 commandes couvrant tous les statuts,
cash encaissé et historique fidélité.

## Tests

Le serveur de production a été testé en HTTP réel (46 vérifications) : création de commande, anti-double
soumission, calcul serveur des totaux, autorisations admin/livreur, cycle complet
`pending → accepted → assigned → picked_up → delivered`, cash collecté/remis, persistance après reconnexion.

```bash
npm run build && PORT=3100 npm run start &
BASE_URL=http://127.0.0.1:3100 PRODUCT_ID=1 node scripts/e2e.mjs
```

## Sécurité

- Mots de passe scrypt (sel aléatoire), sessions opaques hachées SHA-256 en base, cookie http-only/SameSite=Lax.
- Autorisations vérifiées à chaque action/API (customer, admin, driver), transitions contrôlées par un service
  transactionnel avec verrouillage de ligne.
- Anti double-soumission par clé d'idempotence unique + index ; panier signé HMAC pour éviter la falsification.
- Secrets en variables d'environnement (`AUTH_SECRET`, `DATABASE_URL`, identifiants admin), aucune clé payante.

## Limites connues

- Pas d'agrégateur de paiement (COD uniquement, budget 0 MAD).
- Les frais de livraison sont définis par zone (pas de tarification à la distance).
- PWA avec cache de shell + page hors ligne ; notifications push non implémentées.
- Déploiement : l'application est prête pour un hébergement Node + PostgreSQL ; aucun déploiement externe n'a été
  effectué dans cet environnement (URL locale uniquement).
