# @minikit/mongotools

This project provides 2 wrappers :

- **mongodump**,
- **mongorestore**.

There is an autonomous feature called **rotation** that provide a backup file rotation mechanism

- remove N oldest deprecated backups.

## Command line usage

### Initial setup - first time only

```
# get source code
git clone https://github.com/minikit/mongotools.git
# install dependencies
npm install

#~ setup environment variables
cp env/initEnv.template.sh env/initEnv.dontpush.sh
# you must update env/initEnv.dontpush.sh
```

### Set your preferences

```
# source your options
. ./env/initEnv.dontpush.sh
```

### Basic feature

```bash
# create a mongo dump
node mt dump

# create a encrypted mongo dump
node mt dumpz

# list backups
node mt list

# restore a mongo local dump
# please note that mt restore use following options : dropBeforeRestore: true, deleteDumpAfterRestore: true
node mt restore backup/myDatabase__2020-11-08_150102.gz

# rotate backup files
node mt rotation

# Helper : show current options values
node mt options
```

### Add in-line extra options

You could play with env default options plus in-line command extra options

```bash
# create dump of a given 'shippingprices' collection, provide a target backup name as '2023_01_shippingPrices.gz', and show mongodump command
MT_COLLECTION=shippingprices MT_FILENAME=2023_01_shippingPrices.gz MT_SHOW_COMMAND=true node mt dump

# show backup in list
node mt list

# using mongo: drop a given collection
mongo myDb --eval "db.shippingprices.drop()"

# restore collection
MSYS_NO_PATHCONV=1 MT_COLLECTION=shippingprices MT_SHOW_COMMAND=true node mt restore /backup/2023_01_shippingprices.gz
# Note that collection option will produce wildcard in nsInclude arg '--nsInclude myDb.*'
```

## Library use

### Install dependency

You have to import as dependency

```
npm install @minikit/mongotools
```

### Define the requirements, example:

```
import { MongoTools, MTOptions, MTCommand } from "@minikit/mongotools";

var mongoTools = new MongoTools();
const mtOptions = {
        db: 'myDb',
        port: 17017,
        path: '/opt/myapp/backups',
        dropboxToken: process.env.MYAPP_DROPBOX_SECRET_TOKEN
      };
```

### List dumps

```
var promiseResult = mongoTools.list(mtOptions);
```

### Dump

```
var promiseResult = mongoTools.mongodump(mtOptions);
```

### Restore

```
var promiseResult = mongoTools.mongorestore(mtOptions);
```

### Rotation

```
var promiseResult = mongoTools.rotation(mtOptions);
```

## Mongo tools options

Each mongotools feature rely on Mongo tools options (aka. [MTOption](./lib/MTOptions.js)).

Options precedence is the following:

- take `options` attribute if set,
- else take related environment variable if any,
- else take default value if any,
- else if it's a mandatory option, throw an error.

TIP: you could also show current options by doing:

```
console.log(new MTOptions());
```

### shared options

These options are used by dump and restore.

Either `uri` or `host`/`port`/`db`:

| option | env          | required | default value | description                                                                                                                                                   |
| ------ | ------------ | -------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `uri`  | MT_MONGO_URI | **true** | (none)        | mongodump uri, example `mongodb+srv://granted-user:MySecretHere@cluster0.xzryx.mongodb.net/tMyDatababse`. You could omit database name to dump all databases. |

or

| option     | env              | required | default value | description                                                                       |
| ---------- | ---------------- | -------- | ------------- | --------------------------------------------------------------------------------- |
| `db`       | MT_MONGO_DB      | **true** | (none)        | mongo database name. For dump only, you could set it to `*` to dump all databases |
| `host`     | MT_MONGO_HOST    | false    | `127.0.0.1`   | mongo database hostname                                                           |
| `port`     | MT_MONGO_PORT    | false    | `27017`       | mongo database port                                                               |
| `username` | MT_MONGO_USER    | false    | (none)        | mongo database username                                                           |
| `password` | MT_MONGO_PWD     | false    | (none)        | mongo database password                                                           |
| `authDb`   | MT_MONGO_AUTH_DB | false    | `admin`       | mongo auth database                                                               |

#### ssl options

Optional ssl related options

| option              | env                           | required | default value | description                                                      |
| ------------------- | ----------------------------- | -------- | ------------- | ---------------------------------------------------------------- |
| `ssl`               | MT_MONGO_SSL                  | false    | (none)        | if "1" then add `--ssl` option                                   |
| `sslCAFile`         | MT_MONGO_SSL_CA_FILE          | false    | (none)        | .pem file containing the root certificate chain                  |
| `sslPEMKeyFile`     | MT_MONGO_SSL_PEM_KEY_FILE     | false    | (none)        | .pem file containing the certificate and key                     |
| `sslPEMKeyPassword` | MT_MONGO_SSL_PEM_KEY_PASSWORD | false    | (none)        | password to decrypt the sslPEMKeyFile, if necessary              |
| `sslCRLFile`        | MT_MONGO_SSL_CRL_FILE         | false    | (none)        | pem file containing the certificate revocation list              |
| `sslFIPSMode`       | MT_MONGO_SSL_FIPS             | false    | (none)        | if "1" then use FIPS mode of the installed openssl library       |
| `tlsInsecure`       | MT_MONGO_TLS_INSECURE         | false    | (none)        | if "1" then bypass the validation for server's certificate chain |

