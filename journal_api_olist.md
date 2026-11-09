# Journal de l'API Olist — FastAPI

**Nom :** …
**Dates :** J1 … · J2 …
**Fichier rendu :** api_olist_….zip

> Compétences : WCS C1.6 · C2.4 · C2.1 · C2.5 (niveau 2 · Adapter) — Simplon C6 · C7 (partiel)
> Remplis les parties **dans l'ordre**. Remplace chaque « … ». Une phrase suffit sauf mention contraire.

**Chiffres de référence (export de lundi)**
- Lignes : 110 197 · Commandes livrées : 96 478
- CA total : 15 424 463,65 R$ · Panier moyen : 159,88 R$
- Novembre 2017 : 1 154 559,53 R$ et 7 289 commandes
- SP : 5 771 713,93 R$, 40 501 commandes, 37,42 % du CA

---

# J1 — L'API sur le fichier CSV

<a id="j-m1"></a>
## M1 — Lancer l'API et la tester

- Version de Python : 3.14.7
- Commande de lancement que j'ai utilisée : …
- Code et `nb_lignes` renvoyés par `/sante` :
   {"statut":"ok","source":"csv","nb_lignes":110197}
- Différence entre `/docs` et `/sante` ouverts dans le navigateur : doc renvoi une interface, et santé renvoi un csv.
- Problème rencontré et solution (ou « aucun ») : …

<a id="j-m2"></a>
## M2 — Lire une route complète

### Le trajet d'une requête `GET /indicateurs`, en 4 étapes

1. Le navigateur envoie ──GET /indicateurs à l'adresse http://127.0.0.1:8000/
2. FastAPI exécute la fonction indicateurs() du fichier donnees_csv.py, qui appelle main.py
3. Cette fonction lit data/ventes_olist.csv et renvoie un dictionnaire python
4. Le modèle Indicateurs vérifie les champs et leurs types puis FastAPI renvoie un json avec le code 200

### L'expérience du contrat

- Quand j'ai mis `panier_moyen` en commentaire dans le modèle, la réponse  Le panier moyen n'est pas affiché .
- Conclusion : la ligne de la fonction existe mais n'est pas appelée

### Pourquoi deux fichiers (`main.py` et `donnees_csv.py`) ?

  - le fichier donnees_csv.py sert a questionner, et le main py sert a donner une rendu

### Comparaison avec lundi

- `nb_commandes` : 110197 (lundi : 96 478)
- `ca_total` : 15424463.65 (lundi : 15 424 463,65)

<a id="j-m3"></a>
## M3 — Ventes mensuelles

### Tests

- Sans paramètre : 23 mois, du 2016-09 au 2018-08
- `debut=2017-01` et `fin=2017-12` : 12 mois ; novembre 2017 = 1154559.53 R$ et 7289 commandes
- `debut=2018-02` et `fin=2017-01` : code … , message …
- `debut=2017-13` : code 422

### Questions

1. Différence entre un paramètre de chemin et un paramètre de requête, avec un exemple de cette API : Un parameètre de chemin fait partie de l'url, alors qu'un paramètre de requête filtre après le "?" 
   http://127.0.0.1:8000/ventes/mensuelles?debut=2018-02&fin=2017-01
   paramètre de chemin : /ventes/mensuelles
   paramètre de requête : debut=2018-02&fin=2017-01
2. Qui renvoie le code 400 (toi ou FastAPI) ? Et le code 422 ? Pourquoi cette différence ? le code 400 viens de moi, la requête est mal écrite. Le code 422 viens de FastAPI, le code est bien écrit mais FastAPI ne le comprend pas

<a id="j-m4"></a>
## M4 — Top catégories 

### Tests

- Sans paramètre : 10 catégories ; la première est health_beauty avec 9.16 %
- Somme des 10 parts : 62,39 %
- `n=3` : [{"categorie":"health_beauty","ca":1412271.15,"part_ca":9.16},{"categorie":"watches_gifts","ca":1264333.12,"part_ca":8.2},{"categorie":"bed_bath_table","ca":1225209.26,"part_ca":7.94}]
  
- `n=0` : code 422 Unprocessable Content · `n=51` : code 422 Unprocessable Content

### Question

- Que reçoit Rafael s'il n'envoie pas `n` ? Où est-ce décidé dans le code ? 
  Il recevera 10 valeurs car on demande un tableau de 10. 

def top_categories(n: int = Query(---->10<----, ge=1, le=50, description="Nombre de catégories (1 à 50)")):
    """Les n catégories au plus fort chiffre d'affaires."""

<a id="j-m5"></a>
## M5 — Chiffres d'un État

### Mon modèle `Etat` (copie le code)

```python
@app.get("/etats/{code_etat}", response_model=Etat, tags=["Indicateurs"])
def etat(code_etat: str = Path(pattern=r"^[A-Za-z]{2}$", description="Code de l'État sur 2 lettres, ex. SP")):
    """Chiffres d'un État brésilien."""
    code = code_etat.upper()          # « sp » et « SP » donnent le même résultat
    resultat = donnees.etat(code)
    if resultat is None:
        raise HTTPException(status_code=404, detail=f"Aucune vente pour l'État « {code} ».")
    return resultat
```

### Tests

- `/etats/SP` : CA 5771713.93 , commandes 40501 , part 37.42 %
- `/etats/sp` : identique (oui / non) : OUI
- `/etats/XX` : code 404 Not Found , message {"detail":"Aucune vente pour l'État « XX »."}
- `/etats/SPX` : code 422 Unprocessable Content

