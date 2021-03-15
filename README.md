# valheim-k8s

kubernetes deployment for a valheim game-server. Based off the dockerization work by lloesche [here](https://github.com/lloesche/valheim-server-docker).

## Usage

Note: Uses Persistence Volume Claim to store data on persistent volume by default.  

```bash
helm repo add valheim-k8s https://addyvan.github.io/valheim-k8s/
helm repo update
helm install valheim-server valheim-k8s/valheim-k8s  \
  --set secrets.WORLD_NAME=example-world-name \
  --set secrets.SERVER_NAME=example-server-name \
  --set secrets.SERVER_PASS=password \
  --set storage.pvc.storageClassName=sata
```

## Configuration

| Parameter                                  | Description                                                | Default                           |
|:-------------------------------------------|:-----------------------------------------------------------|:----------------------------------|
| `secrets.WORLD_NAME`                       | Prefix of the world files to use (will make new if missing)| `example-world-name`              |
| `secrets.SERVER_NAME`                      | Server name displayed in the server browser(s)             | `example-server-name`             |
| `secrets.SERVER_PASS`                      | Server password                                            | `password`                        |
| `storage.hostVolume`                       | Configure Host Volume Storage                              | `disabled`                        |
| `storage.pvc`                              | Configure Persistence Volume Claim                         | `enabled`                         |
| `networking.serviceType`                   | The type of service e.g `NodePort`, `LoadBalancer` or `ClusterIP` | `LoadBalancer`             |
| `nodeSelector`                             |  | `{}`                 |

### Using a Host Volume

On the persistent volume you wish to use make sure the folder you are mounting exists (ideally empty if you are starting a new world). Once you spin up the game pod you should see the following files created:
```bash
$ ls /data/valheim
adminlist.txt  backups  bannedlist.txt  permittedlist.txt  prefs  worlds
```

### Using an existing world

To use an existing world simply set the `worldName` parameter to the name of your world then save the `.db` and `.fwl` files to the directory mounted into the pod. For example, if your world is named `myworld` then set `worldName: myworld` in your values file (or `--set worldName=myworld`) and assuming you are mounting at `/data/valheim` then your directory should look like: 
```bash
$ ls /data/valheim/worlds/
myworld.db  myworld.fwl
```

## Connecting to your world

Assuming you have taken care of the networking (port-forwarding if needed, LoadBalancer IP is created, ...): 
* In the steam UI (NOT IN GAME) go to view->servers->add favorite
  * set the port to 2457 ([reason for that here](https://github.com/lloesche/valheim-server-docker/discussions/32#discussioncomment-371306))
* To connect double click the server in the steam servers explorer
  * You will be asked for the password in the steam ui and in game

More visual set of instructions [here](https://github.com/mbround18/valheim-docker/discussions/51)

## Potential future updates

If there is interest I can hash out the following:
* Cronjob to save backups to s3, blob storage, or minio
  * Then clear up the space in the hostvol/pvc