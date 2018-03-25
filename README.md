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

Bumping version (e.g. to 1.0.7):

```
export OLD_VERSION=1.0.6
export VERSION=1.0.7
mkdir /tmp/go.$$
export GOPATH=/tmp/go.$$
cd $GOPATH
go get github.com/cunnie/gobonniego
  # ignore `no Go files in ...`
cd src/github.com/cunnie/gobonniego
go get ...
cd -
tar czvf \
  ~/workspace/gobonniego-release/blobs/gobonniego/gobonniego-${VERSION}.tgz \
  src
cd ~/workspace/gobonniego-release
git pull -r --autostash
find packages/gobonniego -type f -print0 |
  xargs -0 perl -pi -e \
  "s/gobonniego-${OLD_VERSION}/gobonniego-${VERSION}/g"
bosh add-blob \
  blobs/gobonniego/gobonniego-${VERSION}.tgz \
  gobonniego/gobonniego-${VERSION}.tgz
vim config/blobs.yml
  # delete `gobonniego/gobonniego-${OLD_VERSION}.tgz` stanza
bosh create-release --force
bosh -e vbox upload-release
bosh -e vbox -n -d gobonniego \
  deploy manifests/example.yml --recreate
bosh -e vbox -d gobonniego run-errand gobonniego
bosh upload-blobs
bosh create-release \
  --final \
  --tarball ~/Downloads/gobonniego-release-${VERSION}.tgz \
  --version ${VERSION} --force
git add -N releases/
git add -p
git ci -v
git tag $VERSION
git push
git push --tags
```