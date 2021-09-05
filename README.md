# elastic-elasticsearch-pod

| License | Versioning | Build |
| ------- | ---------- | ----- |
| [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) | [![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg)](https://github.com/semantic-release/semantic-release) | [![Build status](https://ci.appveyor.com/api/projects/status/ln7itb2bk3j8pdr2/branch/master?svg=true)](https://ci.appveyor.com/project/nikAizuddin/elastic-elasticsearch-pod/branch/master) |

[Elasticsearch](https://github.com/elastic/elasticsearch) Podman Pod.

![concept](docs/source/_static/concept.svg)


## Prerequisites

Build Sphinx image:
```
podman build -t extra2000/sphinx .
```


## How to build docs into HTML

**NOTE:** The `chcon` commands are for SELinux platforms only.

Allow `./docs` to be mounted into Podman container:
```
chcon -R -v -t container_file_t ./docs
```

Create `./output` to store rendered HTML output and allow the directory to be mounted into Podman container:
```
mkdir -pv output
chcon -R -v -t container_file_t ./output
```

Build docs:
```
podman run -it --rm --network none -v ./docs:/srv/docs:ro -v ./output:/srv/docs/build:rw extra2000/sphinx make clean html
```


## How to deploy docs

Import SELinux Security Policy for `httpd` container:
```
sudo semodule -i docs_elastic_elasticsearch_pod.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}
```

Create `no-internet` network:
```
podman network create no-internet --internal
```

Deploy docs:
```
podman run --rm --network no-internet -p 18080:80 -v ./output/html:/usr/local/apache2/htdocs:ro --security-opt label=type:docs_elastic_elasticsearch_pod.process docker.io/library/httpd:2.4
```

Deploy docs using pod:
```
podman play kube --network no-internet docs-elastic-elasticsearch-pod.yaml
```
