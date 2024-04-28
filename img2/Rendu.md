# WebScrapting
Eva BERTRAND, Théo DUCHATEAU, Suzanne MARTIN-WITKOWSKI
#
# Explication du Projet :
## Contextualisation :
Le but global de ce projet est d'automatiser la collecte des données d'un site,  ici il s'agit du site d'Action. Pour y parvenir nous avons suivit différentes étapes : 

1. Exploration du site web
2. Utiliser l’API
3. Structurer le code pour itérer sur toutes les pages du site web
4. Construire le DataFrame des produits

#
## Explication des étapes :

L'étape d'exploration du site a pour but de faire comprendre comment marche le site: son architecture, comment sont cataloguer les produits, et, quel-est le chemin à suivre pour naviguer dans le site. Cette étape nous permet aussi d'analyser les différentes requêtes effectuées via la console développeur.

La seconde étape nous indique qu'il faut utiliser l'API. Pour cela il faut d'abord comprendre comment elle fonctionne et de la requêter.
Ensuite nous devons construire l'URL de l'API afin de requêter une page du site, puis faire la même chose mais pour un produit spécifique.

Dans cette étape, nous devons structurer le code afin de collecter tous les produits de toutes les pages de toutes les catégories. Pour cela nous devons créer une fonction collectant les informations d'un produit. Cette fonction doit récupèrer le JSON depuis l’API et écrire le résultat dans un fichier ".json". Ensuite, faire une fonction récupérant tous les produits d'une page. Puis, nous devons faire une fonction permettant de récupérer les pages de toutes les catégories: collectant toutes les informations de tous les produits de toutes les pages.

Cette dernière étape nous demande de mettre les réponses des produits dans un dataframe pandas. Il nous faut donc rédiger une fonction transformant nos JSON en un dataframe pandas, dans lequel chaque ligne représentera le détail d’un produit.

#
## Explication du code :

Pour mener à bien le processus de collecte des données, nous avons importer 4 modules : request, re, json et pandas(pd).

    ETAPE 1 : RECONSTITUTION DE L'URL
    common_headers = {"User-Agent" : "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36"}

    # Création d'une fonction pour récupérer le token
    def get_token():
        site = 'https://www.action.com'
    
        rep = requests.get(site, headers = common_headers)
        content = rep.content
    
        content_str = content.decode('utf-8')
        match = re.search(r'<meta name="build-id" content="([^"]+)"', content_str)
        
        build_id = match.group(1)
        return build_id

    url = "https://www.action.com/_next/data/"+get_token()+"/fr-fr/c/bricolage.json"
    response = requests.get(url, headers = common_headers)
    response_json = response.json()

Dans cette première étape, nous avons pour objectif de récupérer un token à partir d'un site web (Action), puis de l'utilliser pour construire une URL afin de faire une requête ultérieure pour obtenir des données JSON. 

Nous avons d'abord initialisé un dictionnaire 'common_headers' contenant les informations du User-Agent. L'User-Agent permet d'identifier l'utilisateur lors de la requête.

La fonction 'get_token()' effectue ces étapes suivantes :
- Envoie une requête au site d'Action ('https://www.action.com') avec les en-têtes communs.
- Récupère le contenu HTML de la réponse.
- Décodage du contenu en chaîne de caractères en utilisant l'encodage UTF-8.
- Recherche une balise avec l'attribut 'name' défini sur "build-id" et récupère la valeur de l'attribut 'content' correspondante. Cette valeur est supposée être le token.
- Retourne le token extrait.

La ligne suivante construit une URL en ajoutant le token obtenu précédement à l'URL de base. L'URL résultante est utilisée pour récupérer des données JSON liées à la page "bricolage" à partir du site internet.

Les dernières lignes envoient une requête
à l'URL construite en utilisant les en-têtes communs et récupère la réponse JSON. Les données JSON sont ensuite analysées en un objet Python à l'aide de la méthode json().
#

    ETAPE 2 : RECUPERATION DES INFORMATIONS PRODUITS DE BASE

    def get_products_categ(categ):
        url = "https://www.action.com/_next/data/"+get_token()+"/fr-fr/c/"+categ+".json"
        dict_page = {}
        dict_products = {}
        page_number = 1
        
        while True:
            r = requests.get(url, headers = common_headers, params = {'page' : page_number})
        
            try:
                dict_page[f'Page {page_number}'] = r.json()
                dict_products[f'Page {page_number}'] = {}
                for k in dict_page[f'Page {page_number}']['pageProps']['initialState']:
                    if k.startswith('Product:'):
                        dict_products[f'Page {page_number}'][k] = dict_page[f'Page {page_number}']['pageProps']['initialState'][k]
            except KeyError:
                break
            
            page_number += 1
            
        del(dict_products[f"Page {page_number}"])
        
        return dict_products

    dict_products_jouets = get_products_categ('jouets')

Ici, nous cherchons à récupérer les informations sur les produits d'une catégorie choisie à partir d'un site web (Action).

