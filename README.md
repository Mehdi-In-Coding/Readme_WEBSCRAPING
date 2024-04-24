Bastien E. - Mehdi B.  
25/01/2024


#  Projet Automatisation et Collecte des produits du site ACTION 

🕸️ Webscraping Projet
Ce script Python réalise le webscraping d'informations produits à partir du site web de l'entreprise ACTION  accès depuis le site : https://www.action.com/. Explorez différentes catégories, extrayez les données et enregistrez les informations individuelles de chaque produit en fichiers JSON.

# 1ere partie - Recupération des données


## 🛠️ Prérequis

Assurons-nous d'avoir importer les bibliothèques suivantes avant d'exécuter le script :


```python
import requests as rq
from bs4 import BeautifulSoup
import json
import os
import time 
```

- requests: Effectue des requêtes HTTP pour récupérer des données depuis des URLs.
- BeautifulSoup: Facilite le web scraping en extrayant des informations à partir de pages HTML ou XML.
- json: Manipule des données au format JSON.
- os: Fournit des fonctionnalités pour interagir avec le système d'exploitation, notamment pour la manipulation de fichiers et de répertoires.
- time: Gère les aspects liés au temps, utilisé ici pour introduire des pauses entre les requêtes HTTP.

## 🧱 Structure du Code
### Importations

Utilisation de requests pour effectuer des requêtes HTTP.
Utilisation de BeautifulSoup pour le web scraping.
Utilisation de json, os, et time.
Fonction get_token


```python
url_base = "https://www.action.com/fr-fr/"
user_agent = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36"}
```

- Initialiser "url_base" avec l'URL de base du site web de l'entreprise ACTION
- Initialiser "user_agent" pour éviter les blocages dus à des requêtes non autorisées
et pour fournir des informations sur le client au serveur web.



Définition de l'URL de base et de l'en-tête utilisateur.
Boucle Principale sur les Catégories et les Pages

Obtient le token nécessaire pour les requêtes.
Variables Initiales
```python
def get_token(url):
    reponse = rq.get(url, headers = user_agent)
    html_code = reponse.text
    soup = BeautifulSoup(html_code, 'html.parser')
    meta_tag = soup.find('meta', {'name': 'build-id'})
    build_id = meta_tag['content']
    return build_id
```

Le get_token prend une URL en paramètre. Cette fonction envoie une requête HTTP à l'URL spécifiée, récupère le contenu HTML de la réponse, puis utilise la bibliothèque BeautifulSoup pour analyser le HTML. Et build-id pour d'autre requêtes.


```python
url_produit = []
dossier = "C:/Users/mbena/OneDrive/Bureau/python TP/Webscraping_Projet"
os.makedirs(dossier, exist_ok=True)

categories = ['habitat',
            'cuisine',
            'articles-menagers',
            'papeterie--bureau',
            'hobby',
            'bricolage',
            'jouets',
            'jardin',
            'voyages',
            'hygiene--beaute',
            'boissons--alimentation',
            'multimedia',
            'mode',
            'articles-de-sport',
            'animaux-domestiques']
```


Création des catégories pour le classement des articles depuis le site d'ACTION.

Dans notre projet, on se concentrera que sur la catégorie "Habitat".


- Requêtes HTTP pour Obtenir les Informations Produits


- Parcourir les différentes catégories de produits à travers une boucle sur les pages de chaque catégorie, effectue des requêtes HTTP pour chaque page, extrait les informations des produits, puis écrit ces informations dans des fichiers JSON correspondants.

```python
for categorie in categories :
    print(f'categorie : {categorie}')
    for page in range(1, 2):
        #payload = {'locale': 'fr-fr', 'path': ['c', 'habitat'], 'page': page}
        payload = {'locale': 'fr-fr', 'path': ['c', categorie], 'page': page}
        #reponse = rq.get(f'https://www.action.com/_next/data/{get_token(url_base)}/fr-fr/c/habitat.json', params=payload, headers = user_agent)
        reponse = rq.get(f'https://www.action.com/_next/data/{get_token(url_base)}/fr-fr/c/{categorie}.json', params=payload, headers = user_agent)
        print(f'page {page}')
        page_reponse_dict = reponse.json()
        pageProps_dict = page_reponse_dict["pageProps"]
        try:
            initialState_dict = pageProps_dict["initialState"]    
        except : 
            break
        produit_list = [cle for cle in initialState_dict if cle.startswith("Product:")]
        info_produit_list = [initialState_dict.get(produit) for produit in produit_list]
        url_produit = [dict["href"] for dict in info_produit_list] 
        id_produit = [dict["id"] for dict in info_produit_list]
```


