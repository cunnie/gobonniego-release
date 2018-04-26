# gobonniego-release

[GoBonnieGo](https://github.com/cunnie/gobonniego) BOSH Release (disk benchmarking tool)

## Upload The Release

```bash
bosh upload-release --sha1=7cb7a231c4231ef5cacc4b89208e72f468afa2cc https://github.com/cunnie/gobonniego-release/releases/download/1.0.5/gobonniego-release-1.0.5.tgz
```

## Example

On Google's Cloud, let's test a 256 GiB _pd-standard_ disk.

First, let's create a Cloud Config that has the disk we
want to test:

```yaml
disk_types:
- name: pd-standard-256
  disk_size: 262_144
  cloud_properties:
    type: pd-standard
```

Next, let's upload our configuration to our BOSH Director:

```bash
bosh update-config --type=cloud --name=gobonniego gce-cc.yml
```

Create a BOSH deployment manifest (`gobonniego.yml`):

```yaml
name: gobonniego

instance_groups:
- name: pd-standard-256
  lifecycle: errand
  instances: 1
  jobs:
  - name: gobonniego
    release: gobonniego
    properties:
      dir: /var/vcap/store/gobonniego
      args: -seconds 86400 -iops-duration 60 -json
  persistent_disk_type: pd-standard-256
```

Deploy our manifest:

```bash
bosh -d gobonniego deploy gobonniego.yml
```

Instruct BOSH to run the errand, specifying all output in JSON. Use `jq` to extract the output of the BOSH errand (which also happens to be JSON), and save that to a file:

```bash
bosh -e gce -d gobonniego run-errand pd-standard-256 --json |
  jq -r .Tables[].Rows[].stdout > gce_pd-standard-256.json
```

Look
[here](https://github.com/cunnie/deployments/tree/b8c53bde94133aca4840465a55d403281e6de9ea/gobonniego)
for a real-world example of Cloud Configs and manifests used to benchmark many
different kinds of disks.

### Developer Notes

Developer notes, such as building and test a release, are available [here](docs/DEVELOPER.md).
