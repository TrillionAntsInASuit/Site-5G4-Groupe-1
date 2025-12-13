+++
title = "Notes de cours : Voice-to-Text"
weight = 2
+++

## Introduction
Voice to text

C'est quoi?

Voice to text (ou reconnaissance automatique de la parole) est le processus qui analyse et convertit les mots qui sont prononcés oralement en texte lisible par une machine.

Comment ça fonctionne?

Voice to text fonctionne en trois étapes:
    1. Enregistrement de la parole:
        La langue parlée est capturée par le système par une source audio. Par exemple, un microphone.
    2. Traitement de la parole:
        L'enregistrement vocal est nettoyé des bruits indésirables dans l'audio. Un algorithme analyse les caractéristiques phonétiques et phonémiques de la parole pour identifier des mots individuels en comparant ces caractéristiques avec celles des autres modèles préalablement entraînés.
    3. Génération de de texte:
        Le système convertit les sons reconnus en texte.

![ImageFonction](../../images/fonction.jpg)

La reconnaissance vocale est souvent traitée dans le middleware.

Humain -> Hardware(Input) -> Middleware -> Hardware(Output) -> Humain

![ImageMiddleware](../../images/middleware.png)

Example: On parle(humain) dans un microphone(input) et ça transforme en texte(middleware) sur notre écran(output) qu'on voit(humain).

## Taux d'Erreur de Mots (TEM)

La reconnaissance automatique de la parole n'est pas parfaite. Le Taux d'Erreur de Mots (TEM) indique la précision du système automatique de la parole. Il compare les erreurs comme les mots omis, ajoutés et mal reconnus avec le nombre de mots prononcés. Plus cette valeur est basse, plus la précision du système de reconnaissance automatique de la parole est élevée. Par exemple, si on a un taux d'erreur de 14%, la précision de la transcription est de 86%.

La formule pour le trouver est WER = S+E+I\N.
WER signifie Word Error Rate ce qui est la version anglaise du Taux d'Erreur de Mots.
S signifie le nombre de substitutions, E signifie le nombre d'élisions, I signifie le nombre d'insertions.
N signifie le nombre de mots dans la transcription de référence. Il est le nombre de mots qu'on est supposé avoir si le système de reconnaissance de la parole ne fait aucune erreur.

## Histoire du reconnaissance automatique de la parole

La reconnaissance automatique de la parole a une histoire d'évolution rapide, mais chaque histoire a un début.

Les travaux sur la reconnaissance automatique de la parole ont commencé vers le début du 20<sup>e</sup> siècle. Le premier système capable de faire de la reconnaissance de la parole était dans l'année 1952.

Développé dans les laboratoires Bell par Davis, Biddulph et Balashek. Le système était composé de relais et il était seulement capable de reconnaître des chiffres isolés. Durant les années 1970, la recherche a avancé rapidement avec l'aide des travaux de Jelinek chez IBM. Finalement en 1972, la première commercialisation d'un système de reconnaissance automatique de la parole a été faite par la société Treshold Technologies. Ce système s'appelle VIP100. À cause des systèmes embarqués, aujourd'hui, la reconnaissance de la parole connait une croissance forte.

Marques importantes dans son évolution:

En 1952, on a été capable de faire l'identification de dix chiffres avec un système électronique câblé.

En 1960, on a commencé à utiliser des méthodes numériques dans la reconnaissance de la parole.

En 1965, on a pu faire la reconnaissance de phonèmes en parole continue.

En 1968, on a réussi à faire la reconnaissance de mots isolés avec des systèmes mis sur les gros ordinateurs. On a été capable de reconnaître jusqu'à 500 mots.

En 1970, Leonard E. Baum crée le modèle caché de Markov.

En 1971, aux États-Unis, il y a le lancement du projet ARPA. Ceci a couté 15 millions de dollars et a servi à tester la faisabilité de la compréhension automatique de la parole continue dans des conditions raisonnables.

En 1972, il y a eu le lancement du premier appareil capable de faire la reconnaissance de mots.

En 1978, il y a eu la commercialisation d'un système de reconnaissance de la parole à microprocesseur sur une carte de circuits imprimés.

En 1983, il y a eu la première utilisation au monde d'un système de commande vocale dans un avion de chasse en France.

En 1985, il y a eu la commercialisation des premiers systèmes de reconnaissance de la parole capables de reconnaître plusieurs milliers de mots.

