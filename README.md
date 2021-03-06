Dans cet article, je vais détailler les étapes pour transformer une discussion enregistrée dans un fichier audio en texte, enrichir cette transcription avec des analyses de sentiments, puis terminer par une représentation de cette discussion dans un rapport Power BI afin d'en faire une analyse.

Pour ce faire nous allons utiliser plusieurs services de la plateforme Azure dont principalement :

- Un compte de stockage Azure
- Le service cognitif "Speech to text"
- Azure Logic App
- Azure Cosmos DB

Ci-dessous un schéma de la solution que nous allons mettre en place dans Azure

![sparkle](Pictures/901.png)

La solution peut aussi se déployer rapidement dans votre souscription Azure via le bouton ci-dessous :


[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Ffranmer2%2Fspeechtotextfr%2Fmaster%2FARM%2520Files%2FTemplate.json)

Pour tester la solution, vous pouvez utiliser les [fichiers audio](https://github.com/franmer2/speechtotextfr/tree/master/AudioFile) de test et les déposer dans le container "**audiofiles**" créé lors du déployment de la solution. 

## Prerequis

- Un abonnement Azure
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) 
  

# Création des services Azure
## Création d'un groupe de ressources
Nous allons commencer par créer un groupe de ressouces afin d'héberger les différents services de notre solution de transcription de fichiers audio.

Depuis le portail [Azure](https://portal.azure.com), cliquez sur "**Create a resource**"

![sparkle](Pictures/001.png)

 Puis, recherchez "**Resource group**"

 ![sparkle](Pictures/002.png)


Cliquez sur le bouton "**Create**"

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

Cliquez sur le bouton "**Create**"

![sparkle](Pictures/010.png)



Complétez la création du compte de stockage et cliquez sur "**Review** + **create**"

![sparkle](Pictures/011.png)

Après avoir vérifié les informations de création du compte de stockage, cliquez sur le bouton "**Create**"

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

Cliquez sur le bouton "**Create**"

![sparkle](Pictures/020.png)

Complétez le formulaire de création, puis cliquez sur le bouton "**Create**"

Dans le champ "**Pricing Tier**" sélectionnez "**S0**", car seul ce tier supporte la transcription par lot :

https://docs.microsoft.com/fr-fr/azure/cognitive-services/speech-service/batch-transcription#prerequisites


![sparkle](Pictures/021.png)

Une fois le service créé, copiez une des clefs dans un endroit facile d'accès car nous allons en avoir besoin plus tard. 

Les clefs se trouvents dams la partie "**Keys and Endpoint**" de votre service cognitif

![sparkle](Pictures/903.png)

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

Au final, vous devriez avoir 4 services de créés dans votre groupe de ressources, comme illustré ci-dessous :
![sparkle](Pictures/908.png)

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

### Création de la clef d'accès partagée (SAS Token)

Au niveau de votre compte de sotckage, cliquez sur "**Shared access signature**"

![sparkle](Pictures/904.png)

Définissez votre clef en choissant au minimum :

- Les services
- Le type de ressouce
- Les permissions
- La date d'expiration

Dans notre exemple nous allons cochez les cases :

- "**Blob**"
- "**Object**" (Très important)
- "**Read**"
- Et choisir une date d'expiration

Ci-dessous un exemple

![sparkle](Pictures/905.png)

Après avoir cliqué sur le bouton "**Generate SAS and connection string**", vous pouvez copiez votre clef. Nous allons en avoir besoin un peu plus tard.

![sparkle](Pictures/906.png)



## Création d'une collection Azure Cosmos DB

Nous allons maintenant créer une collection afin de recevoir la transciption au format texte du fichier audio

Depuis votre groupe de ressources, cliquez sur le service Azure Cosmos DB

![sparkle](Pictures/031.png)

Vérifiez de bien être dans la partie "**Overview**" du service, puis cliquez sur "**+ Add Container**"

![sparkle](Pictures/032.png)

Dans la section "Add Container", sélectionnez "**Create new**" pour créer une nouvelle base. Pour cet article, nous allons choisir "**Manual**" pour les "Throughput" et définir la valeur à **400**.

Enfin, donnez un nom à votre collection et choisissez une clef de partionnement. Ici je choisis comme clef la date, car concernant mom projet, on aura de nombreux fichiers par jour et il faudra faire des rapports sur une plage de temps variant de la semaine au mois.

![sparkle](Pictures/033.png)

Après la création du container vous devriez obtenir un résultat similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/034.png)

## Configuration du service Azure Logic Apps

Pour transcrire les fichiers audio qui seront déposés dans le compte de stockage, nous allons utiliser l'API REST de transcription par lot :

https://docs.microsoft.com/fr-fr/azure/cognitive-services/speech-service/batch-transcription

Les étapes ci-dessous vont permettre de configurer le service Azure Logic App pour appeler l'API REST du servive cognitif "Speech" dès qu'un fichier audio arrivera dans le container.

Depuis votre groupe de ressources, cliquez sur le service Azure Logic Apps

![sparkle](Pictures/035.png)

Cliquez ensuite sur "**Blank Logic App**"

![sparkle](Pictures/036.png)

Vous allez être redirigez vers l'éditeur d'Azure Logic Apps

![sparkle](Pictures/037.png)

### Création des paramètres

On va commencer par créer les différents paramètres dont nous allons avoir besoin. De plus ces paramètres pourront être accessibles dans les templates ARM.

Cliquez sur le bouton "**Parameters**"

![sparkle](Pictures/200.png)

Puis sur le bouton "**Add parameters**

![sparkle](Pictures/907.png)

"Rajoutez les paramètres suivants :

