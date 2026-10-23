# FX SENTINEL V2.2 — moteur de simulation contrôlée

## Périmètre
- Branche cible : `develop/v2` uniquement.
- Ne pas fusionner vers `main` ni modifier `backup/stable-v1` avant validation.
- Binance Spot public uniquement ; aucune clé API privée.
- Aucun ordre réel, retrait, trading live ou appel privé de trading.
- Simulation automatique **désactivée par défaut** ; activation explicite dans l’interface.

## Paramètres expérimentaux
- Capital virtuel initial : 1 000 USDT.
- Risque théorique visé : 1 % de l’équité par position.
- Positions longues Spot uniquement.
- Stop-loss indicatif : 2 % sous le cours observé à l’entrée.
- Take-profit indicatif : 4 % au-dessus du cours observé à l’entrée.
- Frais estimés : 0,10 % par côté.
- Slippage estimé : 0,05 % par côté.
- Maximum : 3 positions ouvertes.
- Candidats : paire USDT, variation 24 h de +3 % à +8 %, volume coté >= 1 000 000 USDT.
- Une même paire est protégée contre les réouvertures pendant 30 minutes.
- Durée maximale d’une position : 24 h, évaluée uniquement lors d’un rafraîchissement.

Ces seuils sont des hypothèses techniques de démonstration, pas une stratégie validée. Les frais réels peuvent différer. Le stop-loss ne garantit pas une perte maximale en cas de gap ou de données manquantes.

## Calculs
- Entrée simulée = prix observé majoré du slippage.
- Quantité cible = budget de risque / (entrée simulée - stop indicatif), puis limitée au capital disponible après frais.
- Frais = notionnel * taux de frais.
- Sortie simulée = prix observé minoré du slippage.
- P&L net réalisé = (sortie simulée - entrée simulée) * quantité - frais d’entrée - frais de sortie.
- Équité et P&L latent sont estimés au dernier prix disponible.
- Les sorties stop/take-profit ne sont évaluées qu’au rafraîchissement : le prix exact de franchissement est inconnu.
- L’objectif mensuel est informatif et ne déclenche jamais de position.

## Persistance
Le capital, les positions ouvertes et l’historique sont stockés dans `localStorage` sur cet appareil. Ce n’est pas une base serveur, un journal inviolable ou une surveillance 24 h/24. Effacer les données du navigateur peut les supprimer. Le backend de l’étape 4 sera nécessaire pour une persistance durable.

## Tests avant publication
1. Au chargement, simulateur arrêté, capital à 1 000 USDT, aucune transaction inventée.
2. Sans activation, aucun trade simulé ne s’ouvre.
3. Après activation, seules les candidates remplissant les filtres peuvent être simulées.
4. Une seule position ouverte par paire ; maximum de trois positions.
5. Les frais d’entrée réduisent le capital disponible.
6. À la clôture, le solde reçoit le produit net et le journal enregistre les frais.
7. Un stop/take dépassé utilise le cours observé et précise le dépassement.
8. Réseau coupé ou données périmées : aucune nouvelle position.
9. Après actualisation, positions et historique sont restaurés localement.
10. Export JSON : paramètres et historique inclus.
11. Réinitialisation confirmée : simulateur arrêté, capital à 1 000 USDT, positions et historique de simulation effacés.
12. Vérifier l’absence de clé privée et d’endpoint d’ordre réel.
13. Tester mobile/desktop, erreurs de stockage et valeurs numériques anormales.
14. Ne fusionner vers `main` qu’après revue et tests concluants.

## Limites
- Le navigateur peut suspendre les tâches en arrière-plan.
- Les signaux d’entrée sont simples et non démontrés rentables.
- Les tickers 24 h ne révèlent pas la séquence intrabougie des prix.
- L’historique local peut être modifié ou perdu ; il n’est pas auditable comme un journal serveur.