Enregistrement des informations des produits dans des fichiers JSON.
Pause en Cas de Redirection

```python
 for url_produit, id_produit in zip(url_produit, id_produit):
            code_name = url_produit.replace(url_base, "")
            code_name = code_name.replace("p/", "")
            code_name = code_name.split("/")
            if len(code_name) == 3:
                code_name.pop(2)
            reponse_produit = rq.get(f'https://www.action.com/_next/data/{get_token(url_base)}/fr-fr/p/{code_name[0]}/{code_name[1]}.json', params=payload, headers = user_agent)
            reponse_produit_json = reponse_produit.json()
            pageProps_dict2 = reponse_produit_json["pageProps"]

 if pageProps_dict2.get("__N_REDIRECT_STATUS") is not None :
                time.sleep(10)
                break

 initialState_dict2 = pageProps_dict2["initialState"] 
            product = initialState_dict2[f'Product:{id_produit}']                
            fichier_produit = os.path.join(dossier, f'{code_name[1]}.json')
            json_data = json.dumps(product)
            with open(fichier_produit, "w") as fichier_json:
                fichier_json.write(json_data)
```

Le code vérifie s'il y a une redirection avec la clé "__N_REDIRECT_STATUS". En cas de redirection, il fait une pause de 10 secondes et sort de la boucle. Si aucune redirection n'est détectée, il extrait les informations du produit, les écrit dans un fichier JSON et passe au produit suivant.
Le code, pour chaque article, compile ses informations (description, libellé, id et prix) au format JSON dans le dossier déjà crée. Ces informations sont extraites, puis le code crée un fichier JSON pour chaque article dans le dossier spécifié.


#    2me partie - Gestion des données

## 🛠️ Prérequis

Pour le deuxième jeu de données, on importe les bibliothèques suivantes avant d'exécuter le script :


```python
import pandas as pd
import glob
import json
import matplotlib.pyplot as plt
```
- pandas: Manipule des données tabulaires, souvent utilisé pour le traitement de données.
- glob: Recherche des noms de fichiers ou des chemins d'accès en utilisant des motifs avec des caractères génériques.
- json: Manipule des données au format JSON.
- matplotlib.pyplot: Crée des visualisations, généralement des graphiques, à partir des données.

## 📊💻 Structure du Code
### Importations

```python
def json_to_dataframe(file_path):
    with open(file_path, 'r') as f:
        data = json.load(f)
    df = pd.json_normalize(data)
    return df
```

- Récuperation des données des articles et de la gestion dans le dictionnaire df pour chaque article ensuite(id, code, titre, description, catégorie, image, le prix etc.. )

- Définition du chemin vers un dossier contenant des fichiers JSON de produits, puis utilise la bibliothèque glob pour créer une liste de chemins vers tous les fichiers JSON dans ce dossier.

```python
dossier_produits = "C:/Users/mbena/OneDrive/Bureau/python TP/Webscraping_Projet"

fichiers_json = glob.glob(f"{dossier_produits}/*.json")
```
- Création d'un DataFrame vide avec Pandas, puis itère sur une liste de chemins de fichiers JSON.À chaque itération, il convertit le contenu JSON de chaque fichier en DataFrame et concatène ces DataFrames avec le DataFrame global combined_df. Enfin, il affiche les premières lignes du DataFrame résultant.

```python
combined_df = pd.DataFrame()

for fichier in fichiers_json:
    df = json_to_dataframe(fichier)
    combined_df = pd.concat([combined_df, df], ignore_index=True)

print(combined_df.head())
```

## 📈 Graphiques

### Boîte à moustache pour price.current.amount et price.original.amount
```python
plt.figure(figsize=(10, 6))
combined_df.boxplot(column=['price.current.amount', 'price.original.amount'], vert=False)
plt.title('Comparaison des prix actuels et originaux pour les produits AUCHAN')
plt.xlabel('Prix (en unité de votre devise)')
plt.show()
```

### Histogramme pour price.original.amount
```python
plt.figure(figsize=(10, 6))
plt.hist(combined_df['price.current.amount'], bins=20, facecolor='lightblue', edgecolor='darkblue')
plt.title('Histogramme des prix des articles')
plt.xlabel('Prix (en euros)')
plt.ylabel('Fréquence')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.show()
```

### Histogramme pour price.current.amount
```python
plt.figure(figsize=(10, 6))
plt.hist(combined_df['price.original.amount'], bins=20, facecolor='lightcoral', edgecolor='darkblue')
plt.title('Histogramme des prix des articles en réduction')
plt.xlabel('Prix (en euros)')
plt.ylabel('Fréquence')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.show()
```