- cognitiveSecret (une des clefs du service cognitif)
- azureRegion (le nom de la région Azure (ici "**canadacentral**"))
- storageAccountName (le nom du compte de stockage crée précédement (ici "**audiofiledemo**"))
- SASToken (la signature d'accès partagée crée précédement)

Complétez les valeurs par défaut comme illustré ci-dessous :

![sparkle](Pictures/201.png)

### Création du flux Azure Logic App

Dans la zone de recherche, entrez "**Blob**", puis sélectionnez "**When a blob is added or modified**"

![sparkle](Pictures/038.png)

Donnez un nom à votre chaine de connexion puis sélectionnez le compte de stockage créé au début de cet article.

![sparkle](Pictures/039.png)

Dans le champ "**Container**", cliquez sur l'icône à droite (1) et sélectionnez le container créé précédement dans cet article. Définissez 5 secondes d'intervale entre chaque vérification de la présence de nouveaux fichiers dans le container
 
![sparkle](Pictures/040.png)

Cliquez sur "**+ New step**" afin de créer une action supplémentaire

![sparkle](Pictures/041.png)

Dans la zone de recherche, entrez le mot "json" puis cliquez sur "Data Operations"

![sparkle](Pictures/042.png)

Sélectionnez "**Compose**"

![sparkle](Pictures/043.png)

Dans le champ input, entrez la partie JSON ci-dessous, sachant que nous allons ensuite compléter la partie encadrée en rouge


```javascript
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
```


![sparkle](Pictures/044.png)

Cliquez à droite de "**recordingUrl**" afin de faire apparaître la fenêtre du contenu dynamique. Cliquez sur "**Expression**" puis sur la fonction "**concat**" 

![sparkle](Pictures/045.png)

Dans la première partie de la concatenation, entrez le texte : **'https://'**
Ce qui va donner la première partie de notre concatenation :
 
 ![sparkle](Pictures/202.png)

Rajoutez ensuite une virgule puis le paramètre du compte de stockage "**storageAccountName"**. Attention, les paramètres se trouvent sous "**Dynamic content**".

![sparkle](Pictures/203.png)

Rajoutez ensuite le texte "**,'.blob.core.windows.net',**"

![sparkle](Pictures/204.png)


Nous allons rajouter la partie spécifique au container et au fichier en récupérant l'information de l'étape précédente. Cliquez sur "**Dynamic Content**" puis sur "**List of Files Path**"

![sparkle](Pictures/205.png)

Enfin, nous allons rajouter l'information concernant la signature d'accès partagée (SAS token).

Rajoutez une virgule puis le paramètre "**SASToken**"

![sparkle](Pictures/206.png)



Puis cliquez sur le bouton "**OK**" (ou Update).

![sparkle](Pictures/207.png)

Vous devez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/050.png)

Et le script de la fonction de concatenation pour le champ "**recordingUrl**" doit ressembler à celui ci-dessous :

```javascript
concat('https://',parameters('storageAccountName'),'.blob.core.windows.net',triggerBody()?['Path'],parameters('SASToken'))

```

Cliquez sur le bouton "**New step**"

![sparkle](Pictures/051.png)

Dans la zone de recherche, entrez "http" puis cliquez sur l'icône "**HTTP**"

![sparkle](Pictures/052.png)

Puis sélectionnez "HTTP"

![sparkle](Pictures/053.png)

Nous allons maintenant envoyer le fichier audio au service cognitif.
Dans la configuration de cette étape, nous allons entrer les valeurs suivantes pour les champs ci-dessous (on fera la partie Body ultérieurement):

- **Method** : POST
- **URI** : concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions')
- **Headers** :
  
|               |                  |
| :------------------------ | -------------------------------: |
| Content-Type              |application/json                  |
| Ocp-Apim-Subscription-Key | *le paramètre de la clef de votre service cognitif*|


========================================================================

Comme vu plus haut, la clef de votre service cognitif se trouve dans la section "**Keys and Endpoint**" de votre service cognitif comme illustré ci-dessous

![sparkle](Pictures/054.png)

========================================================================

Cliquez dans le champ "**URI**" puis rajoutez dans "**Expression**" le script ci-dessous :

```javascript
concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions')
```

Cliquez sur le bouton "**OK**


![sparkle](Pictures/208.png)

Entrez les valeurs suivantes pour la partie "**Header**"

|               |                  |
| :------------------------ | -------------------------------: |
| Content-Type              |application/json                  |
| Ocp-Apim-Subscription-Key | *le paramètre de la clef de votre service cognitif*|

Nous allons donc obtenir au niveau de notre Azure Logic App, quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/209.png)

Nous allons maintenant définir le champ "**Body**".

Cliquez dans le champ "**Body**". Dans le menu contextuel, sélectionnez "**Dynamic content**", puis "**Ouput**" sous la partie "**Compose**"


![sparkle](Pictures/210.png)

Vous pouvez renommer vos étapes en cliquant sur les 3 petits points, puis sur "**Rename**". Par exemple, j'ai nommé cette étape "*Post to cognitive service*"

![sparkle](Pictures/211.png)



Vous devriez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous. 


![sparkle](Pictures/212.png)

Nous allons maintenant récupèrer l'ID de la transcription ainsi que l'URL du fichier audio retournés par le service cognitif
Cliquez sur le bouton "**New Step**"pour rajouter l'étape suivante :

![sparkle](Pictures/213.png)

Dans le champ de recherche entrez "**Parse JSON**" puis sélectionnez "**Parse JSON**"

![sparkle](Pictures/214.png)

Cliquez dans le champ "**Content**" puis sur "**Body**" qui se trouve dans "**Dynamic content**" sous "**Post to cognitive service**"

![sparkle](Pictures/215.png)

Dans le champ "**Schema**", copiez le script ci-dessous :

```javascript
{
    "properties": {
        "createdDateTime": {
            "type": "string"
        },
        "id": {
            "type": "string"
        },
        "lastActionDateTime": {
            "type": "string"
        },
        "recordingsUrl": {
            "type": "string"
        },
        "status": {
            "type": "string"
        },
        "statusMessage": {
            "type": "string"
        }
    },
    "type": "object"
}
```
![sparkle](Pictures/216.png)

Cliquez sur le bouton "**New Step**"

![sparkle](Pictures/217.png)




Nous allons maintenant faire une boucle jusqu'à ce que le service cognitif nous retourne la valeur "**Succeeded**"

Dans le champ de recherche entrez le mot "**until**", puis sélectionnez le control "**Until**"

![sparkle](Pictures/058.png)

Cliquez sur "**Add an action**"

![sparkle](Pictures/059.png)

