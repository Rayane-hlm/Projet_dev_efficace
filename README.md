import queue

delimiteur = ','
stock = []
indiceDerniereAlerte = 0
alertes = ['','','']

def afficherMenu():
    """
    Description : Affiche le menu principal et récupère le choix de l'utilisateur
    Entrée : Aucune
    Sortie : Le choix de l'utilisateur (chaine de caractères)
    """
    #affichage du menu et choix utilisateur
    print("1: Ajouter des produits au stock")
    print("2: Afficher alerte la plus vieille")
    print("3: Afficher les stocks")
    print("4: Ajouter des produits dans un colis")
    print("Autre: Quitter")
    choix = input("Votre choix : ")
    return choix

#fonction pour gérer la boucle du menu
def menu():
    """
    Description : Lance la boucle principale du programme tant que l'utilisateur ne quitte pas
    Entrée : Aucune
    Sortie : Aucune
    """
    continuer = True
    while continuer == True:
        choix = afficherMenu()
        continuer = lancerMenu(choix)
    print("Au revoir !")

def lancerMenu(choix):
    """
    Description : Exécute l'action correspondant au choix de l'utilisateur
    Entrée : Le choix de l'utilisateur
    Sortie : Booléen (True pour continuer, False pour quitter)
    """
    #émulation d'une boucle do while
    #appel fonction en fonction du choix utilisateur
    if choix=="1":
        ajouterProduits()
    elif choix=="2":
        afficherAlerte()
    elif choix=="3":
        afficherStock()
    elif choix=="4":
        test = ajouterColis()
        afficherColis(test)
    else:
        return False
    return True

#fonction pour demander à l'utilsateur une chaine de caractères
def demanderChaineProduits():
    """
    Description : Demande une chaine de caractères à l'utilisateur et vérifie sa validité
    Entrée : Aucune
    Sortie : La chaine de caractères validée
    """
    chaine = input("Entrez une chaine de caractères : ")
    while not verifChaine(chaine):
        print("Chaine incorrecte")
        chaine = input("Entrez une chaine de caractères : ")
    return chaine

#fonction pour vérifier si le nombre d'alertes est supérieur à 3
#dans ce cas, on supprime les alertes en les réglant
def maxAlertes():
    """
    Description : Vérifie si le seuil d'alertes est atteint et réapprovisionne si nécessaire
    Entrée : Aucune
    Sortie : Aucune
    """
    global indiceDerniereAlerte
    if indiceDerniereAlerte == 3:
        for alerte in alertes:
            for nombre_produit in range(4):
                ajouterUnProduit(alerte)
        alerte = ''
        indiceDerniereAlerte = 0

#fonction pour ajouter une alerte dans la liste des alertes
def ajoutAlerte(typeProduit):
    """
    Description : Ajoute une alerte pour un type de produit manquant
    Entrée : Le type de produit concerné
    Sortie : Aucune
    """
    global indiceDerniereAlerte
    maxAlertes()
    alertes[indiceDerniereAlerte] = typeProduit
    indiceDerniereAlerte = indiceDerniereAlerte + 1

#fonction pour ajouter plusieurs produits dans le stock
def ajouterProduits(chaineProduits=''):
    """
    Description : Découpe une chaine et ajoute les produits correspondants au stock
    Entrée : Une chaine de produits (optionnelle)
    Sortie : Aucune
    """
    if chaineProduits == '':
        listeDecoupage = decouperChaine(demanderChaineProduits())
    else:
        listeDecoupage = decouperChaine(chaineProduits)
    for produit in listeDecoupage:
        ajouterUnProduit(produit)

#fonction pour rechercher si le type d'un produit existe dans le stock
def rechercherType(produit):
    """
    Description : Recherche l'index correspondant au type du produit dans le stock
    Entrée : Le produit recherché
    Sortie : L'index du type ou None si non trouvé
    """
    if len(stock) != 0:
        for indexType in range(len(stock)):
            if stock[indexType][0].queue[0][0] == produit[0]:
                return indexType
    return None

#fonction pour rechercher si le volume d'un produit existe dans le stock
def rechercherProduit(produit):
    """
    Description : Recherche les indices (type et volume) d'un produit dans le stock
    Entrée : Le produit recherché
    Sortie : Tuple (indexType, indexVolume) ou None
    """
    indexType = rechercherType(produit)
    if(indexType != None):
        for indexVolume in range(len(stock[indexType])):
            if stock[indexType][indexVolume].queue[0][1] == produit[1]:
                return indexType,indexVolume
        else:
            return indexType,None
    return None,None

