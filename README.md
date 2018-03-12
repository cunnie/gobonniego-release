# gobonniego-release

[GoBonnieGo](https://github.com/cunnie/gobonniego) BOSH Release (disk benchmarking tool)

```
bosh upload-release --sha1=7cb7a231c4231ef5cacc4b89208e72f468afa2cc https://github.com/cunnie/gobonniego-release/releases/download/1.0.5/gobonniego-release-1.0.5.tgz
```

Look in the manifests subdirectory for examples on how to use.

## Developer Notes

Rapid testing:

```
 # make changes
bosh create-release --force
bosh -e vbox upload-release
bosh -e vbox -d gobonniego -n deploy manifests/example.yml
bosh -e vbox -d gobonniego run-errand gobonniego
```