En 1986, il y a eu le lancement du projet japonais ATR de téléphone. Ceci avait la capacité de faire la traduction automatique en temps réel.

En 1993, il y a eu le projet SUNDIAL.

En 1997, il y a le lancement du premier logiciel de dictée vocale par la société Dragon. Le nom de ce logiciel de dictée vocale est NaturallySpeaking.

En 2008, en utilisant la reconnaissance automatique de la parole, Google lance une application de recherche sur Internet.

En 2011, Apple met Siri, qui utilise la reconnaissance automatique de la parole, dans ses téléphones.

En 2017, Microsoft réussit à atteindre un niveau de reconnaissance vocale comparable à celui des êtres humains.

En 2019, Amazon introduit la reconnaissance automatique de la parole dans les consultations médicales.

En 2023, Nabla devient capable de faire la transcription et la synthèse d'une consultation. Avec le résultat de consultation, il peut ensuite faire la classification avec le CISP2 et le CIM-10.

Depuis 2024, il y a une augmentation d'utilisation d'intelligence artificielle dans de nombreux logiciels de transcriptions.

## Modèles

Un système de reconnaissance automatique de la parole s'appuie sur trois modèles principaux. Il utilise le modèle de langage, le modèle de prononciation et le modèle acoustico-phonétique.

Modèle de langage:
    On donne la probabilité, on utilise P(W) pour la probabilité, de chaque suite de mots, on utilise W pour la suite de mots, dans le langage cible.
Modèle de prononciation:
    Pour chaque suite de mots W, on utilise les prononciations possibles, on utilise H pour les prononciations possibles, avec les probabilités P(H|W).
Modèle acoustico-phonétique:
    On estime la probabilité P(X|H) de la séquence observée de vecteurs acoustiques, on utilise X pour les vecteurs acoustiques, en fonction d'une prononciation possible H d'une séquence de mots donnée.

Avec ces trois modèles, on devient capable de calculer la probabilité d'une suite de mots correspondant à un signal vocal observé. Pour faire la reconnaissance de la parole, il faut trouver la suite de mots qui a la probabilité la plus élevée. Pour faire ceci, il faut trouver la suite de mots W qui maximise cette expression mathématique: P(H)∑<sub>H</sub>P(H|W)P(X|H)

Pour utiliser ces modèles pour une application, il faut utiliser une grande quantité de corpus annotée. Ceci doit correspondre aux conditions d'utilisation prévues pour le système.

## Méthodes de reconnaissance de la parole

Il y a trois méthodes pour la reconnaissance de la parole. Il y a la reconnaissance synchrone, la reconnaissance en continu et la reconnaissance asynchrone.

Pour la reconnaissance synchrone, on convertit immédiatement la parole en texte. Par contre, cette méthode peut seulement être utilisée pour des fichiers audio avec une durée inférieure à une minute. Cette méthode est utilisée pour le sous-titrage en direct à la télévision.

Pour la reconnaissance en continu, on traite les fichiers audio en temps réel. Des textes fragmentés peuvent apparaître pendant que quelqu'un parle.

Pour la reconnaissance asynchrone, on traite les fichiers audio préenregistrés de grande taille. Ces fichiers audio sont en vue de leur transcription et peuvent être mis en attente pour traitement et restitution ultérieure.

## Classification des systèmes de reconnaissance de la parole

Les systèmes de reconnaissance de la parole sont classifiés selon certains critères. Ces critères sont le type de signal, le type de modèle acoustique, la nature des enregistrements et la langue.

Le type de signal:
    Est-ce que c'est un signal bruité ou non bruité, un signal téléphonique, une large bande, un signal compressé ou non. Un exemple d'un signal non bruité est un microphone casque avec réduction de bruit et un exemple pour un signal téléphonique est un téléphone fixe ou mobile.
Le type de modèle acoustique:
    Est-ce que c'est un modèle monolocuteur ou un modèle multilocuteur.
La nature des enregistrements:
    Est-ce que l'enregistrement est une dictée de texte, un dialogue homme-machine, une commande vocale, un message téléphonique, etc.
La langue:
    Les langues comme le français, l'anglais, l'allemand, etc.

La nature des enregistrements et la langue ont un lien direct avec la taille du vocabulaire et la complexité du modèle de langage. Par exemple, des commandes vocales prendront environ quelques dizaines de mots et des langues comme le français auront besoin de quelques centaines de milliers de mots.

