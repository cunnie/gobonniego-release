## Developer Notes

Rapid testing:

```
 # make changes
bosh create-release --force
bosh -e vbox upload-release
bosh -e vbox -d gobonniego -n deploy manifests/example.yml
bosh -e vbox -d gobonniego run-errand gobonniego
```

Bumping version (e.g. to 1.0.10):

```
export OLD_VERSION=1.0.9
export VERSION=1.0.10
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
find packages/gobonniego README.md -type f -print0 |
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
  deploy manifests/bosh-lite.yml --recreate
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