#fonction pour ajouter un produit dans le stock
def ajouterUnProduit(produit):
    """
    Description : Ajoute un produit spécifique à l'emplacement approprié dans le stock
    Entrée : Le produit à ajouter
    Sortie : Aucune
    """
    index = rechercherProduit(produit)
    if index[0] != None:
        if index[1] == None:
            stock[index[0]].append(queue.Queue())
            stock[index[0]][-1].put(produit)
        else:
            stock[index[0]][index[1]].put(produit)
    else:
        stock.append([queue.Queue()])
        stock[-1][-1].put(produit)

#fonction pour afficher les alertes dans la file
def afficherAlerte():
    """
    Description : Affiche l'alerte la plus ancienne s'il y en a
    Entrée : Aucune
    Sortie : Aucune (affichage console)
    """
    if indiceDerniereAlerte == 0:
        print("Pas d'alertes")
    else:
        print(alertes[0])

#fonction pour afficher le stock de produit
def afficherStock():
    """
    Description : Parcourt et affiche tous les produits présents dans le stock
    Entrée : Aucune
    Sortie : Aucune (affichage console)
    """
    for liste_volume in stock:
        for file in liste_volume:
            for produit in file.queue:
                print(produit)

#fonction pour sortir les produits
def ajouterColis():
    """
    Description : Crée un colis et y ajoute des produits demandés retirés du stock
    Entrée : Aucune
    Sortie : Une pile (LifoQueue) représentant le colis
    """
    colis = queue.LifoQueue()
    listeDecoupage = decouperChaine(demanderChaineProduits())
    listeTemporaire = []
    for produit in listeDecoupage:
        produit = retirerUnProduit(produit)
        if produit != None:
            listeTemporaire.append(produit)
    if len(listeTemporaire) != 0:
        listeTemporaire.sort(key=lambda x: x[1], reverse=True)
        for produit in listeTemporaire:
            colis.put(produit)
    return colis

#fonction pour afficher le contenu du colis
def afficherColis(colis):
    """
    Description : Affiche le contenu d'un colis (pile) en le vidant
    Entrée : La pile (colis) à afficher
    Sortie : Aucune (affichage console)
    """
    while not colis.empty():
        print(colis.get())

#fonction pour retirer un produit du stock
def retirerUnProduit(produit):
    """
    Description : Retire un produit du stock et génère une alerte si le stock est bas
    Entrée : Le produit à retirer
    Sortie : Le produit retiré ou None si non trouvé
    """
    index = rechercherProduit(produit)
    if index[0] != None and index[1] != None:
        file = stock[index[0]][index[1]]
        produit = file.get()
        if (file.qsize() <= 4):
            ajoutAlerte(produit)
        if (file.empty()):
            stock[index[0]].remove(file)
        if (len(stock[index[0]]) == 0):
            stock.remove(stock[index[0]])
        return produit
    print("Produit",produit,"non trouvé")

#fonction pour découpe la chaine de caractères en liste de produits
def decouperChaine(chaine):
    """
    Description : Sépare une chaine de caractères selon le délimiteur défini
    Entrée : La chaine de caractères
    Sortie : Une liste de produits (chaines)
    """
    listeProduits = chaine.split(delimiteur)
    return listeProduits

#vérification de la chaine de caractères rentrée par l'utilisateur
def verifChaine(chaine):
    """
    Description : Vérifie si la chaine de caractères respecte le format attendu
    Entrée : La chaine à vérifier
    Sortie : Booléen (toujours True dans cette version)
    """
    if(True):
        return True
    else:
        return False

#fonction principale du code
def main():
    """
    Description : Point d'entrée principal pour initialiser le stock et lancer le menu
    Entrée : Aucune
    Sortie : Aucune
    """
    #Initialisation du stock par défaut (pour test)
    ajouterProduits("A1,A1,A1,A1,A1,B2,B2,B2,B2,B2,B2,C1,C1,C1,C1,C1")
    menu()

#appel de la fonction principale quand le bouton run est cliqué
if __name__ == '__main__':
    main()
