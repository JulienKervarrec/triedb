# Chapitre 4 -- RawPath, AddressPath, StoragePath (src/path.rs)

`src/path.rs` definit la representation des chemins dans le trie. `RawPath`
est un tableau fixe de 128 nibbles (`[u8; 128]` + une longueur), pense comme
une extension de `Nibbles` (alloy-trie, limite a 64 nibbles) capable de porter
soit un chemin de compte (64 nibbles = keccak256 d une adresse), soit un
chemin complet de storage (128 nibbles = 64 pour l adresse + 64 pour le
slot). Les methodes `push`, `pop`, `extend`, `join`, `slice`,
`common_prefix_length` et `starts_with` fournissent les operations de base
necessaires a l algorithme de trie (recherche de prefixe commun entre deux
chemins pour decider ou brancher).

`pack::<N>()` et `unpack()` convertissent entre nibbles et octets bruts --
c est ce mecanisme qui alimente directement le format de serialisation vu au
chapitre 3 (`prefix.pack_to(...)` dans `serialize_into`). Deux methodes sont
particulierement revelatrices de l architecture a deux etages du trie :
`lower_64_nibbles()` retourne les 64 premiers nibbles (la portion "compte") et
`higher_64_nibbles()` retourne le reste (la portion "storage"), ce qui
correspond exactement a la remarque du README : un compte a toujours un
chemin de 32 octets exactement, donc son type peut etre determine par la
seule longueur du chemin.

`AddressPath::for_address` calcule `Nibbles::unpack(keccak256(address))` --
strictement le chemin d un compte dans le trie d etat. `StoragePath` combine
un `AddressPath` et le keccak256 d une cle de storage
(`for_address_and_slot`), et expose `get_slot_offset()` (toujours 64,
puisque `ADDRESS_PATH_LENGTH = 64`) pour retrouver la frontiere entre les
deux segments. La conversion `From<&StoragePath> for RawPath` les concatene
via `RawPath::join`, produisant le chemin complet de 128 nibbles utilise pour
naviguer le trie de bout en bout, du compte jusqu au slot.
