# Explorer les collaborations en matière de publication à l'aide des données ORCID et DOI
## Résumé du projet
Les ressources disponibles dans ce référentiel Github peuvent être utilisées pour créer une visualisation des activités de collaboration en matière de publication, sur la base des données publiques issues des dossiers ORCID des chercheurs et des métadonnées de publication DOI de Crossref et DataCite. Le script R « Rorcid_Crossref_Authors.R » de ce référentiel peut être utilisé pour récupérer des informations sur les collaborations en matière de publication entre les chercheurs d'une organisation d'origine et d'autres organisations à travers le monde. Le fichier CSV obtenu peut ensuite être chargé dans une application de tableau de bord [Shiny](https://shiny.posit.co/) afin de créer une carte des collaborations et des vues supplémentaires permettant d'explorer les données plus en détail. Ce projet a été rendu possible grâce à un partenariat conclu en 2022 entre la [communauté ORCID US](https://orcidus.lyrasis.org/) (gérée par Lyrasis) et le [programme LEADING](https://mrc.cci.drexel.edu/leading/) de l'université Drexel, et il a été prolongé en 2024 grâce à une collaboration avec la [communauté ORCID-CA](https://www.crkn-rcdr.ca/en/orcid-ca-home) (gérée par CRKN). Ces mises à jour comprenaient l'élargissement des sources de données afin de récupérer les DOI à la fois de CrossRef et de Datacite, ainsi que la transition de Tableau vers une application Shiny pour la visualisation.

## Récupération des données
Nous vous recommandons d'utiliser [R Studio](https://posit.co/) pour exécuter le [script R](https://github.com/crkn-rcdr/ORCID-Data-Visualization/blob/main/Rorcid_Crossref_Authors.R), qui générera les données nécessaires à la création de la visualisation. Le script est conçu pour :

* Récupérer les identifiants ORCID des chercheurs qui ont une affiliation professionnelle actuelle et visible publiquement pour une institution d'origine dans leur dossier ORCID.
* Décompressez les informations publiques relatives aux travaux présentes dans chaque dossier ORCID.
* Récupérer les métadonnées Crossref et DataCite DOI pour chaque œuvre dont la citation ORCID comprend un identifiant Crossref ou DataCite DOI.
* Décompresser la liste des coauteurs inclus dans les métadonnées Crossref DOI pour chaque œuvre
* Récupérer l'identifiant ORCID de chaque coauteur, s'il est disponible.
* Vérifiez l'affiliation professionnelle actuelle dans le dossier ORCID de chaque coauteur.
* Obtenir des informations sur la localisation des institutions des coauteurs 
* Reconditionner les données dans un fichier CSV contenant les identifiants ORCID des auteurs principaux, les identifiants ORCID des coauteurs, les affiliations institutionnelles/emplacements géographiques et les identifiants DOI des publications.

Vous pouvez utiliser le bouton vert « code » ci-dessus pour télécharger un fichier ZIP contenant le fichier `Rorcid_Crossref_Authors.R` ainsi qu'un dossier intitulé « data », dans lequel sera enregistré le fichier CSV obtenu. Vous pouvez également télécharger uniquement le script R et créer votre propre dossier « data » séparément.

Avant de commencer, vous devrez rassembler les informations suivantes concernant votre organisation, en fonction de l'étendue ou de la précision que vous souhaitez donner à votre recherche :

* Nom(s) de l'organisation
* Domaine(s) de messagerie électronique de l'organisation
* Identifiant ROR de l'organisation (recherchez dans le [registre ROR](https://ror.org/search))
* Identifiant GRID de l'organisation (souvent inclus comme « autre identifiant » dans le [registre ROR](https://ror.org/search))
* Identifiant(s) Ringgold de l'organisation (pour trouver le ou les identifiants Ringgold de votre organisation, vous pouvez créer un compte invité sur [le site Ringgold](https://ido.ringgold.com/register) ou envoyer un e-mail à ORCID-CA@crkn.ca afin que nous trouvions votre identifiant Ringgold pour vous)

De plus, vous devrez créer un compte GeoNames afin de récupérer les coordonnées géographiques de chaque organisation :
* [Créez un compte GeoNames](https://www.geonames.org/login) si vous n'en avez pas déjà un
* Gardez votre nom d'utilisateur à portée de main pour configurer le script ultérieurement

Pour obtenir de l'aide afin de récupérer ces informations, veuillez contacter ORCID-CA@crkn.ca.

Ouvrez le fichier `Rorcid_Crossref_Authors.R` dans RStudio. Le script contient plusieurs commentaires et instructions. Les commentaires sont indiqués par la présence d'un dièse (#) précédant le texte du commentaire. Les lignes de texte précédées d'un dièse ne seront pas exécutées en tant que commandes. Les lignes de texte sans dièse seront exécutées en tant que commande lorsqu'elles seront saisies dans R Studio. 
 
La première fois que vous exécutez le script, vous devrez effectuer quelques opérations pour le configurer. Une fois ces opérations effectuées, vous ne devriez plus avoir à les refaire :

* Installez les paquets répertoriés dans le script en décommentant ou en supprimant les dièses des commandes d'installation. Une fois installés, vous ne devriez plus avoir à réinstaller les paquets. Vous pouvez donc laisser les dièses dans les sessions futures. Cependant, vous devrez toujours charger les paquets à chaque fois que vous exécuterez le script.
* Obtenez vos [clés API publiques ORCID](https://info.orcid.org/documentation/features/public-api/) (ID client et secret client). Vous devez disposer d'un compte ORCID iD pour obtenir les clés API publiques. Si vous n'avez pas de compte ORCID iD, vous pouvez en créer un gratuitement sur [https://orcid.org/register](https://orcid.org/register). Suivez les instructions du script.
* Obtenez votre jeton porteur API ORCID dans RStudio. Suivez les instructions fournies dans le script.

Ensuite, vous allez saisir vos valeurs uniques pour lancer le processus de recherche et de récupération des données. Vous devrez saisir les valeurs suivantes (suivez les instructions du script) :

* Votre répertoire de travail = le chemin d'accès au dossier `data`  où sera stocké le fichier CSV final. Le chemin d'accès doit ressembler à ceci : `/Users/rabun/Desktop/ORCID-Data-Visualization-main/data`
* L'année à partir de laquelle vous souhaitez rechercher des publications
* Ringgold ID, GRID ID, ROR ID, domaine de courriel et nom de l'organisation : cela permet au script de commencer par rechercher tous les enregistrements ORCID qui contiennent une ou plusieurs de ces valeurs dans la section  "Emploi" des enregistrements ORCID des personnes. Par exemple : 
   * `ringgold_id <- "3427" `
   * `grid_id <- "grid.266820.8" `
   * `ror_id <- "https://ror.org/05nkf0n29"`
   * `email_domain <- "@unb.ca"`
   * `organization_name <- "University of New Brunswick"`

Notez que si vous souhaitez rechercher plusieurs noms d'organisations (et donc plusieurs identifiants différents) et plusieurs domaines de messagerie, une section du script vous permet de définir plusieurs valeurs pour la recherche (voir ci-dessous).

* Keyword = un mot propre à votre établissement, qui servira à affiner les résultats de recherche pour votre organisation uniquement. Par exemple, « Temple » pourrait être le mot-clé si vous recherchez des résultats provenant de l'université Temple. Si le nom de votre établissement contient des mots courants, vous pouvez utiliser le nom complet de l'organisation comme keyword. Par exemple, le keyword « New » ne serait pas utile pour une recherche sur « The New School », car plusieurs organisations ont le mot « New » dans leur nom.
* Informations géographiques sur votre organisation, notamment la ville, l'État et le pays. Par exemple :

   * `anchor_org<-"University of New Brunswick"`
   * `anchor_city<-"Fredericton"`
   * `anchor_region<-"New Brunswick"`
   * `anchor_country<-"Canada"`
* GeoNames username = la nom d'utilisateur de votre compte de GeoNames que vous avez deja utilisé.

A cette étape, vous avez deux options pour crée la premiere commande.
1) Vous pouvez utiliser les values que vous avez deja entré pour chercher.
2) Vous pouvez chercher pour plusieurs departements ou campus a votre organization si les autres valures était entrer.

Maintenant la scripte peut-être executer la reste de scripte avec les instructions qui se trouve au commentaire dans la scripte.

NOTEZ: La scripte se compose de plusieurs section. Par consequence, si vous avez telechargé les donnés a chaque étape, vous pourriez l'utiliser encore pour en avoir la chance de ne pas recommencer au début de la scripte chaque fois. Cela vous aiderez si quelque chose vous interrompte ou s'il n'y a pas assez de temps pour executer la scripte en entiere dans une journee.

NOTEZ: Il y a plus qu'une partie de scripte, au section « *get employment data* ». Vous avez trois options.
1) Utilise tous les noms des organizations retourner par la scripte.
2) Visualize et change la liste des organization qui peut-être inclus dans les resultats.
3) Utilise tous les noms des organizations qui se countient `anchor_org`.

