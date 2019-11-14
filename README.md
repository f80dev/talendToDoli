# TalendToDolibarr
Ce projet illustre la mise en oeuvre d'un ETL pour alimenter et manipuler les données
du CRM Dolibarr.

Pour la présentation des outils, on peut se référer aux pages web les concernants
- Dolibarr : https://www.dolibarr.fr/
- Talend : https://fr.talend.com/products/data-integration/data-integration-open-studio/

# Objectifs
Dolibarr est un CRM gratuit et open source relativement complet. Pour autant, il n'est pas aussi
complet que d'autres sur 
- l'importation de données issue de bases externes : https://wiki.dolibarr.org/index.php/Module_Imports
- la gestion de lots en interraction avec le système de taggage notamment pour faire des mailing : https://wiki.dolibarr.org/index.php/Module_Mailing

Ainsi grâce à l'ETL Talend, nous allons illustrer dans cet article, la mise en oeuvre d'un projet de mailling de 
masse à un ensemble de prospect importés d'une base SQL dans Dolibarr

Le processus va être décomposé en 4 tâches
- importer une liste d'adresse mail dans Dolibarr sous forme de Tiers,
- tagger les tiers importés suivant une information présente dans la base,
- utiliser les tags des tiers pour constituer une mailing list
- après envoi du mail, changer le statut de communication des tiers contacté

L'ensemble du code est disponible sur le dépôt github : https://github.com/f80dev/TalendToDolibarr

# Notation 
Dans la suite de l'article on utilise :
- <serveur_dolibarr> pour désigner l'adresse de votre serveur hébergeant dolibarr

# Fabrication des jobs
Sous l'Open Data Studio de Talend, les différents processus se retrouvent sous forme de "Job". Ainsi pour chacunes
des tâches décrites précédemment on va faire correspondre un job.

Chaque job met en oeuvre un workflow de composants. Ainsi on aura toujours un ou plusieurs composants représentant :
- la source des données
- le résultat final (qui peut être un traitement, un fichier ...)
- des étapes intermédiaires de transformation des données

Pour communiquer avec Dolibarr on utilise le composant tRest qui permet d'utiliser depuis Talend les API
mises à disposition par Dolibarr

# Communication Talend <=> Dolibarr
La communication entre les deux outils est possible grâce à l'usage des API dolibarr.
Nous n'utilisons qu'un petit nombre d'API pour réaliser notre projet. 
L'ensemble des API est disponible à l'adresse : http://<serveur_dolibar>/dolibarr/api/index.php/explorer/#/

Les API vont manipuler 2 principaux types d'objets qui vont être fabriquer par l'ETL (via le composant tXMLMap) 
avant d'être injecté via
les modules tRest : 
- Les "categories" (étiquettes applicable aux objets de Dolibarr) : https://wiki.dolibarr.org/index.php/Module_Cat%C3%A9gories
- les "thirdparties" correspondant aux tiers : https://wiki.dolibarr.org/index.php/Module_Third_Parties

###L'objet "thirdpartie"
Pour créer un objet "Thirpartie" on initialise certains champs. On détaille 
ci-dessous les plus importants (pour les autres il suffit de consulter la documentation du job)
- name : nom du tiers
- email : adresse email du tiers
- website : adresse internet rattaché au tiers
- address : adresse postale du tiers
- ref_ext : sera une référence de notre tiers construite sur une clé unique pour éviter les doublons
- prospect : initialisé à 0 car nos tiers sont tous des prospects à l'importation
- facebook : contient l'url de la page facebook du tiers
- note_private : va être utiliser comme tampon temporaire pour inscrire certaines informations présentent 
dans la base qui seront converties en "tag" dans le job suivant
- statut_commercial : initialisé à "To contact" puisque l'objectif est de faire un mailing


###L'objet "categorie" :
- label : contient le nom de la catégorie
- color : va permettre de lui donner une couleur particulière, cela peut être utile lorsque le nombre de catégorie devient
important
- type : initialisé à 2 pour dire que la catégorie s'applique à nos clients (et non aux fournisseurs)
- ref : pour la référence de l'objet on utilise son nom (permettant ainsi d'éviter les doublons)
- fk_parent : initialisé à 0 car nos catégorie non pas d'antécédent


# Détail des jobs

- importation des tiers comme prospect :
https://talendtodolibarr.f80.fr/ProspectToDolibarAsTiers_0.2/ProspectToDolibarAsTiers/ProspectToDolibarAsTiers_0.2.html