Puis recherchez le mot "**Delay**" et sélectionnez "**Delay**"

![sparkle](Pictures/060.png)

Paramétrez le délai sur quelques secondes. Dans cet exemple, j'ai paramétré sur 10 secondes. Cliquez sur "**Add an action**"

![sparkle](Pictures/061.png)

Recherchez http, puis sélectionnez "**HTTP**"

![sparkle](Pictures/062.png)

Ici, nous allons attendre une réponse du service cognitif.
Dans le champ "**Method**", sélectionnez "**GET**".

Pour le champ "**URI**" copiez la formule ci-dessous :

```javascript
concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions/',body('Parse_transcription_ID_and_result_URL')?['id'])

```

Pour "**Headers**"
|               |                  |
| :------------------------ | -------------------------------: |
| Content-Type              |application/json                  |
| Ocp-Apim-Subscription-Key | *le paramètre de la clef de votre service cognitif*|


![sparkle](Pictures/063.png)

Même si ce n'est pas forcément nécessaire, mais recommandé, il est possible de renommer les étapes. A droite de "HTTP 2", cliquez sur les 3 points puis sur "**Rename**"


![sparkle](Pictures/064.png)

Vous pouvez nommer cette étape "Get from API", par exemple. Cliquez sur "**Add an action**"

![sparkle](Pictures/065.png)

Nous allons maintenant récupérer le résultat de l'étape précédente et parser le message. Dans le champ de recherche, entrez "Parse JSON", puis sélectionnez "**Parse JSON**"

![sparkle](Pictures/066.png)

Cliquez dans le champ "**Content**". Sélectionnez "**Dynamic content**" puis "**Body**" sous "**Get from API**" 

![sparkle](Pictures/067.png)

Dans le champ "**Schema**", entrez le script JSON ci-dessous

```javascript
{
    "properties": {
        "id": {
            "type": "string"
        },
        "recordingsUrl": {
            "type": "string"
        },
        "status": {
            "type": "string"
        }
    },
    "type": "object"
}
```
Vous devez donc obtenir un résultat similaire à celui ci-dessous

![sparkle](Pictures/068.png)

Revenez au niveau de la boucle "**Until**" pour définir la condition de sortie. Cliquez dans le champ "**Choose a value**". Dans le menu contextuel, cliquez sur "**Dynamic content**" puis cliquez sur "**status**" sous "**Parse JSON**":


![sparkle](Pictures/069.png)

Ensuite définissez la condition à "**is equal to**" "**Succeeded**"

Vous devez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous. Cliquez sur "**New step**"

![sparkle](Pictures/070.png)


********************

A titre de précaution, sauvegardez de temps en temps votre travail :

![sparkle](Pictures/900.png)

********************




Rechechez l'opération "**Parse JSON**". Cliquez dans le champ "**Content**", puis "**Dynamic content**" puis sur "**Body**" sous "**Get from API**"

![sparkle](Pictures/071.png)

Dans le champ "**Schema**", entrez le script JSON ci-dessous :

```javascript
{
    "properties": {
        "createdDateTime": {
            "type": "string"
        },
        "description": {
            "type": "string"
        },
        "id": {
            "type": "string"
        },
        "lastActionDateTime": {
            "type": "string"
        },
        "locale": {
            "type": "string"
        },
        "name": {
            "type": "string"
        },
        "recordingsUrl": {
            "type": "string"
        },
        "reportFileUrl": {
            "type": "string"
        },
        "resultsUrls": {
            "properties": {
                "channel_0": {
                    "type": "string"
                },
                "channel_1": {
                    "type": "string"
                }
            },
            "type": "object"
        },
        "status": {
            "type": "string"
        },
        "statusMessage": {
            "type": "string"
        }
    },
    "type": "object"
}

```