### Question

- Pourquoi un État sans vente renvoie 404 et pas une réponse vide avec le code 200 ? 
  Car nous avons mis une condition dans le "If" de "état" : 

```python 
   if resultat is None:
        raise HTTPException(status_code=404, detail=f"Aucune vente pour l'État « {code} ».")
```
    

<a id="j-m6"></a>
## M6 — Recette complète dans /docs

Pour chaque test : le code obtenu, puis une valeur lue dans la réponse.

1. `GET /sante` (attendu 200) : 127.0.0.1:58819 - "GET /sante HTTP/1.1" 200 OK
2. `GET /indicateurs` (attendu 200) : 127.0.0.1:49642 - "GET /indicateurs HTTP/1.1" 200 OK
3. `GET /ventes/mensuelles` sans paramètre (attendu 200, 23 mois) : 127.0.0.1:50060 - "GET /ventes/mensuelles HTTP/1.1" 200 OK
4. `GET /ventes/mensuelles?debut=2018-01` (attendu 200) : nombre de mois 8 (127.0.0.1:50955 - "GET /ventes/mensuelles?debut=2018-01 HTTP/1.1" 200 OK)
5. `GET /ventes/mensuelles?debut=2018-02&fin=2017-01` (attendu 400) : …
6. `GET /ventes/mensuelles?fin=nov` (attendu 422) : "GET /ventes/mensuelles?fin=nov HTTP/1.1" 422 Unprocessable Content
7. `GET /categories/top` (attendu 200, 10 catégories) : "GET /categories/top HTTP/1.1" 200 OK
8. `GET /categories/top?n=50` (attendu 200) : dernière catégorie de la liste "GET /categories/top?n=50 HTTP/1.1" 200 OK
9.  `GET /categories/top?n=0` (attendu 422) : "GET /categories/top?n=0 HTTP/1.1" 422 Unprocessable Content
10. `GET /etats/RJ` (attendu 200) : part du CA 13.33 %
11. `GET /etats/XX` (attendu 404) : "GET /etats/XX HTTP/1.1" 404 Not Found
12. `GET /etats/SPX` (attendu 422) : "GET /etats/SPX HTTP/1.1" 422 Unprocessable Content

### Bilan du J1

1. Ce qui fonctionne : …
2. Ce qui me reste à comprendre ou à terminer : …

---

# J2 — L'API sur PostgreSQL

<a id="j-m7"></a>
## M7 — Charger les ventes dans PostgreSQL

- Base utilisée : …
- Résultat du contrôle final du script : … lignes, … commandes, … de CA
- Droit donné à `lecteur_olist` sur la table, et pourquoi seulement celui-là : …

<a id="j-m8"></a>
## M8 — Mettre les secrets dans `.env`

- Résultat de la commande de test : …
- Rôle de `.env` : C'est le fichier de configuration de l'environement
- Rôle de `.env.exemple` : …
- Rôle de la ligne `.env` dans `.gitignore` : c'est le fichier qui permet d'exclure des fichier d'un repo github
- J'ai vérifié qu'aucun `.py` ne contient le mot de passe (oui / non) : oui

<a id="j-m9"></a>
## M9 — Réécrire les fonctions en SQL

### Résultat de `verifier_sources.py`

- Bilan affiché : 8 / 8
- Fonction qui m'a demandé le plus de corrections, et pourquoi : …

### Ma requête de `top_categories()` (copie-la)

```sql
        SELECT categorie,
               round(sum(montant), 2) AS ca,
               round(100 * sum(montant) / (SELECT sum(montant) FROM analytique.ventes), 2) AS part_ca
        FROM analytique.ventes
        GROUP BY categorie
        ORDER BY ca DESC
        LIMIT %(n)s
```

### Sécurité

- Pourquoi `%(code)s` plutôt qu'une f-string ? Donne l'exemple d'une valeur dangereuse. %(code)s permet d'eviter les injection SQL. Elle considere la valeur comme une variable seulement.
- Pourquoi faire calculer PostgreSQL plutôt que Python ? …

<a id="j-m10"></a>
## M10 — Basculer l'API sur PostgreSQL

- `/sante` après la bascule : source … , lignes …
- Pourquoi il faut relancer l'API après avoir modifié `.env` : …
- Test de panne (`PGPORT=5999`) : code … , message …
- Ce qu'aurait vu Rafael sans le gestionnaire d'erreur `SourceIndisponible` : …
- Ce que Rafael a dû changer dans son code pour la bascule : …

<a id="j-m11"></a>
## M11 — Écrire le client de Rafael

- Bilan affiché par `client.py` : … / 10
- Nombre de lignes de `ca_mensuel_2017.csv`, en-tête compris : …
- Différence entre `.status_code` et `.json()` : …
- Utilité du paramètre `timeout` : …

<a id="j-m12"></a>
## M12 — README et rendu

- Le test de remise à zéro a fonctionné en suivant seulement le README (oui / non) : …
- Ce que j'ai dû corriger dans le README après ce test : …
- Contenu de mon zip, fichier par fichier : …

### Mes choix techniques (3 à 5 lignes)

- Source de données retenue et pourquoi : …
- Compte de connexion et pourquoi : …
- Gestion des erreurs : …
- Ce que j'ajouterais pour mettre l'API en production : …

### Message à Mariana (3 lignes, sans jargon)

- Ce que Rafael peut faire maintenant : …
- D'où viennent les chiffres : …
- Le chiffre qui prouve que c'est juste : …
