# Chapitre 3 -- L encodage binaire des noeuds (src/node.rs)

Le fichier `src/node.rs` implemente exactement le format decrit dans le
README pour les trois types de noeuds. Un `Node` est un `Box<NodeInner>`
contenant un `prefix: Nibbles` (le chemin partage, 0 a 64 nibbles) et un
`NodeKind`, qui est soit `AccountLeaf` (nonce_rlp, balance_rlp, code_hash,
storage_root optionnel), soit `StorageLeaf` (value_rlp), soit `Branch`
(tableau de 16 `Option<Pointer>`).

`serialize_into` (impl du trait `Value`) ecrit le format binaire compact.
Pour un `StorageLeaf` : octet 0 = type (0), octet 1 = longueur du prefixe en
nibbles, puis le prefixe empaquete, puis la valeur RLP -- teste
explicitement dans `test_storage_leaf_node_serialize` (ex :
`0x0002ab83040506` pour le prefixe `[0xa, 0xb]` et la valeur `[4,5,6]`). Pour
un `AccountLeaf`, un octet de flags combine le bit de type (bit 0), le bit
"a du storage" (bit 7) et le bit "a du code" (bit 6, teste par
`code_hash != KECCAK_EMPTY`), suivi du nonce et du solde RLP, puis
optionnellement le hash de code (32 octets) et le pointeur vers la racine de
storage (37 octets, uniquement si present). Pour un `Branch`, la fonction
`children_slot_size` calcule un nombre de slots parmi {2, 4, 8, 16} selon le
nombre reel d enfants (`next_power_of_two`, minimum 2) -- exactement le
mecanisme de "branching factor variable" annonce par le README pour eviter
que l ajout d un enfant ne force une reallocation a chaque fois. Le bitmask de
16 bits (`children_bitmask`) indique quels indices sont occupes.

Chaque pointeur d enfant occupe 37 octets : 33 pour le RLP (hash prefixe) et 4
pour la localisation. `size_incr_with_new_child` calcule le cout exact en
octets d ajouter un enfant a un noeud existant -- utilise ailleurs dans le
moteur pour decider si une page doit etre eclatee (split) avant l insertion.

`Node` implemente separement `Encodable` (le trait RLP d alloy-rlp) pour
produire l encodage RLP canonique utilise dans le calcul du hash Merkle
(`to_rlp_node`), distinct de son encodage binaire compact sur disque. Le
README l annonce explicitement : le state root reste defini par le RLP
standard, mais le stockage physique utilise un format personnalise, plus
dense, uniquement converti en RLP au moment de hacher.
