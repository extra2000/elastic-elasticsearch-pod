# Changelog

### [2.0.2](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v2.0.1...v2.0.2) (2021-12-10)


### Security

* **CVE-2021-44228:** apply log4j mitigation ([e1bbae6](https://github.com/extra2000/elastic-elasticsearch-pod/commit/e1bbae63f051ccdc0ffb04be380f223b3ce1b588))


### Continuous Integrations

* **semantic-release:** add `security()` release ([d9eebbb](https://github.com/extra2000/elastic-elasticsearch-pod/commit/d9eebbb8e4b2a6c878f79f5e9dacb22ed12090a2))

### [2.0.1](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v2.0.0...v2.0.1) (2021-11-27)


### Fixes

* **es-master-03/.gitignore:** fix typo ([fdcbf83](https://github.com/extra2000/elastic-elasticsearch-pod/commit/fdcbf8312ac09d181eba66307dfb7ad826785983))

## [2.0.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v1.4.0...v2.0.0) (2021-11-27)


### ⚠ BREAKING CHANGES

* **deployments:** existing pod files have been removed and changed into example pod file
* **podman:** We no longer support rootful Podman

### Styles

* **section:** simplify Sections ([68e6f86](https://github.com/extra2000/elastic-elasticsearch-pod/commit/68e6f8627141cb90dce14d7406da95479774026b))


### Code Refactoring

* **configs:** disable memory locks due to rootless Podman ([d3e401d](https://github.com/extra2000/elastic-elasticsearch-pod/commit/d3e401d8d06b283be8f1f6b35f4cd0c6da43a648))
* **deployments:** change pod file into example pod file ([34b6f6f](https://github.com/extra2000/elastic-elasticsearch-pod/commit/34b6f6f733530fd3b255db9b1a660d4c04ea217b))
* **layout:** change Sphinx page to full width ([214b658](https://github.com/extra2000/elastic-elasticsearch-pod/commit/214b658330f66cb56b58371977a3cf805678acdc))
* **pods:** remove `runAsGroup` and `runAsUser` due to rootless Podman ([342fc43](https://github.com/extra2000/elastic-elasticsearch-pod/commit/342fc4360530678dda08bb833e64f80253d4fcad))
* **pods:** remove unused `elastic-ca.p12` ([efe9b88](https://github.com/extra2000/elastic-elasticsearch-pod/commit/efe9b88247b09b613983f20f8879fac8e4a24045))
* **selinux:** remove unnecessary `container_file_t` and `user_home_t` labels ([d34732e](https://github.com/extra2000/elastic-elasticsearch-pod/commit/d34732e3e62044d7e434b62b86a71429a3a40ac3))


### Documentations

* **certificates:** add instructions to distribute certificates and mentions that `elastic-ca.p12` should not be distributed ([39e8a49](https://github.com/extra2000/elastic-elasticsearch-pod/commit/39e8a4997695efefadbe186dd36cb7ebd4336444))
* **deployments:** add instructions to check transport and HTTP certificates ([339b9b0](https://github.com/extra2000/elastic-elasticsearch-pod/commit/339b9b0b39d78a105a454063561922f477178f30))
* **deployments:** add instructions to clone repository ([8a0e99b](https://github.com/extra2000/elastic-elasticsearch-pod/commit/8a0e99b85342835a2176050be3f160b16752e055))
* **deployments:** add instructions to create pod files ([1edc9f0](https://github.com/extra2000/elastic-elasticsearch-pod/commit/1edc9f02f880dd6885092e8dd7ef95ee8bfda31d))
* **deployments:** fix hostnames spacing in Table ([86c9c9f](https://github.com/extra2000/elastic-elasticsearch-pod/commit/86c9c9ff0d9200a6dd9c1551a6f1b97405ad60c0))
* **deployments:** improve `cd` instructions ([42ede61](https://github.com/extra2000/elastic-elasticsearch-pod/commit/42ede61df4fd06249e8c7c82ede4bbba7198bf0b))
* **global-secrets:** add CN name during creating CA cert ([962a0e9](https://github.com/extra2000/elastic-elasticsearch-pod/commit/962a0e9762422f2ca59c7816608bd3c1d2d5eab6))
* **global-secrets:** add instruction to check CA certificate ([83d96b6](https://github.com/extra2000/elastic-elasticsearch-pod/commit/83d96b6af090084c704d32d1b115c10840555ac7))
* **host-preparations:** update instructions for rootless Podman ([209597c](https://github.com/extra2000/elastic-elasticsearch-pod/commit/209597ce33a0c1ba0fda45758c7a4341d3199ca8))
* **podman:** replace rootful Podman into rootless Podman ([5652867](https://github.com/extra2000/elastic-elasticsearch-pod/commit/5652867b09655e445df29ee3a1be02e9a54b3a4e))
* **selinux:** label config files to allow to be mounted into containers ([0aa1cde](https://github.com/extra2000/elastic-elasticsearch-pod/commit/0aa1cded7ffc289aeba92148e6684fda09fbef14))
* **setup:** add `--url` to `elasticsearch-setup-passwords` command ([1811c5a](https://github.com/extra2000/elastic-elasticsearch-pod/commit/1811c5ae92b5eb0203676a9046a2ae63f611a73a))
* **systemd:** change `systemd` instruction for rootless Podman ([dfed5ac](https://github.com/extra2000/elastic-elasticsearch-pod/commit/dfed5acdd770dd389a8c65ca386290a976083e58))

## [1.4.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v1.3.1...v1.4.0) (2021-11-12)


### Features

* **es:** update from version `7.15.1` to `7.15.2` ([d7a5d95](https://github.com/extra2000/elastic-elasticsearch-pod/commit/d7a5d95c9146f1a86b135db44632a6e94c369142))

### [1.3.1](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v1.3.0...v1.3.1) (2021-11-02)


### Fixes

* **deployment:** add missing `_global_secrets_` directory ([dc45741](https://github.com/extra2000/elastic-elasticsearch-pod/commit/dc457413735202ff6a5e80e1bf3c23cbbd85228c))

## [1.3.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v1.2.0...v1.3.0) (2021-11-02)


### Features

* **deployment:** add general single instance deployment ([3ca2618](https://github.com/extra2000/elastic-elasticsearch-pod/commit/3ca2618b99a807173f36f93be91979e2a1f33f37))


### Documentations

* **deployment:** add docs for general single instance deployment ([87ff57f](https://github.com/extra2000/elastic-elasticsearch-pod/commit/87ff57f05217241f6132375e05a1a2c275df0a0a))

## [1.2.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v1.1.0...v1.2.0) (2021-10-25)


### Features

* **es:** upgrade from version `7.15.0` to `7.15.1` ([009e9e1](https://github.com/extra2000/elastic-elasticsearch-pod/commit/009e9e1cfa922edfce3a86a337e98fa0092b90f4))

## [1.1.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v1.0.1...v1.1.0) (2021-10-23)


### Features

* **dockerfile:** upgrade Elasticsearch from `7.14.1` to `7.15.0` ([548f0fd](https://github.com/extra2000/elastic-elasticsearch-pod/commit/548f0fdc62c1b83976810f65b5ce8e3502441038))


### Documentations

* **configs:** add empty `reindex.remote.whitelist` ([4ff2439](https://github.com/extra2000/elastic-elasticsearch-pod/commit/4ff2439700422be8a309651b7bbb951989bb6e96))


### Continuous Integrations

* **appveyor:** ensure certificates always latest ([2a132d7](https://github.com/extra2000/elastic-elasticsearch-pod/commit/2a132d7d404e7876b28cc44da9cfac000586cd04))
* **appveyor:** upgrade Ubuntu from `18.04` to `20.04` ([50cf0cb](https://github.com/extra2000/elastic-elasticsearch-pod/commit/50cf0cb0949f51b2c2a2ef1b53b43c71ead855a1))

### [1.0.1](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v1.0.0...v1.0.1) (2021-09-08)


### Fixes

* **host_preparations:** add `git` installation ([e5d5e77](https://github.com/extra2000/elastic-elasticsearch-pod/commit/e5d5e77307c3c3d5298f41c3414a850263fc954c))

## 1.0.0 (2021-09-06)


### Features

* initial commit ([516ae33](https://github.com/extra2000/elastic-elasticsearch-pod/commit/516ae338c1b04fd587ad61b23648cfb2f074f8c5))


### Documentations

* **README:** update `README.md` ([3ae7d5a](https://github.com/extra2000/elastic-elasticsearch-pod/commit/3ae7d5a40d452c58e8b9a3cbf53570847d5dd963))


### Continuous Integrations

* add AppVeyor with `semantic-release` ([1cb6c8c](https://github.com/extra2000/elastic-elasticsearch-pod/commit/1cb6c8c77234643189f8c02bec41ed3e83b25518))
