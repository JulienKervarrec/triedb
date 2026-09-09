# Parcours francais : triedb (base de donnees pour le trie d etat)

Lecture pedagogique du depot base/triedb : une base de donnees embarquee en Rust qui stocke le trie d etat Ethereum sous forme de Pages de 4 Ko organisees selon la structure du trie, avec MVCC par copie sur ecriture.

Sommaire :

Chapitre 1 Presentation de triedb. Chapitre 2 Le format des pages sur disque. Chapitre 3 L encodage binaire des noeuds (src/node.rs). Chapitre 4 RawPath, AddressPath, StoragePath (src/path.rs). Chapitre 5 Database, Transaction et le cycle MVCC (src/database.rs, src/transaction.rs). Chapitre 6 Limites et perimetre de ce parcours.

Ce parcours est une lecture pedagogique du code source et de la documentation du depot, sans installation ni execution du projet.
