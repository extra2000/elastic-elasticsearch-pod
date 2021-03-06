# Changelog

## [7.3.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v7.2.0...v7.3.0) (2022-06-06)


### Features

* **dockerfiles:** upgrade ES from `8.2.1` to `8.2.2` ([bed87af](https://github.com/extra2000/elastic-elasticsearch-pod/commit/bed87af547bccfc19a51b91fb9453c2e179c7e58))


### Continuous Integrations

* **AppVeyor:** revert "disable test due to Kubic broken mirror" ([f532ca1](https://github.com/extra2000/elastic-elasticsearch-pod/commit/f532ca1abf7fb677fe1b9a60aad0ccfba84bdb65))

## [7.2.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v7.1.0...v7.2.0) (2022-05-25)


### Features

* **dockerfiles:** upgrade ES from `8.2.0` to `8.2.1` ([f984cc3](https://github.com/extra2000/elastic-elasticsearch-pod/commit/f984cc3be921cc021b5581261940b947d322d590))


### Continuous Integrations

* **AppVeyor:** disable test due to Kubic broken mirror ([441f104](https://github.com/extra2000/elastic-elasticsearch-pod/commit/441f104571fb099bee47f8e2e2db6c29e6225fc6))

## [7.1.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v7.0.0...v7.1.0) (2022-05-05)


### Features

* **dockerfiles:** upgrade ES from `8.1.2` to `8.2.0` ([3baad94](https://github.com/extra2000/elastic-elasticsearch-pod/commit/3baad94a2f084884263c8c9ad02d4baad6c2276f))

## [7.0.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v6.2.0...v7.0.0) (2022-04-11)


### ⚠ BREAKING CHANGES

* **pods:** SSL type and name has changed from P12 to PEM for all pods in `cluster-multi-server`
* **pods:** SELinux name has changed for `cluster-multi-server`
* **pods:** container name has changed to `srv01` for all containers in `cluster-multi-server`
* **configs:** certs name and type has changed from P12 to PEM in `cluster-multi-server` configs
* **selinux:** SELinux files and names for `cluster-multi-server` have changed

### Documentations

* **cluster-multi-server:** remove old SSL instructions ([f9edc65](https://github.com/extra2000/elastic-elasticsearch-pod/commit/f9edc65d0ad1ac4453becac7d247b5a62638fa98))
* **cluster-multi-server:** update container filename for systemd ([7820480](https://github.com/extra2000/elastic-elasticsearch-pod/commit/7820480881535aae58930d1072dfa4bf90c4bc2e))
* **cluster-multi-server:** update es-coord container name for setup pswd ([8710b02](https://github.com/extra2000/elastic-elasticsearch-pod/commit/8710b02e59d9cf67795adc877f37f8e1ef6f4a9d))
* **cluster-multi-server:** update SELinux instructions for new name ([5549ba2](https://github.com/extra2000/elastic-elasticsearch-pod/commit/5549ba2b4e14910c9b3d60a8aebd9c592b8ecb9b))


### Code Refactoring

* **configs:** change certs type and name for cluster multi server ([b274809](https://github.com/extra2000/elastic-elasticsearch-pod/commit/b274809a669a897adcdc71d5d6cf0b519a6a2a44))
* **pods:** change SSL type and name for cluster multi server ([1b19092](https://github.com/extra2000/elastic-elasticsearch-pod/commit/1b19092bdbc8580f6ef4932a63e6e8b755e027ea))
* **pods:** rename container name to srv01 for cluster multi server ([b99b74b](https://github.com/extra2000/elastic-elasticsearch-pod/commit/b99b74b29383b9b5f64720804ec91ea3e02b2a3a))
* **pods:** rename SELinux name for cluster multi server ([b141f71](https://github.com/extra2000/elastic-elasticsearch-pod/commit/b141f7102b359b1db8bd7aa2339e5161c463e290))
* **selinux:** rename SELinux name for cluster multi server ([1edce99](https://github.com/extra2000/elastic-elasticsearch-pod/commit/1edce991c020c1bb5631449cc270e51f04011231))

## [6.2.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v6.1.0...v6.2.0) (2022-04-02)


### Features

* **dockerfile:** update ES from `8.1.1` to `8.1.2` ([9c6e7ed](https://github.com/extra2000/elastic-elasticsearch-pod/commit/9c6e7ed68438ef97b72460805e247f8813b29918))


### Documentations

* **known-issues:** simplify instructions ([b86a249](https://github.com/extra2000/elastic-elasticsearch-pod/commit/b86a249ac50311359e2473656abb82c5af0adedc))

## [6.1.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v6.0.0...v6.1.0) (2022-03-31)


### Features

* **dockerfile:** update ES from `8.1.0` to `8.1.1` ([1aa38a6](https://github.com/extra2000/elastic-elasticsearch-pod/commit/1aa38a6fa2503a9860450ed27dfe3be4e960d236))


### Documentations

* **deployment:** simplify `podman generate systemd` ([a58e6aa](https://github.com/extra2000/elastic-elasticsearch-pod/commit/a58e6aa6e848eb6c5b2aa6067405006f96d06a17))

## [6.0.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v5.0.0...v6.0.0) (2022-03-31)


### ⚠ BREAKING CHANGES

* **pods:** types and mount points for certificate have changed
* **docs:** Instructions for SSL certs has changed

### Features

* **dockerfiles:** upgrade ES from version `8.0.1` to `8.1.0` ([1626d96](https://github.com/extra2000/elastic-elasticsearch-pod/commit/1626d96e97417922106656f781551d01d05005e4))


### Code Refactoring

* **docs:** using OpenSSL to generate certs ([74330f7](https://github.com/extra2000/elastic-elasticsearch-pod/commit/74330f7b8848119e158c3ddbde37f9b1f8cf5242))
* **pod:** add `runasuser` as comment ([5566340](https://github.com/extra2000/elastic-elasticsearch-pod/commit/556634046971d84576feae91f23223fcc5aa0baf))
* **pods:** using certs generated from OpenSSL ([0f16222](https://github.com/extra2000/elastic-elasticsearch-pod/commit/0f1622245b578afe000dcf9ce04fd824344b29fb))


### Documentations

* **deployments:** simplify CuRL command ([c8ee24a](https://github.com/extra2000/elastic-elasticsearch-pod/commit/c8ee24a3a7680f849e109e3484624f76f53e51a0))
* **deployments:** using `elasticsearch-reset-password` ([4fcfc3d](https://github.com/extra2000/elastic-elasticsearch-pod/commit/4fcfc3d062f810efc532679f65d42057c74640cf))

## [5.0.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v4.0.0...v5.0.0) (2022-03-10)


### ⚠ BREAKING CHANGES

* **deployments:** Deployment directory for `general-single-instance` has been reorganized

### Features

* **dockerfiles:** update Elasticsearch from `8.0.0` to `8.0.1` ([d25cff5](https://github.com/extra2000/elastic-elasticsearch-pod/commit/d25cff5ffda9ad67d9db9047daaa4f9b4d189866))


### Code Refactoring

* **deployments:** reorganize `general-single-instance` ([bac2ed9](https://github.com/extra2000/elastic-elasticsearch-pod/commit/bac2ed949e6e80bf3439aeae1228c4a7df97b027))
* **docs:** reorganize documentations for `general-single-instance` ([699d669](https://github.com/extra2000/elastic-elasticsearch-pod/commit/699d6695e96807e4818b7b832c3d4525845b543f))

## [4.0.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v3.3.0...v4.0.0) (2022-02-20)


### ⚠ BREAKING CHANGES

* **deployments:** existing pods may no longer compatible because the whole config directory is mounted instead of keystore file
* **dockerfiles:** Elasticsearch 8.0 upgrade has breaking changes

### Features

* **dockerfiles:** upgrade ES from `7.17.0` to `8.0.0` ([9a69794](https://github.com/extra2000/elastic-elasticsearch-pod/commit/9a697944df5d818eb507fd15e2647d1132f64dd6))


### Documentations

* add Chapter `Known Issues` ([e17e33f](https://github.com/extra2000/elastic-elasticsearch-pod/commit/e17e33f731985ed46cd9cf9b5cb7fb49d37a0dfc))


### Code Refactoring

* **deployments:** mount the whole config directory instead of keystore file ([c0ddfed](https://github.com/extra2000/elastic-elasticsearch-pod/commit/c0ddfed3db47bd83d15f4e0e05a4fae1f77d398c))
* **deployments:** remove Cluster Single Server deployment ([7bc992d](https://github.com/extra2000/elastic-elasticsearch-pod/commit/7bc992da1c20fe6a6e3e45f6e42187698900c65a))
* **deployments:** remove RPi Single Instance deployment ([2ee3fcb](https://github.com/extra2000/elastic-elasticsearch-pod/commit/2ee3fcb2242eb571c628377c917bed71b850e257))
* **dockerfiles:** remove S3 plugin since ES `8.0` already include ([d24b8ae](https://github.com/extra2000/elastic-elasticsearch-pod/commit/d24b8ae8cd6203fad0812af7e48eb6b262d007b6))

## [3.3.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v3.2.0...v3.3.0) (2022-02-17)


### Features

* **dockerfile:** upgrade Elasticsearch from `7.16.3` to `7.17.0` ([feb7c54](https://github.com/extra2000/elastic-elasticsearch-pod/commit/feb7c5460ffa2ad28f3ebcc739682a5f67662a25))


### Documentations

* **deployments:** add `-storepass changeit` to `jdk/bin/keytool` cmd ([fdab911](https://github.com/extra2000/elastic-elasticsearch-pod/commit/fdab9110827d76ff1435249750a598548bf7ce02))
* **general-single-instance:** add `cluster.routing.allocation.disk.watermark.enable_for_single_data_node: true` prior to Elasticsearch version 8.0 deprecation warning ([49a7201](https://github.com/extra2000/elastic-elasticsearch-pod/commit/49a7201c7cbd5b8b675b79686ed5a5baceba721d))

## [3.2.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v3.1.0...v3.2.0) (2022-01-29)


### Features

* **dockerfile:** upgrade Elasticsearch from `7.16.2` to `7.16.3` ([ba91ab8](https://github.com/extra2000/elastic-elasticsearch-pod/commit/ba91ab8d310e39449dffddfea90160506325aaf0))
* **general-single-instance:** log ES to file ([2fc7b1f](https://github.com/extra2000/elastic-elasticsearch-pod/commit/2fc7b1fbae7e67f6e9348a584365c3c83d64bf51))


### Documentations

* **general-single-instance:** add instructions to distribute secrets ([fd5c63a](https://github.com/extra2000/elastic-elasticsearch-pod/commit/fd5c63af790d833f62abe5c3428ce3309d243300))
* **general-single-instance:** add warning not to distribute CA signer ([84b2116](https://github.com/extra2000/elastic-elasticsearch-pod/commit/84b21164864900ba39215c10c748ce7ab8a353c0))
* **logo:** remove logo ([de54eca](https://github.com/extra2000/elastic-elasticsearch-pod/commit/de54eca5d5585598bc133c1b3020d38e179f5d8d))

## [3.1.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v3.0.0...v3.1.0) (2022-01-04)


### Features

* **logging:** log cluster-multi-servers instances to files ([1d3b59e](https://github.com/extra2000/elastic-elasticsearch-pod/commit/1d3b59e08199e2a7317f0be667c5f88c495d79d5))


### Documentations

* **deployment:** add instruction to create `configs/log4j2.properties` ([6db57eb](https://github.com/extra2000/elastic-elasticsearch-pod/commit/6db57eb0cc0759eecb2f206ae8cdc8b1a646a47a))

## [3.0.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v2.1.0...v3.0.0) (2021-12-31)


### ⚠ BREAKING CHANGES

* **configs:** Stack Monitoring with self-monitoring no longer works. Instead, use Metricbeat for Stack Monitoring.

### Features

* **dockerfile:** upgrade Elasticsearch from `7.15.2` to `7.16.2` ([ab3e687](https://github.com/extra2000/elastic-elasticsearch-pod/commit/ab3e687397dc84694c7957fb8e3f0dfb363a3d18))


### Fixes

* **configmaps:** add `-Des.transport.cname_in_publish_address=true` to preserve deprecated setting ([8bcea7f](https://github.com/extra2000/elastic-elasticsearch-pod/commit/8bcea7f9ae1a32632d4d38a333c6172617ea6da3))
* **configs:** remove deprecated `xpack.monitoring` settings ([9c76313](https://github.com/extra2000/elastic-elasticsearch-pod/commit/9c76313c493ea949c0666f4749a3ef31250fcccf))

## [2.1.0](https://github.com/extra2000/elastic-elasticsearch-pod/compare/v2.0.2...v2.1.0) (2021-12-14)


### Features

* **general-pods:** externalize JDK `cacerts` file to simplify importing MinIO CA ([56925b3](https://github.com/extra2000/elastic-elasticsearch-pod/commit/56925b31c89992c350e844e0d7bae9469162db08))


### Code Refactoring

* **config:** change default S3 protocol from HTTP to HTTPS ([70f15b6](https://github.com/extra2000/elastic-elasticsearch-pod/commit/70f15b6aa88c1243b8a20a40bfaec1d9a71a8885))


### Documentations

* **deployments:** add instructions to create JDK `cacerts` and import `minio-01` CA cert ([6124657](https://github.com/extra2000/elastic-elasticsearch-pod/commit/61246573e09318d45f76eff85985f022cfcbd46c))

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
