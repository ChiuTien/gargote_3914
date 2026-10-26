### 0.2. Lire le schema rationnel
#### 1. Schema relationnel des 4 tables:

* `categorie`:(<u>__id__</u>, libelle)
* `plat`:(<u>__id__</u>, nom, prix, #id_categorie)
* `commande`:(<u>__id__</u>, date_commande, numero_table)
* `commande_ligne`:(<u>__id__</u>, #id_commande, #id_plat, quantite, prix_unitaire)

#### 2. Pourquoi commande_ligne contient-elle une colonne prix_unitaire, alors que le prix est déjà dans plat ? Donnez un exemple concret où les deux valeurs seraient différentes.

* `commande_ligne` contient une colonne prix_unitaire afin d'eviter le probleme de changement de prix dans le future.
* Exempls, Bao a la liste de tous les plats qu'il a vendu dans commandes, mais si le prix d'un plat change dans le futur alors ces donnees d'avant seront faussee.

#### 3. Que se passe-t-il pour les lignes d’une commande quand on supprime la commande ? Quelle instruction en est responsable ?

* Quand on supprime la commande, toutes les lignes de cette commande seront supprimer
* l'instruction responsable est: `ON DELETE CASCADE` sur la colonne id_commande de commande_ligne

#### 4. Repérez trois données absurdes que le schéma actuel accepte sans broncher (indice : essayez d’insérer à la main un prix négatif). Notez-les : vous les corrigerez au niveau 1 et 2.

1. ```sql
        -- La table plat accepte les prix <= 0 --
        INSERT INTO plat(nom,prix,id_categorie) VALUES('Test',-500,1);
    ```
2. ```sql
        -- La table commande accepte un numero de table invalide --
        INSERT INTO commande(numero_table) VALUES(-1);
    ```
3. ```sql
        -- La table commande_ligne accepte les quantites et prix negatifs --
        INSERT INTO commande_ligne(id_commande,id_plat,quantite,prix_unitaire) VALUES(1,1,-10,-1000);
    ```
### 0.3. Interroger l'API a la main
#### 5. Pour la requête b, quel code obtenez-vous ? Pourquoi pas 200 ?

* J'obtient un code __201__
* Ce n'est pas un __200__ vu que c'est une creation et le code d'une creation c'est __201__

#### 6. Pour la requête d, qui bloque la suppression : PHP ou PostgreSQL ? Montrez la ligne de code qui transforme ce blocage en réponse 409.

* C'est `POSTGRESQL` qui bloque la suppression
* ```php
        try {
            $this->plats->delete($id);
        } catch (\Throwable $e) {
            // Le plat apparaît dans des commandes : la clé étrangère bloque la suppression.
            return $this->erreur('Ce plat figure déjà dans des commandes, il ne peut pas être supprimé.', 409);
        }
    ```

#### 7. Créez une commande avec curl (table 6, deux Romazava et un Café). Écrivez la commande curl complète.

```bash
    curl -i -X POST \
    http://localhost:8080/api/commandes \
    -H "Content-Type: application/json" \
    -d '{"numero_table":"6", "lignes":[ { "id_plat":"4","quantite":"2" } , { "id_plat":"10", "quantite":"1" }]}'
```

### 0.4. Suivre une requête de bout en bout
 # | Étape | Fichier | Fonction / ligne |
|---|---|---|---|
| 1 | Le formulaire est soumis | `components/PlatForm.vue` | `enregistrer() / ligne 33` |
| 2 | La requête HTTP est construite | `api.js` | `requete()` |
| 3 | Vite transmet `/api/...` au port 8080 |`vite.config.js` | `defineConfig()` |
| 4 | CodeIgniter choisit le contrôleur | `config/Routes.php`| `get()/post()` |
| 5 | Le contrôleur lit le JSON |`Api/ApiController.php`|`donneesJson()`|
| 6 | Les règles de validation sont vérifiées |`Api/Plat.php`|`create()/update()`|
| 7 | La ligne est insérée |`Api/Plat.php`|`insert()`|
| 8 | Le formulaire prévient l’écran parent |`src/components/PlatFrom.vue`| `emit(…)` |
| 9 | L’écran recharge la liste | `views/CarteView.vue` |`charger()`|

#### 8.Où est le code métier de la création d’une commande ? Pourquoi n’est-il pas dans le contrôleur Commandes.php ?

- Le metier de la creation d'une commane se trouver dans `services/commdandeService.php`
- Il n'est pas dans le controleur commandes.php afin de bien separer le service metier et le controlleur

### 0.5 Repérer les notions de Vue 

| Notion | À quoi ça sert | Votre exemple |
|--------|----------------|---------------|
| `ref()` | une variable dont le changement met l’écran à jour |`App.vue ligne 14`|
| `computed()` | une valeur calculée à partir d’autres, recalculée automatiquement |`App.vue ligne 15`|
| `v-for` | répéter un élément pour chaque élément d’une liste |`PlatList.vue ligne 20`|
| `v-if` / `v-else` | afficher un élément seulement si une condition est vraie |`PlatForm.vue ligne 60`|
| `v-model` | lier un champ de formulaire à une variable, dans les deux sens |`PlatForm.vue ligne 64`|
| `defineProps` | recevoir des données du composant parent |`PlatForm.vue ligne 5`|
| `defineEmits` / `emit` | envoyer un événement au composant parent |`PlatForm.vue ligne 9`|
| `onMounted` | exécuter du code quand le composant apparaît |`CarteView.vue ligne 40`|

#### 9. Panier.vue ne modifie jamais la quantité lui-même : il émet plus et moins. Pourquoi ce choix ? Qui modifie réellement la quantité ?

- Je ne sais pas pourquoi ce choix
- C'est le composant parent qui modifie reellement la quantite

#### 10. Dans PlatForm.vue, à quoi sert le watch ? Que se passerait-il sans lui quand on clique sur Modifier d’un autre plat ?

- La fonction watch sert de verification pour le formulaire du plat
- Si on l'enleve, le formulaire accepterait les champs vide

### 0.6 Premières modifications

#### 9. FAIT

#### 10. FAIT

#### 11. La catégorie apparaît dans la liste déroulante du formulaire sans que vous ayez écrit de code. Expliquez pourquoi, en citant la route et le composant concernés.

- Le composant concerner est le PlatFrom.vue , les categories sont passer via props categories par le parent CarteView.vue