(Pour obtenir le fichier JSON ci-dessous, j'ai fait un test manuel avec Postman. Je détail cette opération à la fin de cet article)

Vous devriez obtenir quelque chose de similaire à la copie d'écran ci-dessous. Cliquez sur le bouton "**New step**" :

![sparkle](Pictures/072.png)

Dans le champ de recherche entrez "**Condition**" puis cliquez sur "**Condition**"

![sparkle](Pictures/073.png)


Clicquez sur le bouton "**Add**" puis "**Add row**" pour rajouter une seconde condition.

![sparkle](Pictures/074.png)



Cliquez sur "**Choose a value**" puis dans "**Dynamic content**", sélectionnez "**channel_0**" sous "**Parse JSON 2**". Cliquez sur "**OK**".



![sparkle](Pictures/075.png)

Choisissez "**is not equal to**" pour le test, puis l'expression "**null**" comme valeur à tester.

![sparkle](Pictures/076.png)

Répétez l'opération pour la seconde ligne mais pour "**Channel_1**". Vous devriez obtenir quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/077.png)

Vous pouvez renommer cette étape pour plus de clarté. par exemple j'ai nommé cette étape "**Test if channel 0 and 1 is not empty**"


![sparkle](Pictures/079.png)


Dans la partie "**If true**", cliquez sur "**Add an action**"

![sparkle](Pictures/078.png)


Dans le champ de recherche entrez "**http**", puis sélectionnez "**HTTP**

![sparkle](Pictures/080.png)


Dans le champ "**Method**", choisissez "**Get**", puis dans le champ "**URI**" sélectionnez "**Dynamic content**" puis "**channel_0**" sous "**Parse JSON 2**"

Vous pouvez renommer cette étape en quelque chose comme "*Get channel 0 results*". 

Cliquez sur "**Add an action**"

![sparkle](Pictures/081.png)


Dans le champ recherche, entrez "**parse json**" et cliquez sur "**Parse JSON**"

![sparkle](Pictures/082.png)

Dans le champ "**Content**", cliquez sur "**Dynamic content**" puis sur "**Body**" sous "**Get channel 0 results**

![sparkle](Pictures/083.png)

Dans le champ "**Schema**", copiez le script ci-dessous :

```javascript
{
    "properties": {
        "AudioFileResults": {
            "items": {
                "properties": {
                    "AudioFileName": {
                        "type": "string"
                    },
                    "AudioFileUrl": {},
                    "AudioLengthInSeconds": {
                        "type": "number"
                    },
                    "CombinedResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Display": {
                                    "type": "string"
                                },
                                "ITN": {
                                    "type": "string"
                                },
                                "Lexical": {
                                    "type": "string"
                                },
                                "MaskedITN": {
                                    "type": "string"
                                }
                            },
                            "required": [
                                "ChannelNumber",
                                "Lexical",
                                "ITN",
                                "MaskedITN",
                                "Display"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "SegmentResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Duration": {
                                    "type": "integer"
                                },
                                "DurationInSeconds": {
                                    "type": "number"
                                },
                                "NBest": {
                                    "items": {
                                        "properties": {
                                            "Confidence": {
                                                "type": "number"
                                            },
                                            "Display": {
                                                "type": "string"
                                            },
                                            "ITN": {
                                                "type": "string"
                                            },
                                            "Lexical": {
                                                "type": "string"
                                            },
                                            "MaskedITN": {
                                                "type": "string"
                                            },
                                            "Sentiment": {
                                                "properties": {
                                                    "Negative": {
                                                        "type": "number"
                                                    },
                                                    "Neutral": {
                                                        "type": "number"
                                                    },
                                                    "Positive": {
                                                        "type": "number"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "Words": {
                                                "items": {
                                                    "properties": {
                                                        "Confidence": {
                                                            "type": "integer"
                                                        },
                                                        "Duration": {
                                                            "type": "integer"
                                                        },
                                                        "DurationInSeconds": {
                                                            "type": "integer"
                                                        },
                                                        "Offset": {
                                                            "type": "integer"
                                                        },
                                                        "OffsetInSeconds": {
                                                            "type": "number"
                                                        },
                                                        "Word": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "required": [
                                                        "Word",
                                                        "Offset",
                                                        "Duration",
                                                        "OffsetInSeconds",
                                                        "DurationInSeconds",
                                                        "Confidence"
                                                    ],
                                                    "type": "object"
                                                },
                                                "type": "array"
                                            }
                                        },
                                        "required": [
                                            "Confidence",
                                            "Lexical",
                                            "ITN",
                                            "MaskedITN",
                                            "Display",
                                            "Sentiment",
                                            "Words"
                                        ],
                                        "type": "object"
                                    },
                                    "type": "array"
                                },
                                "Offset": {
                                    "type": "integer"
                                },
                                "OffsetInSeconds": {
                                    "type": "number"
                                },
                                "RecognitionStatus": {
                                    "type": "string"
                                },
                                "SpeakerId": {}
                            },
                            "required": [
                                "RecognitionStatus",
                                "ChannelNumber",
                                "SpeakerId",
                                "Offset",
                                "Duration",
                                "OffsetInSeconds",
                                "DurationInSeconds",
                                "NBest"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    }
                },
                "required": [
                    "AudioFileName",
                    "AudioFileUrl",
                    "AudioLengthInSeconds",
                    "CombinedResults",
                    "SegmentResults"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
}
```

Vous devez obtenir un résultat similaire à la copie d'écran ci-dessous. 

Renommez l'étape par quelque chose comme "**Parse Channel 0**".

Cliquez sur "**Add an action**".

![sparkle](Pictures/084.png)

Dans le champ de recherche entrez "**http**" puis cliquez sur "**HTTP**".

![sparkle](Pictures/085.png)

Dans le champ "**Method**", choisissez "**Get**", puis dans le champ "**URI**" sélectionnez "**Dynamic content**" puis "**channel_1**" sous "**Parse JSON 2**".

Vous pouvez renommer cette étape en quelque chose comme "*Get channel 1 results*". 

Cliquez sur "**Add an action**".

![sparkle](Pictures/086.png)

Dans le champ de recherche, entrez "**parse json**" puis cliquez sur "**Parse JSON**".

![sparkle](Pictures/087.png)

Cliquez dans le champ "**Content**", puis sur "**Dynamic content**". Cliquez sur "**Body**" qui se trouve sous "**Get channel 1 results**".

![sparkle](Pictures/088.png)

Dans le champ "**Schema**", copiez le script ci-dessous :

```javascript
{
    "properties": {
        "AudioFileResults": {
            "items": {
                "properties": {
                    "AudioFileName": {
                        "type": "string"
                    },
                    "AudioFileUrl": {},
                    "AudioLengthInSeconds": {
                        "type": "number"
                    },
                    "CombinedResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Display": {
                                    "type": "string"
                                },
                                "ITN": {
                                    "type": "string"
                                },
                                "Lexical": {
                                    "type": "string"
                                },
                                "MaskedITN": {
                                    "type": "string"
                                }
                            },
                            "required": [
                                "ChannelNumber",
                                "Lexical",
                                "ITN",
                                "MaskedITN",
                                "Display"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "SegmentResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Duration": {
                                    "type": "integer"
                                },
                                "DurationInSeconds": {
                                    "type": "number"
                                },
                                "NBest": {
                                    "items": {
                                        "properties": {
                                            "Confidence": {
                                                "type": "number"
                                            },
                                            "Display": {
                                                "type": "string"
                                            },
                                            "ITN": {
                                                "type": "string"
                                            },
                                            "Lexical": {
                                                "type": "string"
                                            },
                                            "MaskedITN": {
                                                "type": "string"
                                            },
                                            "Sentiment": {
                                                "properties": {
                                                    "Negative": {
                                                        "type": "number"
                                                    },
                                                    "Neutral": {
                                                        "type": "number"
                                                    },
                                                    "Positive": {
                                                        "type": "number"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "Words": {
                                                "items": {
                                                    "properties": {
                                                        "Confidence": {
                                                            "type": "integer"
                                                        },
                                                        "Duration": {
                                                            "type": "integer"
                                                        },
                                                        "DurationInSeconds": {
                                                            "type": "integer"
                                                        },
                                                        "Offset": {
                                                            "type": "integer"
                                                        },
                                                        "OffsetInSeconds": {
                                                            "type": "number"
                                                        },
                                                        "Word": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "required": [
                                                        "Word",
                                                        "Offset",
                                                        "Duration",
                                                        "OffsetInSeconds",
                                                        "DurationInSeconds",
                                                        "Confidence"
                                                    ],
                                                    "type": "object"
                                                },
                                                "type": "array"
                                            }
                                        },
                                        "required": [
                                            "Confidence",
                                            "Lexical",
                                            "ITN",
                                            "MaskedITN",
                                            "Display",
                                            "Sentiment",
                                            "Words"
                                        ],
                                        "type": "object"
                                    },
                                    "type": "array"
                                },
                                "Offset": {
                                    "type": "integer"
                                },
                                "OffsetInSeconds": {
                                    "type": "number"
                                },
                                "RecognitionStatus": {
                                    "type": "string"
                                },
                                "SpeakerId": {}
                            },
                            "required": [
                                "RecognitionStatus",
                                "ChannelNumber",
                                "SpeakerId",
                                "Offset",
                                "Duration",
                                "OffsetInSeconds",
                                "DurationInSeconds",
                                "NBest"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    }
                },
                "required": [
                    "AudioFileName",
                    "AudioFileUrl",
                    "AudioLengthInSeconds",
                    "CombinedResults",
                    "SegmentResults"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
}
```

Renommez cette étape en quelque chose comme "*Parse Channel 1*".

Cliquez sur "**Add an action**"

![sparkle](Pictures/089.png)

Dans la champ de recherche, "**compose**", puis cliquez sur "**Compose**".

![sparkle](Pictures/090.png)

Renommez cette étape par quelque chose comme "*Compose document with channel 0 and 1*".

Dans le champ "**Inputs**", copiez le script ci-dessous :


```javascript
{
  "AudioDetails": [
    {
      "Channel_0": ,
      "Channel_1": 
    }
  ],
  "AudioLink": "",
  "Date": "",
  "Language": "English",
  "MyDate": "",
  "id": ""
}

```
Vous devez obtenir un résultat similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/091.png)

