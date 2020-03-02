Dans cet article, je vais détailler les étapes pour transformer une discussion enregistrée dans un fichier audio en texte, enrichir cette transcription avec des analyses de sentiments, puis terminer par une représentation de cette discussion dans un rapport Power BI afin d'en faire une analyse.

Pour ce faire nous allons utiliser plusieurs services de la plateforme Azure dont principalement :

- Un compte de stockage Azure
- Le service cognitif "Speech to text"
- Azure Logic App
- Azure Cosmos DB

## Prerequis

- Un abonnement Azure

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

Après avoir vérifié les informations de création du compte de stockage, cliquz sur le bouton "Create"

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

Complétez le formulaire de création, puis cliquez sur le bouton "**Create**"

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

# Préparation des services
## Compte de stockage
Nous allons maintenant créer un container afin de recevoir les fichiers audio

Depuis votre groupe de ressources, cliquez sur le service de compte de stockage

![sparkle](Pictures/027.png)

Cliquez sur "**Containers**"

![sparkle](Pictures/028.png)

Cliquez ensuite sur le bouton "**+ Container**". Donnez un nom à votre container puis cliquez sur le bouton "**Ok**".

![sparkle](Pictures/029.png)

Le nouveau container doit apparaître dans la liste comme illustré ci-dessous

![sparkle](Pictures/030.png)

## Création d'une collection Azure Cosmos DB

Nous allons maintenant créer une collection afin de recevoir la transciption au format texte du fichier audio

Depuis votre groupe de ressources, cliquez sur le service Azure Cosmos DB

![sparkle](Pictures/031.png)

Vérifiez de bien être dans la partie "**Overview**" du service, puis cliquez sur "**+ Add Container**"

![sparkle](Pictures/032.png)

Dans la section "Add Container", sélectionnez "Create new" pour créer une nouvelle base. Pour cet article, nous allons choisir "**Manual**" pour les "Throughput" et définir la valeur à **400**.

Enfin, donnez un nom à votre collection et choisissez une clef de partionnement. Ici je choisis comme clef la date, car concernant mom projet, on aura de nombreux fichiers par jour et il faudra faire des rapports sur une plage detemps vairant de la semaine au mois.

![sparkle](Pictures/033.png)

Après la création du container vous devriez obtenir un résultat similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/034.png)

## Configuration du service Azure Logic Apps

Pour transcrire les fichiers audio qui seront déposés dans le compte de stockage, nous allons utiliser l'API REST de transcription par lot :

https://docs.microsoft.com/fr-fr/azure/cognitive-services/speech-service/batch-transcription

Les étapes ci-dessous vont permettre de configurer le service Azure Logic App pour appeler l'API REST du servive cognitif "Speech" dès qu'un fichier audio arrivera dans le container.

Depuis votre groupe de ressources, cliquez sur le service Azure Logic Apps

![sparkle](Pictures/035.png)

Cliquez ensuite sur "Blank Logic App"

![sparkle](Pictures/036.png)

Vous allez 6etre ensuite redirigez vers l'éditeur d'Azure Logic Apps

![sparkle](Pictures/037.png)

Dans la zone de recherche, entrez "**Blob**", puis sélectionnez "**When a blob is added or modified**"

![sparkle](Pictures/038.png)

Donnez un nom à votre chaine de connexion puis sélectionnez le compte de stockage créer au début de cet article

![sparkle](Pictures/039.png)

Dans le champ "**Container**", cliquez sur l'icône à droite (1) et sélectionnez le container créé précédement dans cet article. Définissez 5 secondes d'intervale entre chaque vérification de la présence de nouveaux fichiers dans le container
 
![sparkle](Pictures/040.png)

Cliquez sur "**+ New step**" afin de créer une action supplémentaire

![sparkle](Pictures/041.png)

Dans la zone de recherche, entrez le mot "JSON" puis cliquez sur "Data Operations"

![sparkle](Pictures/042.png)

Sélectionnez "**Compose**"

![sparkle](Pictures/043.png)

Dans le champ input, entrez la partie JSON ci-dessous, sachant que nous allons ensuite compléter la partie en rouge

{
  "description": "La description est optionnelle",
  "locale": "en-US",
  "name": "transcription-batch",
  "properties": {
    "AddSentiment": "True",
    "AddWordLevelTimestamps": "True",
    "ProfanityFilterMode": "Masked",
    "PunctuationMode": "DictatedAndAutomatic"
  },
  "recordingsUrl": 
}

![sparkle](Pictures/044.png)