**Fonction get_products_categ(categ) :**
Cette fonction prend un argument categ, qui représente la catégorie des produits que l'on souhaîte.
- La première ligne construit une URL en utilisant la valeur de 'categ' et le token.
- On initialise un dictionnaire vide 'dict_page' qui stockera les données JSON brutes récupérées pour chaque page.
- On initialise un dictionnaire vide 'dict_products', il sera utilisé pour stocker les informations des produits extraites des données JSON.
- 'page_number' est initialisé à 1, représentant la première page de résultats.

**Boucle while True :**
Cette boucle s'exécute indéfiniment jusqu'à ce qu'une exception (KeyError) se produise.
À chaque itération, une requête est effectuée à l'URL construite avec le numéro de page actuel.
Les données JSON de la page sont ensuite stockées dans 'dict_page' sous la clé correspondante à la page actuelle.
Ensuite, la boucle parcourt les données JSON pour extraire des informations des produits et les stocke dans dict_products sous la clé correspondante à la page actuelle.

**Gestion d'exception KeyError :**
Si une exception se produit, cela signifie qu'il n'y a plus de pages à récupérer, et la boucle se termine.

Nettoyage de données :
La dernière page, qui a provoqué l'exception, est supprimée de 'dict_products'.

**Renvoi des données :**
La fonction renvoie le dictionnaire 'dict_products', contenant les informations des produits de toutes les pages récupérées pour la catégorie choisie.

