# firestore-data-migrations

A data migration tool designed for [Google Firestore](https://firebase.google.com/products/firestore).

## Features

* Undo a migration if something goes wrong
* Idempotency so migrations can be safely interrupted
* Version tracking so migrations are applied in order
* Data is represented as simple JSON or YAML files
* Use any tool, such as [`jq`](https://stedolan.github.io/jq/) or VSCode, to change your data
* Minimizes expensive writes by checking for diffs
* Use data backed up to [Google BigQuery](https://firebase.google.com/products/extensions/firestore-bigquery-export) for minimizing costs
* Creates a `migration_yyyy-mm-dd_hh:mm.json` metadata file with every migration for easy debugging
* Uses transactions and checks timestamps to avoid overwriting live data that changes during migration
* Can create additional backups in firestore to avoid reliance on local backups
* Use firestore emulator for testing

## How it works

`fbm` works with local copies of your data and `git` to make migrations. Each firestore object is stored in a file. Each migration is stored as a git commit. 

In this way, you can change your data any way you want, store the full history of changes efficiently, and deploy them atomically. 

Undoing a migration is simply reverting a git commit. 

## Limitations

`fbm` is limited by the underlying limitations of git, your local machine, the cost of firestore operations, and the [firestore quotas](https://firebase.google.com/docs/firestore/quotas). In practice, large migrations of >100k objects can take several hours to complete, because of [firestore rate limits](https://firebase.google.com/docs/firestore/quotas).

## Differences from SQL migration tools

* Does not require any schema
* Operates on local data that you export from your firestore database
* No `up` and `down` transformations
* Migrations can work on a subset of your database

## Expected workflow

### Export a collection as JSON files
```
$ fdm export mycollection
...created mycollection/E4S8LelB1ZI1zAL8eGAj.json
...created mycollection/n3Qji2UVgSXmNUHYD9GK.json
...
```

### Update object files
```
$ cd mycollection
$ fdm map 'obj => ({...obj, name: name.trim()})'
...updated 123 objects
$ fdm map 'obj => ({...obj, list: obj.list || []})'
...updated 45 objects
```

### Review the changes
```
$ fdm diff
--- a/mycollection/E4S8LelB1ZI1zAL8eGAj.json
+++ b/mycollection/E4S8LelB1ZI1zAL8eGAj.json
@@ -1,5 +1,6 @@
 {
-  name: ' slim shady'
+  name: 'slim shady',
+  list: []
 }
 
...
```

### Create a migration
```
$ fdm create mymigration 'trim whitespace from name and add empty list prop'
git: [mymigration 1c99de66] trim whitespace from name and add empty list prop
git:  123 files changed, 168 insertions(+), 0 deletions(-)
```

### Dry-run the migration
```
$ fdm dryrun mymigration
...updated object E4S8LelB1ZI1zAL8eGAj in collection mycollection (dryrun)
...updated object n3Qji2UVgSXmNUHYD9GK in collection mycollection (dryrun)
...
...updated 123 objects with 123 reads and 123 writes (dryrun)
...total cost: $0.0000984 (dryrun)
```

### Run the migration
```
$ fdm run mymigration
...updated object E4S8LelB1ZI1zAL8eGAj in collection mycollection
...updated object n3Qji2UVgSXmNUHYD9GK in collection mycollection
...
...updated 123 objects with 123 reads and 123 writes
...total cost: $0.0000984
```

### Undo the migration (if something went wrong)
```
$ fdm undo mymigration
...updated object E4S8LelB1ZI1zAL8eGAj in collection mycollection
...updated object n3Qji2UVgSXmNUHYD9GK in collection mycollection
...
...updated 123 objects with 123 reads and 123 writes
...total cost: $0.0000984
```

## Related reading
* https://github.com/kevlened/fireway
* https://stackoverflow.com/questions/53491754/