Si vous continue avec les instructions et execute les commandes dans la scripte, juste-qu'au fin, la fichier CSV complete sera remplis dans votre dossier des donnés.
Quand vous avez la fichier de CSV, if faut que les noms des organization et les villes. Vous pouvez utiliser la project [Open Refine](https://openrefine.org/). Cela peut vous aiderez pour reparer les noms qui était écris incorrecte ou se contiens des espaces extra, et aussie pour remplire l'information des villes qui peut-être incorrecte, ou meme pas presente.

## Des Considerations et Contexte pour les Donnés
**Les Erreurs des Donnés:** 
Les donnés que vous avez commandé avez la scripte `Rorcid_Crossref_Authors.R` sont pas toujours correcte. Les numeros dans les donnés ne sont pas definitif. Les donnés que vous avez commandé pour votre organization sont une visualization durant une periode de temps dans laquelle vous avez executé la scripte. Par consequence, les donnés peut changer si les profiles d'ORCID change avec les rechercheurs qui se continue avec leurs publication.

Des examples des eurreurs qui peut-être dans votres donnés:
* Une profile d'ORCID qui se manque
* Les donnés geographique qui manque, et donc des donnés qui manque dans la carte des collaborations.
* Des coquille dans les noms des organization ou les villes qui se produit les ORCIDs incorrecte inclut dans les donnés.

Nous voulons souligner que les donnés ne sont pas pour evaluer ou comparer les rechercheurs contre des autres parce-que ce n'est pas parfaite comme une representation d'un(e) rechercheur(e). Par consequence, ce n'est pas une visualization de l'impacte. Les resources dans c'ette dossier c'est juste une facon d'approcher l'information d'ORCID.

**Calculer les Collaboration:**
Dans la commande des donnés, les collaborations sont calculé par evaluer chaque auteur(e) pour leur(e) collaborations, les resultats sont calculé encore. Les collaborations d'une publication ne sont pas compté plus-qu'une fois par intitution. Par example, si 2 rechercheurs a l'universite de Nouveau Brunswick avez ecrit une papier avec un(e) rechercheur(e) de l'universite de Toronto, cela sera *une* collaboration pour l'UNB et *une* pour l'UofT.

**Les organizations du passe et present:**
Les données extraites pour chaque auteur prennent également en compte l'ensemble de leur carrière. Le script extrait également l'institution actuelle des auteurs collaborateurs. Cela réduit les champs vides, qui sont plus nombreux lorsqu'on tente de déterminer l'affiliation au moment de la création du DOI en raison du manque d'entrées historiques relatives à l'emploi dans les profils ORCID. Cela évite également les divergences potentielles entre la date de création du DOI et la date de publication, qui est parfois vide. Cela permet également de traiter les deux auteurs de la même manière en termes de comptage.

**Les Dates:**
Pour l'onglet « Recherche individuelle » du tableau de bord Tableau, si vous ou une autre personne rencontrez des difficultés pour trouver votre/leur identifiant ORCID dans l'extraction de données ou la recherche, voici quelques éléments à vérifier :
## Crée Votre Propre Affiche de Shiny 
Une fois que vous avez obtenu le fichier CSV correspondant à votre recherche, vous pouvez charger vos données dans l'application Shiny afin de créer votre visualisation. Reportez-vous à la page [Personnalisation de votre propre tableau de bord Shiny](https://github.com/crkn-rcdr/ORCID-Data-Visualization/blob/43f8f2924059bf9b6f84f7da4a715fb40eea0a95/shiny-visualization/customizing-shiny-dashboard.md). N'oubliez pas de consulter également la [documentation sur le tableau de bord](https://github.com/crkn-rcdr/ORCID-Data-Visualization/blob/main/shiny-visualization/shiny-dashboard-documentation.md). 
## «Je ne peux pas trouver ma profile d'ORCID, pourquoi? »
Pour l'onglet « Recherche individuelle » du tableau de bord Tableau, si vous ou une autre personne rencontrez des difficultés pour trouver votre/leur identifiant ORCID dans l'extraction de données ou la recherche, voici quelques éléments à vérifier:

## 1. Avez-vous une profile d'ORCID?
Si vous n'avez pas encore créé d'identifiant ORCID, rendez-vous sur www.orcid.org pour créer votre profil ORCID. ORCID attribue un identifiant numérique permanent (également appelé identifiant ORCID) afin de vous distinguer des autres chercheurs.
## 2. Avez-vous etabli votre profile d'ORCID *apres* la commande des donnés?
Si vous avez créé votre identifiant ORCID après la collecte des données utilisées pour alimenter ce tableau de bord, votre identifiant ORCID n'apparaîtra pas dans le tableau de bord tant que les données n'auront pas été collectées à nouveau.
## 3. Est-ce que l'information dans votre profile d'ORCID est correcte?
Prenez le temps de vérifier que votre institution et votre lieu de travail actuels sont correctement indiqués dans votre profil ORCID — les fautes de frappe sont fréquentes ! Si vous travaillez à distance pour une institution, vous devrez indiquer le lieu principal de celle-ci afin d'apparaître dans les données. Si vous corrigez des informations dans votre profil ORCID après l'extraction des données utilisées pour ce tableau de bord, votre identifiant ORCID n'apparaîtra pas dans le tableau de bord tant que les données n'auront pas été extraites à nouveau.
## 4. Avez-vous collaborer avec des collaborations depuis la commande des donnés?
Ce tableau de bord ne comprend que les collaborations à des articles réalisées pendant la période de collecte des données. Si vous n'avez réalisé aucune collaboration à des articles pendant cette période, votre identifiant ORCID n'apparaîtra pas dans la liste des collaborations.
## 5. Contecter quelqu'un pour vous aider:
Parlez-vous avec votre administrateur d'ORCID de votre campus ou RCDC pour de l'aide supplementaire.

## Les resources pour Shiny
* [Shiny Get Started Tutorials](https://shiny.posit.co/r/getstarted/shiny-basics/lesson1/)
* [Shiny Cheatsheet](https://shiny.posit.co/r/articles/start/cheatsheet/)
* [Shiny Community Forums](https://forum.posit.co/c/shiny/8)
* [Shiny Getting Help Page](https://shiny.posit.co/r/help.html)
* [A11Y project checklist](https://www.a11yproject.com/checklist/)
* [Accessibility (a11y) Tooling for Shiny](https://github.com/ewenme/shinya11y)
* [Financial Times Visual Vocabulary](https://github.com/Financial-Times/chart-doctor/blob/main/visual-vocabulary/README.md)

## Questions et supporte
Pour demander les questions, commander de l'aide, ou pour donner son avis, veuillez contacter le service d'assistance communautaire CRKN ORCID-CA à l'adresse ORCID-CA@crkn.ca.

## Licence d'utilisation
Ce référentiel, [ORCID Data Visualization](https://github.com/crkn-rcdr/ORCID-Data-Visualization), © 2024 par [CRKN](https://www.crkn-rcdr.ca/en/orcid-ca-home), est sous licence [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/?ref=chooser-v1).

Ce projet est un dérivé de [Collaboration Data Visualization](https://github.com/lyrasis/ORCID-Data-Visualization) © 2022 par [Lyrasis](https://orcidus.lyrasis.org/data-visualization/), utilisé sous licence [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/?ref=chooser-v1). Les modifications et ajouts ont été apportés par [CRKN](https://www.crkn-rcdr.ca/en/orcid-ca-home). 