Cliquez à droite de **"Channel_0":** et juste avant la virgule. Rajoutez "**Body**" qui se trouve dans "**Dynamic content**" sous "**Parse Channel 0**"

![sparkle](Pictures/092.png)

Mainenant, cliquez à droite de **"Channel_1":**. Rajoutez "**Body**" qui se trouve dans "**Dynamic content**" sous "**Parse Channel 1**"

![sparkle](Pictures/093.png)

Cliquez à droite de **"AudioLink":**, entre les doubles quotes, puis rajoutez "**recordingUrl**" qui se trouve dans "**Dynamic content**" sous "**Parse transcription ID and result URL**"

![sparkle](Pictures/094.png)

Cliquez à droite de **"Date":**, entre les doubles quotes, puis rajoutez l'expression :
```javascript
utcNow()
```

Cliquez sur "**OK**"

![sparkle](Pictures/095.png)

Cliquez à droite de **"MyDate":**, entre les doubles quotes, puis rajoutez l'expression


```javascript
formatDateTime(utcNow(),'yyyy-MM-dd')
```

Cliquez sur "**OK**"

![sparkle](Pictures/096.png)

Cliquez à droite de **"id":**, entre les doubles quotes, puis cliquez sur "**id**" qui se trouve dans "**Dynamic content**" sous "**Parse transcription ID and result URL**"

![sparkle](Pictures/097.png)

Vous devez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous.

Cliquez sur "**Add an action**"

![sparkle](Pictures/098.png)

Dans le champ de recherche, entrez "**parse**" puis cliquez sur "**Parse JSON**"

![sparkle](Pictures/099.png)

Renommez l'étape en quelque chose comme "*Parse document with channel 0 and 1 and additional information*"

Dans le champ "**Content**", cliquez sur "**Outputs**" qui se trouve dans "**Dynamic content**" sous "**Compose document with channel 0 and 1**"


![sparkle](Pictures/100.png)


Dans le champ "**Schema**", rajoutez le script ci-dessous :

