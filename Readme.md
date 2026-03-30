# serverless-certificate-creator

[![serverless](http://public.serverless.com/badges/v3.svg)](http://www.serverless.com)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/amplify-education/serverless-domain-manager/master/LICENSE)

# Table of Contents

- [Description](#description)
- [Installation](#installation)
- [Usage Requirements](#usage-requirements)
- [Usage](#usage)
- [Configuration Reference](#configuration-reference)
- [Combine with serverless-domain-manager](#combine-with-serverless-domain-manager)
  * [Examples](#examples)
- [Changelog](#changelog)
- [License](#license)

# Description

This serverless plugin creates and manages ACM certificates for your custom domains in API Gateway.
Use it in your CI/CD flow to automatically create a certificate, create the necessary Route 53 record sets to validate the certificate via DNS validation, and wait until the certificate has been validated.

## Key Features

- Automatic ACM certificate creation with DNS validation
- Route 53 record set management for certificate validation
- Tag synchronisation on existing certificates
- Configurable key algorithm (defaults to `EC_prime256v1`)
- Certificate info export to YAML files
- Full pagination support for accounts with 100+ certificates
- Compatible with **Serverless Framework v3 and v4**

# Installation

```bash
npm i serverless-certificate-creator-v3 --save-dev
```

> **Note:** The npm package name is `serverless-certificate-creator-v3` starting from v2.0.0.

# Usage Requirements

Make sure you have the following installed before starting:
* [Node.js](https://nodejs.org/en/download/) >= v18.0.0
* [npm](https://www.npmjs.com/get-npm)
* [Serverless Framework](https://serverless.com/framework/docs/providers/aws/guide/installation/) >= v3.0.0 or [OSLS](https://github.com/pocketenv-io/osls) (compatible with both)

# Usage

Open `serverless.yml` and add the following:

```yaml
plugins:
  - serverless-certificate-creator-v3

custom:
  customCertificate:
    # required
    certificateName: 'abc.somedomain.io'
    # optional
    idempotencyToken: 'abcsomedomainio'
    # required if hostedZoneIds is not set (can also be an array)
    hostedZoneNames: 'somedomain.io.'
    # required if hostedZoneNames is not set
    hostedZoneIds: 'XXXXXXXXX'
    # optional – default is false. When true, a YAML file with the certificate ARN
    # is written after running `serverless create-cert` (can also be an array)
    writeCertInfoToFile: false
    # optional – only used when writeCertInfoToFile is true
    certInfoFileName: 'cert-info.yml'
    # optional – default is us-east-1 (required for Edge-type API Gateway domains)
    region: eu-west-1
    # optional – key algorithm for the certificate (default: EC_prime256v1)
    keyAlgorithm: 'EC_prime256v1'
    # optional – Subject Alternative Names
    subjectAlternativeNames:
      - 'www.somedomain.io'
      - 'def.somedomain.io'
    # optional – tags applied to the certificate in ACM
    tags:
      Name: 'somedomain.com'
      Environment: 'prod'
    # optional – default false. Use UPSERT instead of CREATE for DNS records
    rewriteRecords: false
    # optional – default true. Set to false to skip certificate management for a stage
    enabled: true
```

Now you can run:

```bash
serverless create-cert
```

To remove the certificate and delete the CNAME record sets from Route 53, run:

```bash
serverless remove-cert
```

Certificates are also automatically created during `serverless deploy` and cleaned up during `serverless remove`.

# Configuration Reference

| Option | Required | Default | Description |
|---|---|---|---|
| `certificateName` | **Yes** | — | The domain name for the certificate |
| `hostedZoneNames` | Yes* | — | Route 53 hosted zone name(s). Required if `hostedZoneIds` is not set |
| `hostedZoneIds` | Yes* | — | Route 53 hosted zone ID(s). Required if `hostedZoneNames` is not set |
| `idempotencyToken` | No | `''` | Idempotency token for the ACM request |
| `region` | No | `us-east-1` | AWS region for the certificate |
| `keyAlgorithm` | No | `EC_prime256v1` | Key algorithm for the certificate (e.g. `RSA_2048`, `EC_prime256v1`) |
| `subjectAlternativeNames` | No | `[]` | Additional domain names for the certificate |
| `tags` | No | — | Key-value pairs to tag the certificate in ACM |
| `writeCertInfoToFile` | No | `false` | Write certificate ARN to a YAML file |
| `certInfoFileName` | No | `cert-info.yml` | Filename(s) for certificate info output |
| `rewriteRecords` | No | `false` | Use UPSERT instead of CREATE for DNS validation records |
| `enabled` | No | `true` | Enable/disable the plugin for a given stage |

# Combine with serverless-domain-manager

If you combine this plugin with [serverless-domain-manager](https://github.com/amplify-education/serverless-domain-manager) you can automate the complete process of creating a custom domain with a certificate.

## Examples

Install the plugins:

```bash
npm i serverless-certificate-creator-v3 --save-dev
npm i serverless-domain-manager --save-dev
```

Open `serverless.yml` and add the following:

```yaml
plugins:
  - serverless-certificate-creator-v3
  - serverless-domain-manager

custom:
  customDomain:
    domainName: abc.somedomain.io
    certificateName: 'abc.somedomain.io'
    basePath: ''
    stage: ${self:provider.stage}
    createRoute53Record: true
  customCertificate:
    certificateName: 'abc.somedomain.io'
    idempotencyToken: 'abcsomedomainio'
    hostedZoneNames: 'somedomain.io.'
    region: eu-west-1
    enabled: true
    rewriteRecords: false
```

Now you can run:

```bash
serverless create-cert
serverless create_domain
```

Please make sure to check out the complete sample project [here](https://github.com/schwamster/serverless-certificate-creator/tree/master/examples/certificate-creator-example).

### Reference Certificate Arn via variableResolvers

Since version 1.2.0 of this plugin you can use the following syntax to access the certificate's ARN in other plugins:

```
${certificate:${self:custom.customCertificate.certificateName}:CertificateArn}
```

If you are on Serverless >= 2.27.0 and have elected to use the variable resolver `variablesResolutionMode: 20210219`, use this syntax:

```
${certificate:${self:custom.customCertificate.certificateName}.CertificateArn}
```

For the variable resolver `variablesResolutionMode: 20210326`, the supported syntax is:

```
${certificate(${self:custom.customCertificate.certificateName}):CertificateArn}
```

See the Serverless [docs](https://serverless.com/framework/docs/providers/aws/guide/plugins#custom-variable-types) for more information.

# Changelog

### v2.2.2
- Add configurable `keyAlgorithm` option (defaults to `EC_prime256v1`)
- Bump AWS SDK and dev-dependency versions

### v2.2.1
- Revert hosted zone ID/name lower-case normalisation

### v2.2.0
- Improve hosted zone handling and matching

### v2.1.0
- Convert codebase to ESM (`"type": "module"`)

### v2.0.1
- Modernize dependencies and imports

### v2.0.0
- **Breaking:** Migrate from AWS SDK v2 to AWS SDK v3 (modular `@aws-sdk/client-acm` and `@aws-sdk/client-route-53`)
- **Breaking:** Rename npm package to `serverless-certificate-creator-v3`
- **Breaking:** Require Node.js >= 18.0.0
- Add Serverless Framework v3 and v4 compatibility
- Add tag synchronisation on existing certificates
- Full async/await rewrite
- Pagination support for 100+ certificates

### v1.5.x and earlier
- See [original repository](https://github.com/schwamster/serverless-certificate-creator) for historical changelog

# License

Copyright (c) 2018 Bastian Töpfer, contributors.

Released under the [MIT license](https://tldrlegal.com/license/mit-license).