## Champs d'application de la reconnaissance automatique de la parole

Grâce aux avancements dans le domaine du Machine Learning, les systèmes de reconnaissance automatique de la parole deviennent plus précis et performants. Plusieurs secteurs utilisent cette technologie pour améliorer l'efficacité, la satisfaction de leurs clients et d'autres aspects.

Par contre, il y a cinq domaines qui utilisent la reconnaissance automatique de la parole le plus. Les télécommunications, les plateformes vidéo, la surveillance des médias, les vidéoconférences et les assistants vocaux sont les domaines principaux.

Télécommunications:
    Les centres de contact utilisent la reconnaissance automatique de la parole pour la transcription et l'analyse des conversations avec les clients. Il faut des transcriptions exactes pour faire le suivi des appels et les solutions téléphoniques qui utilisent les serveurs Cloud.
Plateformes vidéo:
    Les plateformes vidéo utilisent la reconnaissance automatique de la parole pour la création des sous-tires en temps réel. Ils utilisent aussi la reconnaissance de la parole pour la catégorisation des contenus.
Surveillance des médias:
    Les API de la reconnaissance automatique de la parole permettent d'analyser plusieurs médias comme les émissions de télévision, les podcasts et les émissions de radio. Ils permettent de trouver la fréquence d'apparition de mentions de marques ou de thèmes.
Vidéoconférences:
    Les applications qui permettent la réunion comme Zoom, Microsoft Teams et Google Meet ont besoin de la transcription exacte et de l'analyse de transcription pour obtenir des informations clés et prendre des mesures appropriées. La reconnaissance automatique de la parole permet aussi de faire les sous-titres en direct pour les vidéoconférences.
Assistants vocaux:
    Les assistants vocaux comme Amazon Alexa, Google Assistant et Siri d'Apple sont basés sur la reconnaissance automatique de la parole. On peut répondre aux questions, faire des tâches et interagir avec d'autres appareils.

## Points forts et faibles de la reconnaissance automatique de la parole

La reconnaissance automatique de la parole présente des avantages et des inconvénients comparée à la transcription traditionnelle.

Les points forts:

    Un des avantages principaux est la précision très forte de la reconnaissance automatique de la parole. Ceci est parce qu'on peut les entrainer avec beaucoup de données. Ceci permet d'améliorer la qualité des sous-titres ou des transcriptions et de les fournir en temps réel.

    Un autre avantage est qu'on peut augmenter l'efficacité de plusieurs choses avec la reconnaissance automatique de la parole. Il permet aux entreprises d'augmenter le nombre de leurs services et de les proposer à un plus grand nombre de clients. Dans un contexte scolaire ou professionnel, on peut documenter des cours et des réunions, ce qui augmente la rapidité de prise de notes.

    La reconnaissance automatique de la parole donne un avantage pour les personnes qui sont handicapées. Ils peuvent écrire du texte sur leur ordinateur ou leur cellulaire sans avoir besoin de saisir du texte physiquement. Au lieu de taper leurs textes physiquement, ils peuvent écrire des messages textuels, des notes, des e-mails et beaucoup plus avec juste leur voix. Par exemple, un étudiant qui souffre de dyslexie ou d'une blessure qui affecte son bras ne peut pas écrire de façon physique facilement, mais avec un ordinateur Microsoft ils peuvent écrire des textes avec seulement leur voix. Ceci est dû au fait que l'ordinateur Microsoft contient les services d'Azure Speech. 

Les points faibles:

    Un inconvénient est que même si la précision de la reconnaissance vocale automatique est très forte, il ne réussit pas à atteindre la précision des humains. Ceci est parce que les paroles contiennent la nuance. Ils contiennent des accents, des dialectes, des tonalités différentes et des bruits parasites. Même si la reconnaissance automatique de la parole est bonne, il ne peut pas complètement éliminer ces variables.

    Un autre inconvénient pour la reconnaissance automatique de la parole sont les données personnelles. Il peut avoir des inquiétudes pour la vie privée et la sécurité des données.

## Source
- https://www.ionos.fr/digitalguide/sites-internet/developpement-web/automatic-speech-recognition/
- https://www.ibm.com/fr-fr/think/topics/speech-to-text
- https://fr.wikipedia.org/wiki/Reconnaissance_automatique_de_la_parole