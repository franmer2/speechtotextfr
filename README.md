Dans cet article, je vais détailler les étapes pour transformer une discussion enregistrée dans un fichier audio en texte, enrichir cette transcription avec des analyses de sentiments, puis terminer par une représentation de cette discussion dans un rapport Power BI afin d'en faire une analyse.

Pour ce faire nous allons utiliser plusieurs services de la plateforme Azure dont principalement :

- Un compte de stockage Azure
- Le service cognitif "Speech to text"
- Azure Logic App
- Azure Cosmos DB

## Prerequis

- un abonnement Azure

# Création des services Azure
## Création d'un groupe de ressources
Nous allons commencer par créer un groupe de ressouces afin d'héberger les différents services de notre solution de transcription de fichiers audio.

Depuis le portail [Azure](https://portal.azure.com), cliquez sur "Create a resource"

![sparkle](Pictures/001.png)

 Puis, recherchez "Resource group"

 ![sparkle](Pictures/002.png)


Cliquez sur le bouton "Create"

![sparkle](Pictures/003.png)

Dans le champ "**Resource group**", donnez un nom à votre groupe de ressources. Cliquez sur le bouton "**Review + Create**"

![sparkle](Pictures/004.png)

Dans l'écran de validation, cliquez sur le bouton "**Create**"

![sparkle](Pictures/005.png)

Revenez à l'accueil du portail Azure. Cliquez sur le menu burger en haut à gauche puis sur "**Resource** **groups**"

![sparkle](Pictures/006.png)

Cliquez sur le groupe de ressources créé précédement

![sparkle](Pictures/007.png)

## Création du compte de stockage

Une fois dans le groupe de ressources, cliquez sur le bouton "**Add**" 

![sparkle](Pictures/008.png)

Recherchez le compte de stockage

![sparkle](Pictures/009.png)

Cliquez sur le bouton "Create"

![sparkle](Pictures/010.png)

Complétez la création du compte de stockage et cliquez sur "**Review** + **create**"

![sparkle](Pictures/011.png)

Après avoir vérifier les informations de création du compte de stockage, cliquz sur le bouton "Create"

![sparkle](Pictures/012.png)

## Creation du service Azure Logic App
Dans le groupe de ressources précedement créé, cliquez sur le bouton "**Add**"

![sparkle](Pictures/013.png)

Puis recherchez "**Logic App**"

![sparkle](Pictures/014.png)

Cliquez sur le bouton "**Create**"

![sparkle](Pictures/015.png)

Complétez l'étape de création du service "Logic App", puis cliquez sur "**Review + create**"

![sparkle](Pictures/016.png)

Dans la fenêtre de validation, cliquz sur le bouton "**Create**"

![sparkle](Pictures/017.png)

## Création du service cognitif "Speech to text"
Dans le groupe de ressources précedement créé, cliquez sur le bouton "**Add**"

![sparkle](Pictures/018.png)

Recherchez le service "**Speech**"

![sparkle](Pictures/019.png)

Cliquez sur le bouton "Create"

![sparkle](Pictures/020.png)

Complétez le formulaire de création, puis cliquez sur le bouton "Create"

![sparkle](Pictures/021.png)

## Création du service Azure Cosmos DB

Dans le groupe de ressources précedement créé, cliquez sur le bouton "**Add**"

![sparkle](Pictures/022.png)

Recherchez "**Azure Cosmos DB**"

![sparkle](Pictures/023.png)

Cliquez sur le bouton "**Create**"

![sparkle](Pictures/024.png)

Remplissez le formulaire de dréation du service. Choisissez l'API "**Core (SQL)**".
Cliquez sur le bouton "**Review + create**"

![sparkle](Pictures/025.png)

Après avoir validé les informations, cliquez sur le bouton "**Create**"

![sparkle](Pictures/026.png)