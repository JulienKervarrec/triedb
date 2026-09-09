# Chapitre 6 -- Limites et perimetre de ce parcours

Ce parcours couvre le README (justification architecturale complete et
format des pages), `src/node.rs` (encodage binaire et RLP des trois types de
noeuds), `src/path.rs` (RawPath, AddressPath, StoragePath) et l API publique
de haut niveau exposee par `src/database.rs` et `src/transaction.rs`
(ouverture, MVCC via begin_ro/begin_rw, commit atomique).

Sont volontairement laisses hors champ : le contenu detaille du module
`storage/` (le `StorageEngine` lui-meme, sa logique de recherche et
d insertion dans le trie, le split de pages, le calcul du state root avec
overlay) ainsi que ses sous-modules `overlay_root` et `proofs` ; le module
`page/` (le `PageManager`, la gestion physique des pages, les reservations
atomiques mentionnees dans l historique des commits) ; le module `meta/`
(le `MetadataManager` et la gestion des Root Pages) ; le module `executor/`
(le pool de threads dedie) ; les benchmarks (`benches/`) et l outil en ligne
de commande (`cli/`) ; ainsi que `pointer.rs` (la structure exacte de
`Pointer`, au-dela de son role de reference 37 octets deja decrit).

L objectif reste le meme que pour les parcours precedents de cette
bibliotheque : comprendre precisement comment triedb transforme le trie
d etat Ethereum, normalement stocke dans une base Cle/Valeur generique, en
une structure de pages de 4 Ko organisee selon la topologie meme du trie,
avec un MVCC par copie sur ecriture, sans pretendre couvrir l integralite du
moteur de stockage sous-jacent.