#

    ETAPE 3 : RECUPERATION DES INFORMATIONS PRODUITS COMPLETES (Moitié des pages de la catégorie JOUETS)

    prod_dico_jouets = {}
    prod_jouets_comp = {}
    i = 1

    for page in range(1, (len(dict_products_jouets)+1)//2):
    
        for product_dict in dict_products_jouets[f"Page {page}"]:
        
            # Création de 3 variables pour constituer url : code, name, id
            code = dict_products_jouets[f"Page {page}"][product_dict]['code']
            ID = dict_products_jouets[f"Page {page}"][product_dict]['id']
            ref = dict_products_jouets[f"Page {page}"][product_dict]['href']
            name_list = ref.split('/')
            name = name_list[-2]
        
            # Récuperer un dictionnaire de données de la page de chaque produit
            url_prod = f"https://www.action.com/_next/data/{get_token()}/fr-fr/p/{code}/{name}.json"
            product_response = requests.get(url_prod, headers = common_headers)
            prod_dico = product_response.json()
        
            # Naviguer jusqu'à Product:<id>
            prod_dico_jouets = prod_dico['pageProps']['initialState'][f"Product:{ID}"]
        
            # Créer un fichier json a partir de prod_dico_p1
            json_file_path = f'data\{name}({ID}).json'
            with open(json_file_path, 'w') as json_file:
                json.dump(prod_dico_jouets, json_file)
            
            # Ajouter les informations du produit au dictionnaire complet
            prod_jouets_comp[i] = prod_dico_jouets
            i+=1

Mantenant, nous souhaitons récupérer les données sur le site d'Action pour collecter des informations sur une catégorie choisie (Jouets).

**Initialisation des variables :**
prod_dico_jouets: dictionnaire vide utilisé pour stocker les informations détaillées de chaque produit à partir des pages.
prod_jouets_comp: dictionnaire vide servant à contenir les informations complètes de tous les produits.

**Boucle principale pour parcourir les pages :**
La boucle itère sur les pages de produits. La division par 2 assure que la boucle ne parcourt que la moitié des pages.

**Extraction des informations de base du produit :**
Extraction du code, de l'ID, de la référence du produit et du nom du produit à partir du dictionnaire de produits.

**Construction de l'URL du produit et récupération des informations détaillées :**
Construction de l'URL du produit en utilisant les informations extraites et récupération des données à partir de l'URL.

**Création d'un fichier JSON pour chaque produit :**
Création d'un fichier JSON pour chaque produit avec le nom du produit et l'ID dans le chemin du fichier.

**Ajout des informations du produit au dictionnaire complet :**
Ajout des informations du produit au dictionnaire avec une clé (i) et incrémentation de i pour le prochain produit.

#
    ETAPE 4 : CREATION D'UN DATAFRAME AVEC LES INFORMATIONS DES PRODUITS
    import pandas as pd

    # Convertir le dictionnaire en DataFrame
    df = pd.DataFrame.from_dict(prod_jouets_comp, orient='index')

    # Appliquer la fonction lambda pour extraire les colonnes des dictionnaires imbriqués
    df = pd.json_normalize(df.to_dict('records'))

    # Spécifier le chemin pour le fichier Excel
    excel_file_path = 'output.xlsx'

    # Enregistrer le DataFrame au format Excel
    df.to_excel(excel_file_path, index=False)

    print(f"Données exportées avec succès vers {excel_file_path}")

Cette dernière étape nous voulons organiser dans un DataFrame les données extraîtes précédement, normaliser les données pour traiter les structures, puis exporter dans un fichier Excel.

**Conversion du dictionnaire en DataFrame :**
Nous commençons d'abord par créer un DataFrame pandas à partir du dictionnaire 'prod_jouets_comp'. Les clés du dictionnaire sont utilisées comme index du DataFrame.

**Extraction des colonnes des dictionnaires imbriqués :**
Après avoir créé le DataFrame, la méthode to_dict('records') est utilisée pour convertir le DataFrame en une liste de dictionnaires. Ensuite, la fonction 'pd.json_normalize' est appliquée pour déplier les colonnes qui contiennent des dictionnaires imbriqués, permettant ainsi une représentation plus clair des données.

**Spécification du chemin du fichier Excel de sortie :**
Le chemin du fichier Excel dans lequel les données seront exportées est spécifié. Dans cet exemple, le fichier sera nommé "output.xlsx".

**Enregistrement du DataFrame au format Excel :**
Le DataFrame résultant est enregistré dans un fichier Excel spécifié par 'excel_file_path'. L'argument index=False indique de ne pas inclure les index du DataFrame dans le fichier Excel.

**Impression d'un message de succès :**
Enfin, un message est affiché pour indiquer que le processus d'exportation des données vers le fichier Excel a été effectué avec succès. Ce message inclut le chemin complet du fichier Excel.

#
## Analyses Statistiques :
![Alt text](image.png)

Ce premier graphique nous présente le nombre de produits des 5 marques les plus représentées.La marque la plus présente sur le site d'Action est 'Playmobil' avec plus de 30 produits. Ensuite la marque 'Disney' compte une vingtaine de produits et la marque 'Mini Matters' compte 15 produits sous sous nom. Les deux marques suivantes : 'Zuru' et 'Hasbro' ont environ 14 produits sur le site internet. 

#
![Alt text](image-1.png)

Ce graphique nous présente le nombre de produits des 5 sous-catégories les plus représentées. Nous pouvons d'abor remarquer que la sous-catégorie 'Jeux' compte 300 produits. Ensuite, la sous-catégorie "Peluches et poupées" a entre 50 et 100 produits. Les catégories 'Véhicules et voitures', 'Jouets pour bébés' et 'Figurines et sets' ont chacune moins de 50 produits.
#
![Alt text](image-2.png)

Ce diagramme en barre nous montre la distribution des prix. Nous remarquons dans un premier temps que beaucoups de produits ont un prix peu élevés, la majorité des produits ont un prix se situant entre 0 et 22€. Toutefois, il y a quelques exceptions, certains produits ont dew prix plus élevés: environ 30, 50 ou encore 80€.
#
![Alt text](image-3.png)

Ici, ce graphique nous montre la moyenne des prix des 5 marques les plus chères. La marque 'Fur Real' a la moyenne de prix la plus élevée avec une moyenne de 20,0€. Les marques 'Fisher Price' et 'Ecoiffier' ont respectivement une moyenne de prix de d'environ 17,5€. Les deux marques 'Mega Blocks' et 'Smoby' ont chacune une moyenne de 15,0€.
#
![Alt text](image-4.png)

Ce graphique nous présente la moyenne des prix des 5 sous-catégories les plus chères. Les sous-catégories 'Bricolage' et 'Véhicules et voitures' ont toute les deux une moyenne de prix supérieure à 8,0€. Ensuite, la sous-catégorie 'Jouets en bois' a une moyenne de 8,0€. Enfin, nous pouvons remarquer que les marques 'Jouets pour bébés' et 'Articles de sport' ont un prix moyen se situant à environ 7,0€.
#
![Alt text](image-5.png)

Ce diagramme circulaire donne la répartition des disponibilité des produits. On peut voir que 85,8% des produite sont disponibles sur le site internet d'Action. Ainsi, réciproquement, 14,2% des produits sont indisponibles.
#
![Alt text](image-6.png)

Sur ce graphique en barre, nous avons les 5 marques les plus demandées, par les indisponibilité de produits. Nous pouvons remarquer dans un premier temps que la marque 'MGA' a 100% de ses produits indisponibles. Après nous pouvons voir que les marques 'LEGO' et 'Dolly Star' ont 50% des leurs produits qui sont indisponibles sur le site Action. La marque 'Hasbro' voit environ 45% des ses produits indisponibles. Et, la cinquième marque 'Playmobil' a environ 38% de produits indisponibles.
#
![Alt text](image-7.png)

Sur ce dernier graphique, nous avons les 5 sous-catégories les plus demandées, par indisponibilité de produits. D'abord, nous pouvons dire que ces 5 sous-catégorie ont un pourcentage de produits indisponibles supérieur à 10%. Plus précisément, c'est la sous-catégorie "Puzzles" ayant le plus de demandes, avec plus de 20% de produits indisponibles. Ensuite, on remarque que les sous-catégories 'Figurines et sets' ,'Jeux' et 'Véhicules et voitures' ont toutes les 3 environ 15% de produits indisponibles. Enfin la sous-catégorie 'Jouet en bois' a environ 13% de produits indisponibles.
#