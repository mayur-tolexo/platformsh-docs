---
title: "Ibexa DXP with Fastly"
weight: 1
sidebarTitle: "Fastly"
description: |
  Ibexa DXP is a "commercial extended" version of Ibexa OSS that includes, among other things, support for push-based purging on the Fastly CDN.
---

{{< description >}}

## Remove Varnish configuration

Since eZ Platform 1.13.5, 2.4.3 and 2.5.0, Varnish is enabled by default when deploying on Platform.sh. In order to use Fastly, Varnish must be disabled:

 - Remove environment variable `SYMFONY_TRUSTED_PROXIES: "TRUST_REMOTE"` in [.platform.app.yaml](https://github.com/ezsystems/ezplatform/blob/master/.platform.app.yaml)
 - Remove the Varnish service in [.platform/services.yaml](https://github.com/ezsystems/ezplatform/blob/master/.platform/services.yaml)
 - In [.platform/routes.yaml](https://github.com/ezsystems/ezplatform/blob/master/.platform/routes.yaml), change routes to use `app` instead of the `varnish` service you removed in previous step:

```diff
 "https://{default}/":
     type: upstream
-     upstream: "varnish:http"
+     upstream: "app:http"
```

## Setting up Ibexa DXP to use Fastly

Ibexa DXP's documentation includes instructions on how to [configure Ibexa DXP for Fastly](https://doc.ibexa.co/en/latest/guide/http_cache/#serving-varnish-through-fastly).  Follow the steps there to prepare Ibexa DXP for Fastly.

## Set credentials on Platform.sh

The best way to provide the Fastly credentials and configuration to Ibexa DXP on Platform.sh is via environment variables.  That way private credentials are never stored in Git.

Using the CLI, run the following commands to set the configuration on your master environment.  (Note that they will inherit to all other environments by default unless overridden.)

```bash
platform variable:create -e master --level environment env:HTTPCACHE_PURGE_TYPE --value 'fastly'
platform variable:create -e master --level environment env:FASTLY_SERVICE_ID --value 'YOUR_ID_HERE'
platform variable:create -e master --level environment env:FASTLY_KEY --value 'YOUR_ID_HERE'
```

Replacing `YOUR_ID_HERE` with the Fastly Service ID and Key obtained from Fastly.

Note: On a Platform.sh Dedicated Cluster, set those values on the `production` branch instead:

```bash
platform variable:set -e production env:HTTPCACHE_PURGE_TYPE fastly
platform variable:set -e production env:FASTLY_SERVICE_ID YOUR_ID_HERE
platform variable:set -e production env:FASTLY_KEY YOUR_ID_HERE
```

## Configure Fastly and Platform.sh

See the alternate [Go-live process for Fastly](/domains/cdn/_index.md#client-authenticated-tls) on Platform.sh.  This process is the same for any application.
