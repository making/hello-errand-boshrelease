```
bosh init-release --dir=hello-errand-boshrelease --git
cd hello-errand-boshrelease/

bosh generate-job hello

mkdir -p jobs/hello/templates/bin
cat <<'EOF' > jobs/hello/templates/bin/run.sh
#!/bin/bash
echo "Hello World! @$(date)"
EOF
chmod +x jobs/hello/templates/bin/run.sh

cat <<'EOF' > jobs/hello/spec
---
name: hello

templates:
  bin/run.sh: bin/run

packages: []

properties: {}
EOF


bosh create-release --name=hello-errand --force --timestamp-version --tarball=/tmp/hello-errand-boshrelease.tgz
bosh upload-release /tmp/hello-errand-boshrelease.tgz


cat <<'EOF' > manifest.yml
---
name: hello-errand

releases:
- name: hello-errand
  version: latest

stemcells:
- os: ubuntu-xenial
  alias: xenial
  version: latest

instance_groups:
- name: hello
  lifecycle: errand
  jobs:
  - name: hello
    release: hello-errand
    properties: {}
  instances: 1
  stemcell: xenial
  azs: [z1]
  vm_type: default
  networks:
  - name: default

update:
  canaries: 1
  max_in_flight: 3
  canary_watch_time: 30000-600000
  update_watch_time: 5000-600000
EOF

bosh deploy -d hello-errand manifest.yml -n
bosh -d hello-errand run-errand hello
```