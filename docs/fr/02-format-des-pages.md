# Chapitre 2 -- Le format des pages sur disque

Toute donnee de TrieDB est decoupee en pages de exactement 4 Ko, l unite
minimale d entree/sortie disque sur un SSD moderne. Les pages 0 et 1 sont
reservees aux deux Root Pages (page racine courante et precedente) qui
portent les metadonnees globales de la base ; les pages numero 256 et plus
stockent les Subtrie Pages, chacune contenant un sous-arbre complet de
Branch, Account et Storage nodes.

Une Root Page contient : un Snapshot ID (8 octets, compteur auto-incremente
commencant a 1 -- la Root Page ayant le plus grand Snapshot ID est la version
active), le State Root (32 octets, hash de la racine du sous-trie), le numero
de la Subtrie Page racine, le Max Page Number (dernier numero de page
reference, utilise pour tronquer apres un arret brutal non propre), et une
liste chainee de pages orphelines terminee par zero. Le README precise
explicitement que ces pages orphelines ne sont jamais relues apres
l initialisation -- leur format n a donc pas besoin d etre optimise pour la
lecture.

Une Subtrie Page utilise un format de Slotted Page : un en-tete (Snapshot ID),
un compteur du nombre de Cells (1 octet, 255 max), puis une liste de Pointeurs
de 3 octets chacun (12 bits d offset depuis la fin de la page + 12 bits de
taille de la Cell), et enfin les Cells elles-memes, ecrites a l envers depuis
la fin de la page. Un pointeur entierement a zero est une "tombstone" -- une
Cell supprimee. L ordre des Pointeurs determine l ordre logique des noeuds
dans la page, ce qui permet de reordonner des noeuds sans jamais reecrire le
contenu des Cells elles-memes -- une optimisation directe pour minimiser le
travail de serialisation lors d une modification.

Ce choix de conception (mettre a jour une Cell individuelle sans reserialiser
toute la page) est cite dans le README comme la raison principale pour
laquelle TrieDB peut se permettre le Copy on Write sans amplification
d ecriture excessive : dans un MPT (Merkle Patricia Trie), toute modification
d un sous-trie change de toute facon le hash de tous ses ancetres jusqu a la
racine, donc reecrire les pages ancetres est deja necessaire independamment
du format de stockage choisi.