### mongodump options

| option                   | env                    | required | default value           | description                                                           |
| ------------------------ | ---------------------- | -------- | ----------------------- | --------------------------------------------------------------------- |
| `path`                   | MT_PATH                | false    | `backup`                | dump target directory, created if it doesn't exist                    |
| `dumpCmd `               |                        | false    | `mongodump`             | mongodump binary                                                      |
| `fileName`               | MT_FILENAME            | false    | `<dbName_date_time.gz>` | dump target filename                                                  |
| `encrypt`                |                        | false    | false                   | encrypt the dump using secret                                         |
| `secret`                 | MT_SECRET              | false    | null                    | secret to use if encrypt is enabled                                   |
| `encryptSuffix`          |                        | false    | `.enc`                  | encrypt file suffix                                                   |
| `includeCollections`     |                        | false    | (none)                  | **Deprecated** - please use `collection`                              |
| `collection`             | MT_COLLECTION          | false    | (none)                  | Collection to include, if not specified all collections are included  |
| `excludeCollections`     | MT_EXCLUDE_COLLECTIONS | false    | (none)                  | Collections to exclude, if not specified all collections are included |
| `numParallelCollections` |                        | false    | 4                       | Number of collections mongodump should export in parallel.            |
| `viewsAsCollections`     |                        | false    | false                   | When specified, mongodump exports read-only views as collections.     |

Simple example:

```
import { MongoTools, MTOptions, MTCommand } from "node-mongotools";
var mongoTools = new MongoTools();

mongoTools.mongodump({
   db:'myDatabase',
   path:'backup',
   username:'root', password:'mypass', authDb:'admin'
})
.then((success) => console.info("success", success) )
.catch((err) => console.error("error", err) );
```

### mongorestore options

| option                   | env          | required | default value  | description                                                    |
| ------------------------ | ------------ | -------- | -------------- | -------------------------------------------------------------- |
| `dumpFile`               | MT_DUMP_FILE | true     | (none)         | dump file to restore                                           |
| `restoreCmd`             |              | false    | `mongorestore` | mongorestore binary                                            |
| `dropBeforeRestore`      |              | false    | false          | set it to `true` to append `--drop` option                     |
| `deleteDumpAfterRestore` |              | false    | false          | set it to `true` to remove restored backup file                |
| `decrypt`                |              | false    | false          | decrypt the dump using secret. Activated if suffix is detected |
| `secret`                 | MT_SECRET    | false    | null           | secret to use if decrypt is enabled                            |

Simple example:

```
import { MongoTools, MTOptions, MTCommand } from "node-mongotools";
var mongoTools = new MongoTools();

mongoTools.mongorestore({
   dumpFile:'backup/myDatabase__2020-11-8_160011.gz',
   username:'root', password:'mypass', authDb:'admin'
})
.then((success) => {
  console.info("success", success.message);
  if (success.stderr) {
    console.info("stderr:\n", success.stderr);// mongorestore binary write details on stderr
  }
})
.catch((err) => console.error("error", err) );
```

### Rotation options

A safe time windows is defined by :

- `now - rotationWindowsDays day(s)` ===> `now`  
  where backups can't be removed.

Backup out of safe time windows are called `deprecated backup`.

- `rotationMinCount`: minimum deprecated backups to keep,
- `rotationCleanCount`: number of (oldest) deprecated backups to delete.

| option                | env                      | required | default value | description                                            |
| --------------------- | ------------------------ | -------- | ------------- | ------------------------------------------------------ |
| `rotationDryMode`     | MT_ROTATION_DRY_MODE     | false    | false         | dont do delete actions, just print it                  |
| `rotationWindowsDays` | MT_ROTATION_WINDOWS_DAYS | true     | 15            | safe time windows in days since now                    |
| `rotationMinCount`    | MT_ROTATION_MIN_COUNT    | true     | 2             | minimum deprecated backups to keep.                    |
| `rotationCleanCount`  | MT_ROTATION_CLEAN_COUNT  | true     | 10            | number of (oldest first) deprecated backups to delete. |

Simple example:

```
MT_ROTATION_CLEAN_COUNT=2 \
MT_ROTATION_DRY_MODE=true \
MT_ROTATION_WINDOWS_DAYS=3 \ node mt rotation
```

Example details: if there is a backup that is more than 3 days old, keep 2 newer ones and delete the 10 oldest.
