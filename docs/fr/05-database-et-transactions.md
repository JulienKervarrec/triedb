# Chapitre 5 -- Database, Transaction et le cycle MVCC (src/database.rs, src/transaction.rs)

`Database` (src/database.rs) enveloppe un `StorageEngine` et un
`TransactionManager` proteges par un `Mutex`. `DatabaseOptions` expose les
options d ouverture classiques (`create`, `create_new`, `wipe`, `meta_path`,
`max_pages`, `num_threads`) avec une semantique explicitement calquee sur
`std::fs::OpenOptions`. `Database::open_with_options` ouvre d abord le
fichier de metadonnees (`MetadataManager`), lit le nombre de pages actif,
configure le `PageManager` en consequence, construit un pool de threads
dedie aux calculs intensifs (le hachage, notamment) via
`executor::threadpool`, puis assemble le `StorageEngine`.

`begin_ro` et `begin_rw` (fonctions libres, generiques sur
`DB: Deref<Target = Database>` pour fonctionner aussi bien avec une reference
qu avec un `Arc<Database>`, comme le verifie `test_db_arc_tx`) implementent le
coeur du MVCC. `begin_ro` lit le contexte courant du moteur de stockage
(`storage_engine.read_context()`) et enregistre la transaction aupres du
`TransactionManager` avec son `snapshot_id` -- une lecture ne bloque jamais
et voit toujours l etat tel qu il etait au moment de son ouverture, meme si
des ecritures sont commitees ensuite (verifie explicitement par
`test_set_get_account`, ou une transaction RO ouverte avant un commit ne voit
pas les changements, tandis qu une transaction RO ouverte apres les voit).
`begin_rw` obtient un contexte d ecriture et, si le `TransactionManager`
retourne un `min_snapshot_id` en avancant, debloque
(`storage_engine.unlock`) les anciennes versions qui ne sont plus
referencees par aucune transaction en cours -- c est le mecanisme de
recyclage des pages orphelines evoque au chapitre 2, declenche uniquement
quand plus aucun lecteur n en a besoin.

`Transaction<DB, K>` est generique sur un type marqueur `RO` ou `RW` (via le
trait scelle `TransactionKind`), ce qui permet au compilateur d interdire
`set_account`/`set_storage_slot`/`commit` en ecriture sur une transaction en
lecture seule. Cote ecriture, `set_account` et `set_storage_slot` n ecrivent
rien immediatement : ils accumulent les changements dans une
`HashMap<RawPath, Option<TrieValue>>` (`pending_changes`). `commit()`
draine cette map, appelle `storage_engine.set_values` pour appliquer tous
les changements d un coup, puis `storage_engine.commit` pour persister -- un
commit est donc atomique par construction, tous les changements d une
transaction sont appliques ensemble ou pas du tout. La transaction retourne
le nouveau `state_root` (`B256`) directement depuis le contexte mis a jour.
