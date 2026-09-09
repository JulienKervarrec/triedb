# Chapitre 1 -- Presentation de triedb

TrieDB (nom de code TBD) est une base de donnees embarquee ecrite en Rust,
concue pour un seul objectif : stocker le trie d etat d Ethereum (comptes et
slots de storage) de maniere beaucoup plus rapide que les bases Cle/Valeur
generiques traditionnellement utilisees par les clients d execution (LSM-tree
comme LevelDB chez geth, B-tree comme MDBX chez Reth).

Le probleme que le README documente precisement est le suivant. Lire une
valeur du trie d etat (un compte ou un slot) demande de traverser le trie de
la racine vers la feuille, ce qui coute en theorie O(log N) acces disque. Mais
si chaque noeud du trie est lui-meme stocke comme un enregistrement
independant dans une base Cle/Valeur generique, chaque acces a un noeud coute
lui-meme O(log N) (l index interne de la base doit etre recherche), donnant un
cout total reel de O(log N * log N). A l echelle d Ethereum ou de Base, cela
represente environ 50 operations disque pour lire ou ecrire un seul compte ou
slot.

TrieDB choisit une approche differente : au lieu de traiter chaque noeud comme
independant, elle organise le disque selon la structure meme du trie. Des
noeuds relies (des sous-tries) sont regroupes dans des Pages de 4 Ko, et
trouver un enfant ne demande que de suivre un pointeur direct vers la page
suivante. Cette disposition ramene le cout theorique a O(log N), et en
pratique a environ 8 operations disque par acces (voire ~4 lorsque plusieurs
niveaux du trie tiennent dans une meme page), soit un facteur 10 de gain de
bande passante et de latence.

Pour permettre des lectures et ecritures simultanees (execution EVM parallele,
lecture d etat pendant que le bloc suivant s execute), TrieDB implemente le
controle de concurrence multiversion (MVCC) via un schema de copie sur
ecriture (Copy on Write), sans avoir besoin de journal d ecriture anticipee
(Write Ahead Log).
