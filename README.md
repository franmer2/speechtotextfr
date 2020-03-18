Dans cet article, je vais détailler les étapes pour transformer une discussion enregistrée dans un fichier audio en texte, enrichir cette transcription avec des analyses de sentiments, puis terminer par une représentation de cette discussion dans un rapport Power BI afin d'en faire une analyse.

Pour ce faire nous allons utiliser plusieurs services de la plateforme Azure dont principalement :

- Un compte de stockage Azure
- Le service cognitif "Speech to text"
- Azure Logic App
- Azure Cosmos DB

Ci-dessous un schéma de la solution que nous allons mettre en place dans Azure

![sparkle](Pictures/901.png)

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

Dans la première partie de la concatenation, entrez l'adresse de votre compte de stockage. Cette adresse peut se retrouver dans les propriétés de votre compte de stockage. **N'oubliez pas les quotes pour entrer l'adresse : 'https://audiofiledemo.blob.core......'**
 
![sparkle](Pictures/046.png)

Ce qui va donner la première partie de notre concatenation :

![sparkle](Pictures/047.png)

Rajoutez une virgule à la fin du texte. Nous allons rajouter la partie spécifique au container et au fichier en récupérant l'information de l'étape précédente. Cliquez sur "**Dynamic Content**" puis sur "**List of Files Path**"

![sparkle](Pictures/048.png)

Puis cliquez sur le bouton "**OK**"

![sparkle](Pictures/049.png)

Vous devez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/050.png)

Et le script de la fonction de concatenation doit ressembler à celui ci-dessous :

```javascript
concat('https://audiofiledemo.blob.core.windows.net',triggerBody()?['Path'])

```

Cliquez sur le bouton "**New step**"

![sparkle](Pictures/051.png)

dans la zone de recherche, entrez "http" puis cliquez sur l'icône "**HTTP**"

![sparkle](Pictures/052.png)

Puis sélectionnez "HTTP"

![sparkle](Pictures/053.png)

Nous allons maintenant envoyer le fichier audio au service cognitif.
Dans la configuration de cette étape, nous allons entrer les valeurs suivantes pour les champs ci-dessous (on fera la partie Body ultérieurement):

- **Method** : POST
- **URI** : https://canadacentral.cris.ai/api/speechtotext/v2.0/Transcriptions
- **Headers** :
  
| Content-Type              |                 application/json |
| :------------------------ | -------------------------------: |
| Ocp-Apim-Subscription-Key | *la clef de votre service cognitif*|

La clef de votre service cognitif se trouve dans la section "**Keys and Endpoint**" de votre service cognitif comme illustré ci-dessous

![sparkle](Pictures/054.png)

Nous allons donc obtenir au niveau de notre Azure Logic App, quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/055.png)

Nous allons maintenant définir le champ "**Body**".

Cliquez dans le champ "**Body**". Dans le menu contextuel, sélectionnez "**Dynamic content**", puis "**Ouput**" sous la partie "**Compose**"


![sparkle](Pictures/056.png)

Vous devriez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous. Cliquez sur le bouton "**New Step**"pour rajouter l'étape suivante :


![sparkle](Pictures/057.png)

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

Dans le champ "**Method**", sélectionnez "**GET**", puis remplissez les champs URI et Headers comme précédemment. Ici, nous allons attendre une réponse du service cognitif.

![sparkle](Pictures/063.png)