```javascript
{
    "properties": {
        "AudioDetails": {
            "items": {
                "properties": {
                    "Channel_0": {
                        "properties": {
                            "AudioFileResults": {
                                "items": {
                                    "properties": {
                                        "AudioFileName": {
                                            "type": "string"
                                        },
                                        "AudioFileUrl": {},
                                        "SegmentResults": {
                                            "items": {
                                                "properties": {
                                                    "ChannelNumber": {},
                                                    "Duration": {
                                                        "type": "integer"
                                                    },
                                                    "NBest": {
                                                        "items": {
                                                            "properties": {
                                                                "Confidence": {
                                                                    "type": "number"
                                                                },
                                                                "Display": {
                                                                    "type": "string"
                                                                },
                                                                "ITN": {
                                                                    "type": "string"
                                                                },
                                                                "Lexical": {
                                                                    "type": "string"
                                                                },
                                                                "MaskedITN": {
                                                                    "type": "string"
                                                                },
                                                                "Words": {
                                                                    "items": {
                                                                        "properties": {
                                                                            "Duration": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Offset": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Word": {
                                                                                "type": "string"
                                                                            }
                                                                        },
                                                                        "required": [
                                                                            "Word",
                                                                            "Offset",
                                                                            "Duration"
                                                                        ],
                                                                        "type": "object"
                                                                    },
                                                                    "type": "array"
                                                                }
                                                            },
                                                            "required": [
                                                                "Confidence",
                                                                "Lexical",
                                                                "ITN",
                                                                "MaskedITN",
                                                                "Display",
                                                                "Words"
                                                            ],
                                                            "type": "object"
                                                        },
                                                        "type": "array"
                                                    },
                                                    "Offset": {
                                                        "type": "integer"
                                                    },
                                                    "RecognitionStatus": {
                                                        "type": "string"
                                                    },
                                                    "SpeakerId": {}
                                                },
                                                "required": [
                                                    "RecognitionStatus",
                                                    "ChannelNumber",
                                                    "SpeakerId",
                                                    "Offset",
                                                    "Duration",
                                                    "NBest"
                                                ],
                                                "type": "object"
                                            },
                                            "type": "array"
                                        }
                                    },
                                    "required": [
                                        "AudioFileName",
                                        "AudioFileUrl",
                                        "SegmentResults"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    },
                    "Channel_1": {
                        "properties": {
                            "AudioFileResults": {
                                "items": {
                                    "properties": {
                                        "AudioFileName": {
                                            "type": "string"
                                        },
                                        "AudioFileUrl": {},
                                        "SegmentResults": {
                                            "items": {
                                                "properties": {
                                                    "ChannelNumber": {},
                                                    "Duration": {
                                                        "type": "integer"
                                                    },
                                                    "NBest": {
                                                        "items": {
                                                            "properties": {
                                                                "Confidence": {
                                                                    "type": "number"
                                                                },
                                                                "Display": {
                                                                    "type": "string"
                                                                },
                                                                "ITN": {
                                                                    "type": "string"
                                                                },
                                                                "Lexical": {
                                                                    "type": "string"
                                                                },
                                                                "MaskedITN": {
                                                                    "type": "string"
                                                                },
                                                                "Words": {
                                                                    "items": {
                                                                        "properties": {
                                                                            "Duration": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Offset": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Word": {
                                                                                "type": "string"
                                                                            }
                                                                        },
                                                                        "required": [
                                                                            "Word",
                                                                            "Offset",
                                                                            "Duration"
                                                                        ],
                                                                        "type": "object"
                                                                    },
                                                                    "type": "array"
                                                                }
                                                            },
                                                            "required": [
                                                                "Confidence",
                                                                "Lexical",
                                                                "ITN",
                                                                "MaskedITN",
                                                                "Display",
                                                                "Words"
                                                            ],
                                                            "type": "object"
                                                        },
                                                        "type": "array"
                                                    },
                                                    "Offset": {
                                                        "type": "integer"
                                                    },
                                                    "RecognitionStatus": {
                                                        "type": "string"
                                                    },
                                                    "SpeakerId": {}
                                                },
                                                "required": [
                                                    "RecognitionStatus",
                                                    "ChannelNumber",
                                                    "SpeakerId",
                                                    "Offset",
                                                    "Duration",
                                                    "NBest"
                                                ],
                                                "type": "object"
                                            },
                                            "type": "array"
                                        }
                                    },
                                    "required": [
                                        "AudioFileName",
                                        "AudioFileUrl",
                                        "SegmentResults"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    }
                },
                "required": [
                    "Channel_0",
                    "Channel_1"
                ],
                "type": "object"
            },
            "type": "array"
        },
        "AudioLink": {
            "type": "string"
        },
        "Date": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "MyDate": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```

Vous devez obtenir quelque chose de similaire à la copie d'écran ci-dessous.

Cliquez sur "**Add an action**".

![sparkle](Pictures/101.png)


Dans le champ de recherche, entrez "**cosmos**", puis cliquez sur "**Create or update document**"


![sparkle](Pictures/102.png)


*********************************
Il est fort probable que Azure Logic App vous demande de créer une connection vers votre compte Azure Cosmos DB.

Choissisez votre compte et cliquez sur le bouton "**Create**"

![sparkle](Pictures/107.png)


*********************************


Complétez les champs "**Database ID**" et "**Collection ID**" avec les informations de votre compte Azure Cosmos DB

![sparkle](Pictures/103.png)

Cliquez dans le champ "**Document**", puis sur "**Body**" qui se trouve dans "**Dynamic content**" sous "**Parse document with channel 0 and 1**"


![sparkle](Pictures/104.png)


Cliquez sur "**Add new parameter**"


![sparkle](Pictures/105.png)


Puis choisissez "**Partition key value**"


![sparkle](Pictures/106.png)

Puis **entre des doubles quotes**, rajoutez le champ "**MyDate**" qui se trouve dans "**Dynamic content**" sous "**Parse document with channel 0 and 1**"


![sparkle](Pictures/108.png)

Nous allons maintenant traiter les fichiers audio avec seulement un seul canal. Comme les étapes sont quasiment identiques à celles que nous venons de voir, je ne vais pas les détailler.

Ci-dessous une vue globale de la branche "**If false**" de notre condition :

![sparkle](Pictures/109.png)



Dans la partie "**If false**" de la première condition "**Test if channel 0 and 1 is not empty**", rajoutez une seconde condition pour ne tester que le canal 0. 

![sparkle](Pictures/218.png)


Dans la partie "**If true**" de notre seconde condition, rajoutez le traitement du canal 0 uniquement, en suivant les étapes précédentes **mais avec les nuances suivantes** :

Pour l'étape HTTP :

![sparkle](Pictures/219.png)

Pour l'étape Parse JSON

![sparkle](Pictures/220.png)

Copiez le script ci-dessous dans le champ schema :

