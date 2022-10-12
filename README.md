# S3 Datastore Implementation

This is an implementation of the datastore interface backed by amazon s3.

**NOTE:** Plugins only work on Linux and MacOS at the moment. You can track the progress of this issue here: https://github.com/golang/go/issues/19282

## Bundling

As go plugins can be finicky to correctly compile and install, you may want to consider bundling this plugin and re-building go-ipfs. If you do it this way, you won't need to install the `.so` file in your local repo, i.e following the above Building and Installing section, and you won't need to worry about getting all the versions to match up.

```bash
# We use go modules for everything.
> export GO111MODULE=on

# Clone go-ipfs.
> git clone https://github.com/ipfs/go-ipfs
> cd go-ipfs

# Pull in the datastore plugin (you can specify a version other than latest if you'd like).
> go get github.com/opensaucerer/go-ds-s3/plugin@latest

# Add the plugin to the preload list.
> echo -en "\ns3ds github.com/opensaucerer/go-ds-s3/plugin 0" >> plugin/loader/preload_list

# ( this first pass will fail ) Try to build go-ipfs with the plugin
> make build

# Update the deptree
> go mod tidy

# Now rebuild go-ipfs with the plugin
> make build

# (Optionally) install go-ipfs
> make install
```

## Detailed Installation

For a brand new ipfs instance (no data stored yet):

1. Edit $IPFS_DIR/config to include s3 details (see Configuration below).

### Configuration

The config file should include the following:

```json
{
  "Datastore": {
  ...

    "Spec": {
      "mounts": [
        {
          "child": {
            "type": "s3ds",
            "region": "us-east-1",
            "bucket": "$bucketname",
            "rootDirectory": "$bucketsubdirectory",
            "accessKey": "",
            "secretKey": ""
          },
          "mountpoint": "/blocks",
          "prefix": "s3.datastore",
          "type": "measure"
        },
```

If the access and secret key are blank they will be loaded from the usual ~/.aws/.
If you are on another S3 compatible provider, e.g. Linode, then your config should be:

```json
{
  "Datastore": {
  ...

    "Spec": {
      "mounts": [
        {
          "child": {
            "type": "s3ds",
            "region": "us-east-1",
            "bucket": "$bucketname",
            "rootDirectory": "$bucketsubdirectory",
            "regionEndpoint": "us-east-1.linodeobjects.com",
            "accessKey": "",
            "secretKey": ""
          },
          "mountpoint": "/blocks",
          "prefix": "s3.datastore",
          "type": "measure"
        },
```

2. Overwrite `$IPFS_DIR/datastore_spec` as specified below (_Don't do this on an instance with existing data - it will be lost_).

If you are configuring a brand new ipfs instance without any data, you can overwrite the datastore_spec file with:

```
{"mounts":[{"bucket":"$bucketname","mountpoint":"/blocks","region":"us-east-1","rootDirectory":"$bucketsubdirectory"},{"mountpoint":"/","path":"datastore","type":"levelds"}],"type":"mount"}
```

Otherwise, you need to do a datastore migration.

## Contribute

Feel free to join in. All welcome. Open an [issue](https://github.com/opensaucerer/go-ds-s3/issues)! Or create a [Pull Request](https://github.com/opensaucerer/go-ds-s3/pulls)