Même si ce n'est pas forcément nécessaire, il est possible de renommer les étapes. A droite de "HTTP 2", cliquez sur les 3 points puis sur "**Rename**"


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
    "items": {
        "properties": {
            "status": {
                "type": "string"
            }
        },
        "required": [
            "status"
        ],
        "type": "object"
    },
    "type": "array"
}
```
Vous devez donc obtenir un résultat similaire à celui ci-dessous

![sparkle](Pictures/068.png)

Revenez au niveau de la boucle "**Until**" pour définir la condition de sortie. Cliquez dans le champ "**Choose a value**". Dans le menu contextuel, cliquez sur "**Expression**" puis entrez la commande ci-dessous puis cliquez sur le bouton "**OK**" :


```javascript
body('Parse_JSON')[0]['status']
```
![sparkle](Pictures/069.png)

Ensuite définissez la condition à "**is equal to**" "**Succeeded**"

============================================

A titre de précaution, sauvegardez de temps en temps votre travail :

![sparkle](Pictures/900.png)

============================================



Vous devez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous. Cliquez sur "**New step**"

![sparkle](Pictures/070.png)

Rechechez l'opération "**Parse JSON**". Cliquez dans le champ "**Content**", puis "**Dynamic content**" puis sur "**Body**" sous "**Get from API**"

![sparkle](Pictures/071.png)

Dans le champ "**Schema**", entrez le script JSON ci-dessous :

```javascript
{
    "items": {
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
            "models": {
                "items": {
                    "properties": {
                        "createdDateTime": {
                            "type": "string"
                        },
                        "datasets": {
                            "type": "array"
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
                        "modelKind": {
                            "type": "string"
                        },
                        "name": {
                            "type": "string"
                        },
                        "properties": {
                            "properties": {
                                "ModelClass": {
                                    "type": "string"
                                },
                                "Purpose": {
                                    "type": "string"
                                },
                                "VadKind": {
                                    "type": "string"
                                }
                            },
                            "type": "object"
                        },
                        "status": {
                            "type": "string"
                        }
                    },
                    "required": [
                        "modelKind",
                        "datasets",
                        "id",
                        "createdDateTime",
                        "lastActionDateTime",
                        "status",
                        "locale",
                        "name",
                        "description",
                        "properties"
                    ],
                    "type": "object"
                },
                "type": "array"
            },
            "name": {
                "type": "string"
            },
            "properties": {
                "properties": {
                    "AddWordLevelTimestamps": {
                        "type": "string"
                    },
                    "Duration": {
                        "type": "string"
                    },
                    "ProfanityFilterMode": {
                        "type": "string"
                    },
                    "PunctuationMode": {
                        "type": "string"
                    }
                },
                "type": "object"
            },
            "recordingsUrl": {
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
        "required": [
            "recordingsUrl",
            "resultsUrls",
            "models",
            "statusMessage",
            "id",
            "createdDateTime",
            "lastActionDateTime",
            "status",
            "locale",
            "name",
            "description",
            "properties"
        ],
        "type": "object"
    },
    "type": "array"
}

```

(pour obtenir le fichier JSON ci-dessous, j'ai fait un test manuel avec Postman. Je détail cette opération à la fin de cet article)

Vous devriez obtenir quelque chose de similaire à la copie d'écran ci-dessous. Cliquez sur le bouton "**New step**" :

![springle](Pictures/072.png)

Dans le champ de recherche entrez "**Condition**" puis cliquez sur "**Condition**"

![springle](Pictures/073.png)

Cliquez dans le champ "**Choose a value**". Dans "**Dynamic content**", cliquez sur "**channel_0**" sous "**Parse JSON 2**"

![springle](Pictures/074.png)

L'étape s'inclut alors dans une boucle *"For each"* comme illustré ci-dessous :

![springle](Pictures/075.png)

Cliquez à nouveau sur l'étape "**Condition**". Choisissez la condition "**Is not equal to**". Cliquez dans le champ "**Choose a value**" puis cliquez sur "**Expression**". Dans le champ "**Fx**" entrez le mot "**null**". Cliquez sur le bouton "**OK**"


![springle](Pictures/076.png)



Dans la partie "**If true**", cliquez sur "**Add an action**"

![springle](Pictures/077.png)

Dans la zone de recherche, entrez "**http**" puis sélectionnez "**HTTP**"

![springle](Pictures/078.png)

Avec l'action suivante, nous allons récupérer les informations concernant le canal numéro 0. Pour ce faire, nous avons besoin de l'URL contenant ces informations. Cette URL se trouve dans la sortie de l'action "*Parse JSON 2*"

Dans le champ "**Method**", choisissez "**GET**". Cliquez dans URI, puis dans "**Dynamic content**" sélectionnez "**channel_0**" sous "**Parse JSON 2**".


![springle](Pictures/079.png)

Renommez l'action "**HTTP 2**" par quelque chose commme "**Get channel 0 results**"

![springle](Pictures/080.png)

Cliquez sur "**Add an action**"

![springle](Pictures/081.png)

Dans la zone de recherche entrez "**Parse JSON**", puis sélectionnez "**Parse JSON**".

![springle](Pictures/082.png)

Cliquez dans le champ "**Content**", puis dans "**Dynamic content**", cliquez sur "**Body**" sous "**Get channel 0 results**"

![springle](Pictures/083.png)

Dans le champ "**Schema**", collez le script ci-dessous :


```javascript
{
    "properties": {
        "properties": {
            "properties": {
                "AudioFileResults": {
                    "properties": {
                        "items": {
                            "properties": {
                                "properties": {
                                    "properties": {
                                        "AudioFileName": {
                                            "properties": {
                                                "type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        },
                                        "AudioFileUrl": {
                                            "properties": {},
                                            "type": "object"
                                        },
                                        "SegmentResults": {
                                            "properties": {
                                                "items": {
                                                    "properties": {
                                                        "properties": {
                                                            "properties": {
                                                                "ChannelNumber": {
                                                                    "properties": {},
                                                                    "type": "object"
                                                                },
                                                                "Duration": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "NBest": {
                                                                    "properties": {
                                                                        "items": {
                                                                            "properties": {
                                                                                "properties": {
                                                                                    "properties": {
                                                                                        "Confidence": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Display": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "ITN": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Lexical": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "MaskedITN": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Sentiment": {
                                                                                            "properties": {
                                                                                                "properties": {
                                                                                                    "properties": {
                                                                                                        "Negative": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "Neutral": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "Positive": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        }
                                                                                                    },
                                                                                                    "type": "object"
                                                                                                },
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Words": {
                                                                                            "properties": {
                                                                                                "items": {
                                                                                                    "properties": {
                                                                                                        "properties": {
                                                                                                            "properties": {
                                                                                                                "Duration": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                },
                                                                                                                "Offset": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                },
                                                                                                                "Word": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "required": {
                                                                                                            "items": {
                                                                                                                "type": "string"
                                                                                                            },
                                                                                                            "type": "array"
                                                                                                        },
                                                                                                        "type": {
                                                                                                            "type": "string"
                                                                                                        }
                                                                                                    },
                                                                                                    "type": "object"
                                                                                                },
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        }
                                                                                    },
                                                                                    "type": "object"
                                                                                },
                                                                                "required": {
                                                                                    "items": {
                                                                                        "type": "string"
                                                                                    },
                                                                                    "type": "array"
                                                                                },
                                                                                "type": {
                                                                                    "type": "string"
                                                                                }
                                                                            },
                                                                            "type": "object"
                                                                        },
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "Offset": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "RecognitionStatus": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "SpeakerId": {
                                                                    "properties": {},
                                                                    "type": "object"
                                                                }
                                                            },
                                                            "type": "object"
                                                        },
                                                        "required": {
                                                            "items": {
                                                                "type": "string"
                                                            },
                                                            "type": "array"
                                                        },
                                                        "type": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "type": "object"
                                                },
                                                "type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        }
                                    },
                                    "type": "object"
                                },
                                "required": {
                                    "items": {
                                        "type": "string"
                                    },
                                    "type": "array"
                                },
                                "type": {
                                    "type": "string"
                                }
                            },
                            "type": "object"
                        },
                        "type": {
                            "type": "string"
                        }
                    },
                    "type": "object"
                }
            },
            "type": "object"
        },
        "type": {
            "type": "string"
        }
    },
    "type": "object"
}

```
Renommez l'action en quelque chose comme "**Parse channel 0 results**".
Dans le champ "**Schema**", vous devez avoir quelque chose comme illustré dans la copie d'écran ci-dessous.

Renommez aussi la première condition par "**Test channel 0**".

Cliquez sur "**Add an action**" (**ATTENTION** !! Cliquez sur celui **en dehors** du test "*If true*")

![springle](Pictures/084.png)



Dans le champ de recherche, entrez le mot "**if**", puis sélectionnez "**Condition**".

![springle](Pictures/085.png)

Cliquez dans le champ "**Choose a value**". Cliquez sur "**Dynamic content**", puis sur "**channel_1**" sous "**Parse JSON 2**"

![springle](Pictures/086.png)

Puis sélectionnez "**is not equal to**" et cliquez sur "**Choose a value**" et "**Expression**"
Dans le champ "**Fx**", entrez l'expression "**null**". Cliquez sur le bouton "**OK**".


![springle](Pictures/087.png)

Dans la partie "**If true**" de la condition, cliquez sur "**Add an action**"

![springle](Pictures/088.png)


Avec l'action suivante, nous allons récupérer les informations concernant le canal numéro 1. Pour ce faire, nous avons besoin de l'URL contenant ces informations. Cette URL se trouve dans la sortie de l'action "*Parse JSON 2*"

Dans la zone de recherche, entrez "**http**" puis sélectionnez "**HTTP**".

![springle](Pictures/089.png)

Dans le champ "**Method**", sélectionnez "**GET**". Cliquez dans le champ "**URI**", puis sur "**Dynamic content**" et sélectionnez "**channel_1**" sous "**Parse JSON 2**".

Cliquez sur "**Add an action**"



![springle](Pictures/090.png)

Dans le champ de recherche, entrez "**Parse JSON**" puis cliquez sur "**Parse JSON**"


![springle](Pictures/091.png)

Cliquez sur le champ "**Content**", puis "**Dynamic content**" et sélectionnez "**Body**" sous "**Get channel 1 results**"

![springle](Pictures/092.png)

Dans le champ "**Schema**" collez le script JSON ci-dessous

```javascript

{
    "properties": {
        "properties": {
            "properties": {
                "AudioFileResults": {
                    "properties": {
                        "items": {
                            "properties": {
                                "properties": {
                                    "properties": {
                                        "AudioFileName": {
                                            "properties": {
                                                "type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        },
                                        "AudioFileUrl": {
                                            "properties": {},
                                            "type": "object"
                                        },
                                        "SegmentResults": {
                                            "properties": {
                                                "items": {
                                                    "properties": {
                                                        "properties": {
                                                            "properties": {
                                                                "ChannelNumber": {
                                                                    "properties": {},
                                                                    "type": "object"
                                                                },
                                                                "Duration": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "NBest": {
                                                                    "properties": {
                                                                        "items": {
                                                                            "properties": {
                                                                                "properties": {
                                                                                    "properties": {
                                                                                        "Confidence": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Display": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "ITN": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Lexical": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "MaskedITN": {
                                                                                            "properties": {
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Sentiment": {
                                                                                            "properties": {
                                                                                                "properties": {
                                                                                                    "properties": {
                                                                                                        "Negative": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "Neutral": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "Positive": {
                                                                                                            "properties": {
                                                                                                                "type": {
                                                                                                                    "type": "string"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        }
                                                                                                    },
                                                                                                    "type": "object"
                                                                                                },
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        },
                                                                                        "Words": {
                                                                                            "properties": {
                                                                                                "items": {
                                                                                                    "properties": {
                                                                                                        "properties": {
                                                                                                            "properties": {
                                                                                                                "Duration": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                },
                                                                                                                "Offset": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                },
                                                                                                                "Word": {
                                                                                                                    "properties": {
                                                                                                                        "type": {
                                                                                                                            "type": "string"
                                                                                                                        }
                                                                                                                    },
                                                                                                                    "type": "object"
                                                                                                                }
                                                                                                            },
                                                                                                            "type": "object"
                                                                                                        },
                                                                                                        "required": {
                                                                                                            "items": {
                                                                                                                "type": "string"
                                                                                                            },
                                                                                                            "type": "array"
                                                                                                        },
                                                                                                        "type": {
                                                                                                            "type": "string"
                                                                                                        }
                                                                                                    },
                                                                                                    "type": "object"
                                                                                                },
                                                                                                "type": {
                                                                                                    "type": "string"
                                                                                                }
                                                                                            },
                                                                                            "type": "object"
                                                                                        }
                                                                                    },
                                                                                    "type": "object"
                                                                                },
                                                                                "required": {
                                                                                    "items": {
                                                                                        "type": "string"
                                                                                    },
                                                                                    "type": "array"
                                                                                },
                                                                                "type": {
                                                                                    "type": "string"
                                                                                }
                                                                            },
                                                                            "type": "object"
                                                                        },
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "Offset": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "RecognitionStatus": {
                                                                    "properties": {
                                                                        "type": {
                                                                            "type": "string"
                                                                        }
                                                                    },
                                                                    "type": "object"
                                                                },
                                                                "SpeakerId": {
                                                                    "properties": {},
                                                                    "type": "object"
                                                                }
                                                            },
                                                            "type": "object"
                                                        },
                                                        "required": {
                                                            "items": {
                                                                "type": "string"
                                                            },
                                                            "type": "array"
                                                        },
                                                        "type": {
                                                            "type": "string"
                                                        }
                                                    },
                                                    "type": "object"
                                                },
                                                "type": {
                                                    "type": "string"
                                                }
                                            },
                                            "type": "object"
                                        }
                                    },
                                    "type": "object"
                                },
                                "required": {
                                    "items": {
                                        "type": "string"
                                    },
                                    "type": "array"
                                },
                                "type": {
                                    "type": "string"
                                }
                            },
                            "type": "object"
                        },
                        "type": {
                            "type": "string"
                        }
                    },
                    "type": "object"
                }
            },
            "type": "object"
        },
        "type": {
            "type": "string"
        }
    },
    "type": "object"
}

```
Vous pouvez renommer cette action pour plus de clarté.
Cliquez sur "**Add an action**"



![springle](Pictures/093.png)

Dans le champ de recherche, entrez "**Compose**" puis cliquez sur "**Compose**"

![springle](Pictures/094.png)

Nous allons donc préparer le fichier JSON pour ensuite l'envoyer dans la base Azure Cosmos DB. Avec l'action Compose, nous allons formater le fichier JSON en fonction de nos besoins. Je vais par exemple, rajouter un champ que je vais nommer "MyDate", afin de créer la clef de partition pour l'enregistrement des résultats dans la collection Azure Cosmos DB.

Dans le champ "**Inputs**", collez  le script JSON ci-deesous :

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
  "MyDate":"",
  "Language": "English",
  "id": ""
}
```

![springle](Pictures/095.png)


Maintenant, nous allons compléter les valeurs pour *Channel_0, Channel_1, Date, MyDate* et *id*.

Cliquez à droite de **"Channel_0":** et **juste avant** la virgule, puis rajoutez le champ "**Body**" qui se trouve dans "**Dynamic content**" sous "**Parse channel 0 results**"

![springle](Pictures/096.png)

Faîtes de même avec **"Channel_1"** mais cette fois en prenant le champ "**Body**" se trouvant sous "**Parse channel 1 results**"

![springle](Pictures/097.png)


A droite de "**AudioLink**", entre les doubles quotes (""), rajoutez "**recordingsUrl**", qui se trouve dans "**Dynamic content**" sous "**Parse JSON 2**"

![springle](Pictures/098.png)

A droite du champ **"Date:"**, entre les doubles quotes (""), rajoutez l'expression **utcNow()**. Cliquez sur le bouton "**OK**".

![springle](Pictures/099.png)

A droite du champ **"MyDate:"**, entre les doubles quotes (""), rajoutez l'expression **formatDateTime(utcNow(),'yyyy-MM-dd')**. Cliquez sur le bouton "**OK**".

![springle](Pictures/100.png)

A droite du champ **"id:"**, entre les doubles quotes (""), rajoutez l'expression **body('Parse_JSON_2')[0]['id']**. Cliquez sur le bouton "**OK**".

![springle](Pictures/101.png)

Vous devez donc obtenir quelque chose de similaire à la copie d'écran ci-dessous. Vous pouvez aussi renommer cette action pour plus de clarté. Ici, j'ai renommé cette étape "**Compose JSON document with 2 audio channels**". (*Nous utiliserons ultérieurement ce que nous venons de faire lors cette étape pour la branche "***If false***" du test du second cannal*)

Cliquez sur "**Add an action**"

![springle](Pictures/102.png)

Rajoutez une action "**Parse JSON**", 

![springle](Pictures/103.png)


Dans le champ "**Content**", Rajoutez "**Outputs**" qui se trouve sous "**Compose JSON document with 2 audio channels**"

![springle](Pictures/104.png)


Dans le champ "**Schema**", rajoutez le scripts ci-dessous 




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
        "MyDate": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```

Vous devriez avour un résultat similaire à la copie d'écran ci-dessous.

Cliquez sur "**Add an action**"

![springle](Pictures/105.png)

Maintenant que nous avons récupéré les informations pour les 2 canaux de notre fichier audio, nous allons enregistrer les résultats dans une collection Azure Cosmos DB.

Dans le champ de recherche, entrez le mot "**cosmos**", puis sélectionnez "**Create or update document**"

![springle](Pictures/106.png)

Donnez un nom à votre connexion puis sélectionnez le compte Azure Cosmos DB que vous souhaitez utiliser. Cliquez sur le bouton "**Create**"

![springle](Pictures/107.png)


Dans les listes déroulantes "**Database ID**" et "**Collection ID**", sélectionnez les informations de votre base et collection Azure Cosmos DB. Cliquez dans le champ "**Document**" puis sélectionnez "**Body**" sous "**Parse JSON 3**"

![springle](Pictures/108.png)

Cliquez dans le champ "**Add new parameter**", puis sélectionnez "**Partition key value**"


![springle](Pictures/109.png)

Cliquez dans la champ "Partition key value".
**ATTENTION !!!!** **Rajoutez les doubles quotes** pour entourer le contenu dynamique "**MyDate**", celui qui se trouve sous "**Parse JSON 3**". 


![sparkle](Pictures/110.png)

Nous allons maintenant traiter le cas des fichiers audio avec juste un seul cannal. Pour ce faire, nouys allons simplement renseigner la partie "*if false*" du test du second canal

Sous la partie "**If false**", cliquez sur "**Add an action**".

![sparkle](Pictures/111.png)

Dans le champ de recherche, entrez "**compose**", puis cliquez sur "**Compose**"

![sparkle](Pictures/112.png)

Collez le script ci-dessous dans le champ "**Inputs**"

```javascript
{
  "AudioDetails": [
    {
      "Channel_0":
      
    }
  ],
  "AudioLink": "",
  "Date": "",
  "MyDate":"",
  "Language": "English",
  "id": ""
}
```
Répétez les étapes vu précédement pour l'étape "**Compose JSON document with 2 audio channels**" mais pour un cannal seulement. Vous devez obtenir quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/113.png)

Cliquez sur "**Add an action**"

![sparkle](Pictures/114.png)

Puis dans le champ de recherche entrez "**parse**", puis sélectionnez "**Parse JSON**"

![sparkle](Pictures/115.png)

Cliquez dans le champ "**Content**" puis sur "**Dynamic content**". Séctionnez "**Ouputs**" sous "**Compose JSON document with just 1 audio channel**"

![sparkle](Pictures/116.png)

Cliquez dans le champ "**Schema**" et collez le script ci-dessous


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
        "MyDate": {
            "type": "string"
        },
        "Language": {
            "type": "string"
        },
        "id": {
            "type": "string"
        }
    },
    "type": "object"
}
```

Vous devriez obtenir quelque chose de similaire à la copie d'écran ci-dessous.
Cliquez sur "**Add an action**" 

![sparkle](Pictures/117.png)

Dans la zone de recherche, entrez "**cosmos**", puis cliquez sur "**Create or update document**" puis entrez les informations comme nous l'avons vu précédement dans cet article. 

![sparkle](Pictures/118.png)

**Attention** de prendre "**Body**" et "**Mydate**" de l'étape "**Parse JSON 4**". 
Vous devriez obtenir quelque chose de similaire à la copie d'écran ci-dessous :

![sparkle](Pictures/119.png)

Nous allons maintenant rajouter la dernière étape de notre Logic App. Cette étape consiste à supprimer les informations de transcription qui restent stockés au niveau du service cognitif afin de laisser la place libre pour la prochaine transcription.

Cliquez sur le bouton "**New step**"

![sparkle](Pictures/120.png)

Dans la zone de recherche, entrez "**http**" puis cliquez sur "**HTTP**"

![sparkle](Pictures/121.png)

Dans le champ "**Method**", sélectionnez "**DELETE**".
Cliquez dans le champ URI, puis cliquez sur "**Expression**". Collez le script ci-dessous. Cliquez sur le bouton "**OK**"


```javascript
concat('https://<your-cognitive-service-region>.cris.ai/api/speechtotext/v2.0/Transcriptions/',body('Parse_JSON')[0]['id'])
```

![sparkle](Pictures/122.png)


Remplacez "<your-cognitive-service-region"> par la region de votre service cognitif. Dans mon cas, c'est *canadacentral*

Enfin, dans le champ "**header**", rajoutez les information suivantes :

| Content-Type              |                 application/json |
| :------------------------ | -------------------------------: |
| Ocp-Apim-Subscription-Key | *la clef de votre service cognitif*|

Comme illustré ci-dessous

![sparkle](Pictures/123.png)

Cliquez sur le bouton "**Save**"

![sparkle](Pictures/124.png)

# Test de la solution

Maintenant que nous avons terminé notre logic app, nous allons nous assurer que notre service est bien dans le status "**Enabled**" 


![sparkle](Pictures/129.png)



Dans le cas contraire, cliquez sur le bouton "**Enable**"

![sparkle](Pictures/130.png)

Avec Azure Storage Explorer, déposez le fichier audio, qui se trouve dans le dossier AudioFile, dans votre compte de stockage. Votre Azure Logic App doit alors se déclencher et commencer le process de transcription du fichier audio


![sparkle](Pictures/131.png)

Si tout va bien, l'opération doit se dérouler avec succes

![sparkle](Pictures/132.png)

Cliquez sur "**Succeeded**" (ou "Failed" suivant les cas :)) afin d'avoir plus de  détails 

![sparkle](Pictures/133.png)

Allez voir le résultat dans votre collection Azure Cosmos DB.

![sparkle](Pictures/134.png)


# Power BI
Maintenant que les données du fichier sont stockées dans une base Azure Cosmos DB, il est naturel de vouloir visualiser ces données via Power BI. Je ne vais pas détailler comment faire le rapport mais juste illustrer comment démarrer. Et je vous partage aussi un rapport que j'ai fait, pour exemple.

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

Dans Power Quesry Editor, cliquez sur les flèches pour accéder aux champs de votre collection

![sparkle](Pictures/142.png)

Choisissez les champs dont vous avez besoin et cliquez sur le bouton "**OK**"

![sparkle](Pictures/143.png)

Pour les colonnes qui ont encore les doubles flèches, cliquez dessus pour choisir les champs que vous souhaitez intégrer dans le rapport. Il est possible de répéter cette opération plusieurs fois, tout dépend de la profondeur de la structure du document.

![sparkle](Pictures/144.png)

Ci-dessous, un exemple de rapport Power BI (disponible sous le dossier [*Power BI*](https://github.com/franmer2/speechtotextfr/tree/master/Power%20BI)). J'ai rajouté la possibilité de faire un "**Drillthrough**" au niveau des vues générales des transcriptions afin d'avoir le détail des conversations.

![sparkle](Pictures/902.png)


Pour représenter la discussion dans le rapport, j'ai utilisé une visualisation de type Diagramme de Gantt

![sparkle](Pictures/145.png)


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



