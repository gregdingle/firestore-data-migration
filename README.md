# firestore-data-migrations

A data migration tool designed for [Google Firestore](https://firebase.google.com/products/firestore).

## Features

* Version tracking so that a migration can only be applied on top of older migrations

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

### Create a migration
```
$ fdm create mymigration 'trim whitespace from name and add empty list prop'
git: [mymigration 1c99de66] trim whitespace from name and add empty list prop
git:  123 files changed, 168 insertions(+), 0 deletions(-)
```

### Review changes
```
$ fdm diff mymigration
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

### Dry-run the migration
```
$ fdm dryrun mymigration
...updated object E4S8LelB1ZI1zAL8eGAj in collection mycollection (dryrun)
...updated object n3Qji2UVgSXmNUHYD9GK in collection mycollection (dryrun)
...
...updated 123 objects
```

### Run the migration
```
$ fdm run mymigration
...updated object E4S8LelB1ZI1zAL8eGAj in collection mycollection
...updated object n3Qji2UVgSXmNUHYD9GK in collection mycollection
...
...updated 123 objects
```

## Related reading
* https://github.com/kevlened/fireway
* https://stackoverflow.com/questions/53491754/