```javascript
{
    "properties": {
        "AudioFileResults": {
            "items": {
                "properties": {
                    "AudioFileName": {
                        "type": "string"
                    },
                    "AudioFileUrl": {},
                    "AudioLengthInSeconds": {
                        "type": "number"
                    },
                    "CombinedResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Display": {
                                    "type": "string"
                                },
                                "ITN": {
                                    "type": "string"
                                },
                                "Lexical": {
                                    "type": "string"
                                },
                                "MaskedITN": {
                                    "type": "string"
                                }
                            },
                            "required": [
                                "ChannelNumber",
                                "Lexical",
                                "ITN",
                                "MaskedITN",
                                "Display"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "SegmentResults": {
                        "items": {
                            "properties": {
                                "ChannelNumber": {},
                                "Duration": {
                                    "type": "integer"
                                },
                                "DurationInSeconds": {
                                    "type": "number"
                                },
                                "NBest": {
                                    "items": {
                                        "properties": {
                                            "Confidence": {
                                                "type": "number"
                                            },
                                            "Display": {
                                                "type": "string"
                                            },
                                            "ITN": {
                                                "type": "string"
                                            },
                                            "Lexical": {
                                                "type": "string"
                                            },
                                            "MaskedITN": {
                                                "type": "string"
                                            },
                                            "Sentiment": {
                                                "properties": {
                                                    "Negative": {
                                                        "type": "number"
                                                    },
                                                    "Neutral": {
                                                        "type": "number"
                                                    },
                                                    "Positive": {
                                                        "type": "number"
                                                    }
                                                },
                                                "type": "object"
                                            },
                                            "Words": {
                                                "items": {
                                                    "properties": {
                                                        "Confidence": {
                                                            "type": "integer"
                                                        },
                                                        "Duration": {
                                                            "type": "integer"
                                                        },
                                                        "DurationInSeconds": {
                                                            "type": "integer"
                                                        },
                                                        "Offset": {
                                                            "type": "integer"
                                                        },
                                                        "OffsetInSeconds": {
                                                            "type": "number"
                                                        },
                                                        "Word": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "required": [
                                                        "Word",
                                                        "Offset",
                                                        "Duration",
                                                        "OffsetInSeconds",
                                                        "DurationInSeconds",
                                                        "Confidence"
                                                    ],
                                                    "type": "object"
                                                },
                                                "type": "array"
                                            }
                                        },
                                        "required": [
                                            "Confidence",
                                            "Lexical",
                                            "ITN",
                                            "MaskedITN",
                                            "Display",
                                            "Sentiment",
                                            "Words"
                                        ],
                                        "type": "object"
                                    },
                                    "type": "array"
                                },
                                "Offset": {
                                    "type": "integer"
                                },
                                "OffsetInSeconds": {
                                    "type": "number"
                                },
                                "RecognitionStatus": {
                                    "type": "string"
                                },
                                "SpeakerId": {}
                            },
                            "required": [
                                "RecognitionStatus",
                                "ChannelNumber",
                                "SpeakerId",
                                "Offset",
                                "Duration",
                                "OffsetInSeconds",
                                "DurationInSeconds",
                                "NBest"
                            ],
                            "type": "object"
                        },
                        "type": "array"
                    }
                },
                "required": [
                    "AudioFileName",
                    "AudioFileUrl",
                    "AudioLengthInSeconds",
                    "CombinedResults",
                    "SegmentResults"
                ],
                "type": "object"
            },
            "type": "array"
        }
    },
    "type": "object"
}
```



Pour l'étape "**Compose for only channel 0**", utilisez le script ci dessous :

```javascript
{
  "AudioDetails": [
    {
      "Channel_0":     
    }
  ],
  "AudioLink": "",
  "Date": "",
  "Language": "English",
  "MyDate": "",
  "id": ""
}

```


![sparkle](Pictures/110.png)
 

Pour l'étape "**Parse document for Channel 0** ", utilisez le script ci-dessous :

```javascript
{
    "properties": {
        "AudioDetails": {
            "items": {
                "properties": {
                    "Channel_0": {
                        "properties": {
                            "AudioFileResults": {
                                "items": {
                                    "properties": {
                                        "AudioFileName": {
                                            "type": "string"
                                        },
                                        "AudioFileUrl": {},
                                        "SegmentResults": {
                                            "items": {
                                                "properties": {
                                                    "ChannelNumber": {},
                                                    "Duration": {
                                                        "type": "integer"
                                                    },
                                                    "NBest": {
                                                        "items": {
                                                            "properties": {
                                                                "Confidence": {
                                                                    "type": "number"
                                                                },
                                                                "Display": {
                                                                    "type": "string"
                                                                },
                                                                "ITN": {
                                                                    "type": "string"
                                                                },
                                                                "Lexical": {
                                                                    "type": "string"
                                                                },
                                                                "MaskedITN": {
                                                                    "type": "string"
                                                                },
                                                                "Words": {
                                                                    "items": {
                                                                        "properties": {
                                                                            "Duration": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Offset": {
                                                                                "type": "integer"
                                                                            },
                                                                            "Word": {
                                                                                "type": "string"
                                                                            }
                                                                        },
                                                                        "required": [
                                                                            "Word",
                                                                            "Offset",
                                                                            "Duration"
                                                                        ],
                                                                        "type": "object"
                                                                    },
                                                                    "type": "array"
                                                                }
                                                            },
                                                            "required": [
                                                                "Confidence",
                                                                "Lexical",
                                                                "ITN",
                                                                "MaskedITN",
                                                                "Display",
                                                                "Words"
                                                            ],
                                                            "type": "object"
                                                        },
                                                        "type": "array"
                                                    },
                                                    "Offset": {
                                                        "type": "integer"
                                                    },
                                                    "RecognitionStatus": {
                                                        "type": "string"
                                                    },
                                                    "SpeakerId": {}
                                                },
                                                "required": [
                                                    "RecognitionStatus",
                                                    "ChannelNumber",
                                                    "SpeakerId",
                                                    "Offset",
                                                    "Duration",
                                                    "NBest"
                                                ],
                                                "type": "object"
                                            },
                                            "type": "array"
                                        }
                                    },
                                    "required": [
                                        "AudioFileName",
                                        "AudioFileUrl",
                                        "SegmentResults"
                                    ],
                                    "type": "object"
                                },
                                "type": "array"
                            }
                        },
                        "type": "object"
                    }
                },
                "required": [
                    "Channel_0"
                ],
                "type": "object"
            },
            "type": "array"
        },
        "AudioLink": {
            "type": "string"
        },
        "Date": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "MyDate": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```
Vous devez obtenir un résulat similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/111.png)

Puis pour l'étape Azure Cosmos DB

![sparkle](Pictures/221.png)


Nous allons maintenant rajouter la dernière étape de notre Logic App. Cette étape consiste à supprimer les informations de transcription, au niveau du service cognitif, afin de laisser la place libre pour la prochaine transcription.

Cliquez sur le bouton "**New step**"



Dans la zone de recherche, entrez "**http**" puis cliquez sur "**HTTP**"

![sparkle](Pictures/121.png)

Dans le champ "**Method**", sélectionnez "**DELETE**".

Cliquez dans le champ **URI**, puis cliquez sur "**Expression**". Collez le script ci-dessous. Cliquez sur le bouton "**OK**"


```javascript
concat('https://',parameters('azureRegion'),'.cris.ai/api/speechtotext/v2.0/Transcriptions/',body('Parse_transcription_ID_and_result_URL')?['id'])
```

![sparkle](Pictures/122.png)


Enfin, dans le champ "**header**", rajoutez les information suivantes :

| Content-Type              |                 application/json |
| :------------------------ | -------------------------------: |
| Ocp-Apim-Subscription-Key | *Le paramètre de la clef de votre service cognitif*|

Comme illustré ci-dessous

![sparkle](Pictures/123.png)

Cliquez sur le bouton "**Save**"

![sparkle](Pictures/124.png)

# Test de la solution

Maintenant que nous avons terminé notre logic app, nous allons nous assurer que notre service est bien dans le status "**Enabled**" 


![sparkle](Pictures/129.png)



Dans le cas contraire, cliquez sur le bouton "**Enable**"

![sparkle](Pictures/130.png)

Avec Azure Storage Explorer, déposez le fichier audio, qui se trouve dans le dossier [AudioFile](https://github.com/franmer2/speechtotextfr/tree/master/AudioFile), dans votre compte de stockage. Votre Azure Logic App doit alors se déclencher et commencer le process de transcription du fichier audio


![sparkle](Pictures/131.png)

Si tout va bien, l'opération doit se dérouler avec succes

![sparkle](Pictures/132.png)

Cliquez sur "**Succeeded**" (ou "Failed" suivant les cas :)) afin d'avoir plus de  détails 

![sparkle](Pictures/133.png)

Allez voir le résultat dans votre collection Azure Cosmos DB.

![sparkle](Pictures/134.png)


# Power BI
Maintenant que les données du fichier sont stockées dans une base Azure Cosmos DB, il est naturel de vouloir visualiser ces données via Power BI. Je ne vais pas détailler comment faire le rapport mais juste illustrer comment démarrer. Et je vous [partage](https://github.com/franmer2/speechtotextus/tree/master/Power%20BI) aussi un rapport que j'ai fait, pour exemple.

Depuis Power BI Desktop, cliquez sur "**Get Data**" puis sur "**More**"

![sparkle](Pictures/137.png)

Cliquez sur "**Azure**" puis sur "**Azure Cosmos DB**". Cliquez sur le bouton "**Connect**"

![sparkle](Pictures/138.png)

Renseignez l'URL de votre compte Azure Cosmos DB (que vous pouvez retrouver sur le portail Azure), puis cliquez sur le bouton "**OK**"

![sparkle](Pictures/139.png)

Entrez la clef de votre compte Azure Cosmos DB (que vous pouvez retrouver sur le portail Azure). Cliquez sur le bouton "**Connect**"

![sparkle](Pictures/140.png)

Vous devriez être capable de naviguer dans vos bases de données et collections. Choisissez votre collection et cliquez sur le bouton "**Transform Data**"

![sparkle](Pictures/141.png)

Dans Power Query Editor, cliquez sur les flèches pour accéder aux champs de votre collection

![sparkle](Pictures/142.png)

Choisissez les champs dont vous avez besoin et cliquez sur le bouton "**OK**"

![sparkle](Pictures/143.png)

Pour les colonnes qui ont encore les doubles flèches, cliquez dessus pour choisir les champs que vous souhaitez intégrer dans le rapport. Il est possible de répéter cette opération plusieurs fois, tout dépend de la profondeur de la structure du document.

![sparkle](Pictures/144.png)

Ci-dessous, un exemple de rapport Power BI (disponible sous le dossier [*Power BI*](https://github.com/franmer2/speechtotextfr/tree/master/Power%20BI)). J'ai rajouté la possibilité de faire un "**Drillthrough**" au niveau des vues générales des transcriptions afin d'avoir le détail des conversations.

![sparkle](Pictures/902.png)


Pour représenter la discussion dans le rapport, j'ai utilisé une visualisation de type Diagramme de Gantt

![sparkle](Pictures/145.png)



# Et si j'ai des fichiers dans une autre langue ?

J'ai eu à traiter des fichiers en Anglais et en Français.
A l'époque (été 2019 :)), j'ai créé un second container pour recevoir les fichiers en Français, j'ai cloné mon Logic App, dans lequel j'ai changé le trigger de déclenchement pour le faire pointer sur le nouveau container. Puis j'ai changé la configuration de l'appel au service cognitif pour passer la langue Française en paramètre. Avec une grosse restriction ! L'analyse de sentiment n'est pas supportée, donc il faut définir ce paramètre sur "**False**" :(. 



![sparkle](Pictures/146.png)

La liste des langues supportées est disponible [ici](https://docs.microsoft.com/fr-fr/azure/cognitive-services/speech-service/language-support)

Aujourd'hui, il est peut-être possible de détecter la langue du fichier audio et de la passer en paramètre, mais je n'ai pas regardé ce point.


# Sous le capot

Vous devez peut-être vous poser la question à propos des longs scripts que l'on doit rajouter lors des étapes "Parse JSON" et comment les générer. Pour ce faire, j'ai utilisé l'outil [Postman](https://www.postman.com/) (mais je ne dis pas que c'est la meilleure façon de faire). Avec Postman, j'ai envoyé manuellement un fichier audio au service cognitif. Toujours avec l'outil Postman, je récupère les résultats au format JSON.

Ci-dessous la configuration de Postman pour envoyer un fichier audio au service cognitif.

La configuration de la partie "**Headers**"

![sparkle](Pictures/125.png)

Un exemple de configuration de la partie "**Body**"

![sparkle](Pictures/126.png)

Ensuite je récupére le résultat avec un "Get"

![sparkle](Pictures/127.png)

Que je copie dans la partie "**Use sample payload to generate schema**" de l'étape "**Parse JSON**" d'Azure Logic App

![sparkle](Pictures/128.png)

# Troubleshooting

Une des erreurs que j'ai eu lors des tests, était dû au fait que le service cognitif conservait les informations des fichiers audio précédent. C'est pour cette raison que la dernière étape du Logic App est une action "Delete" au niveau du service cognitif.

Dans le cas où vous avez une erreur avec Logic app, vous pouvez vérifier avec Postman, avec une commande "GET", si des informations sont encore présentes au niveau du service cognitif. Si c'est le cas, copiez l'id du document retourné.

![sparkle](Pictures/135.png)


Puis utilisez cet id pour envoyer une commande "DELETE" avec Postman, de la manière illustrée ci-dessous :


![sparkle](Pictures/136.png)



