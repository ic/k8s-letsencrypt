# Kubernetes + Let's Encrypt Setup Automation

## Before starting

There may be today way better methods to achieve the same. For example:

* [Bitnami tutorial using Helm and the Nginx ingress extension](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-services-with-ingress-tls-letsencrypt/)
* The Kubernetes [ingress-nginx](https://kubernetes.github.io/ingress-nginx/user-guide/tls/) project.

I recommend to have a look at those first. The following is used in production, but there are rought edges. Perhaps the above more "official" initatives are easier to use. As far as I can tell, one possible advantage of this repo is that it automates most steps (all the others I checked need quite a few steps).

## Overview

This piece attempts to setup SSL on a Kubernetes cluster endpoint.

Before:

    curl http://mydomain.com  #=> 200
    curl https://mydomain.com  #=> Timeout, at best

After:

    curl http://mydomain.com  #=> 200
    curl https://mydomain.com  #=> 200

The resulting settings are just the beginning to secure an existing cluster endpoint.

## Usage

### Requirements

The tool assumes a runtime environment:

* Linux or macOS only.
* The Gcloud SDK available on your PATH. It will check for `gcloud` and `kubectl` only.

### CLI

The tool is `setup`. Assuming the requirements are available:

    ./setup <target Gcloud project name> <Glcoud user account> <project name> <desired DNS name> <DNS name administrator email> [production]

All the parameters are required, except the last one (production):

* `target Gcloud project name`: Name of the Gcloud project, as it appears in the Gcloud console. `setup` will login to that project.
* `Gcloud user account`: User account used to `gcloud auth login`. The account must be able to create `cluster-admin` service accounts.
* `project name`: Name of the project (no space allowed). It basically becomes the prefix of all the resources created by `setup`.
* `desired DNS name`: The domain name that you want to secure with Let's Encrypt. You must be able to edit the DNS records.
* `DNS name administrator email`: The email address of an administrator for the domain name. This is required by Let's Encrypt on registration.
* `production`: If empty, `setup` will use [Let's Encrypt staging environment](https://letsencrypt.org/docs/staging-environment/). If set to some value (e.g. "production"), `setup` switches to Let's Encrypt production environment.

### Staging and Production

Please do try the tool first with Let's Encrypt staging environment. This really helps making sure everything gets well, before applying to production. Also, Let's Encrypt rate limits are pretty low, and it is quite frustrating to get blocked and have to wait until *the next week*.

For convenience, `setup` generates a deletion script that cleans up all the resources it created. `setup` reports the script invocation command, basically something like:

    kubectl delete -f /tmp/mytest/delete_myproject.yml

Where `delete_myproject.yml` has been generated for the `myproject` project name.

### Extra details

* The container running `certbot` uses the `epic/kube-nginx-letsencrypt:0.1` image. The `Dockerfile` for that image is [available here](https://github.com/ic/kube-nginx-letsencrypt). This image is based on [prior work](https://github.com/sjenning/kube-nginx-letsencrypt) by @sjenning, with minor changes for usability.

### Gotchas

To date, the container registering to Let's Encrypt may fail a few times before succeeding (hopefully). Please allow for a few minutes! Typically, 2-4 failed containers report logs like:

    > kubectl logs mytest-lscrypt-job-5v77j
    Saving debug log to /var/log/letsencrypt/letsencrypt.log
    Starting new HTTPS connection (1): acme-staging.api.letsencrypt.org
    Obtaining a new certificate
    Performing the following challenges:
    http-01 challenge for mydomain.com
    Using the webroot path /root for all unmatched domains.
    Waiting for verification...
    Cleaning up challenges
    Failed authorization procedure. mydomain.com (http-01): urn:acme:error:unauthorized :: The client lacks sufficient authorization :: Invalid response from http://mydomain.com/.well-known/acme-challenge/sD13uStkK9Sf3o5Pir8zouE0NA7TWytV4Igq3K1gnps: "<!DOCTYPE html>\n<html lang=en>\n  <meta charset=utf-8>\n  <meta name=viewport content=\"initial-scale=1, minimum-scale=1, width=dev"
    ...

At this time, this problem looks like a timing issue. `setup` does wait for the DNS to get updated and point to the cluster endpoint's IP address. Yet the job running the Let's Encrypt container may attempt too early. Just waiting for a few failed attempts has ended to successful completion here.

## Ideas for improvements

Ideas for possible issues and feature requests. The appear here, as there is no plan at this point. Perhaps if there is popular demand! Then, please go ahead and register as GitHub issues.

* The tool does registration, but it does not support renewal (yet, I need it too).
* `setup` does not select the target cluster. Basically need to add: `gcloud config set container/cluster <target cluster name>`
* Remove dependency on the Gcloud SDK, by using the APIs directly.
* Accept more configuration parameter to completely plug into an existing cluster (e.g. specify the name of the Ingress to secure, etc.).

## Acknowledgements

This piece is based on prior work:

* @sjenning [early work](https://github.com/sjenning/kube-nginx-letsencrypt) on a container running `certbot` for Let's Encrypt.
* @thejsj [similar work](https://github.com/thejsj/kubernetes-letsencrypt-demo) on Kubernetes + Let's Encrypt, with the accompanying [blog post](https://runnable.com/blog/how-to-use-lets-encrypt-on-kubernetes).

These earlier works guided me, but did not work entirely. Perhaps they became outdated with newer version of Kubernetes or Docker. Another reason is that you can see in `setup` that the procedure is in two stages. Anyway, I hope this repository helps somewhere.

Work done with the support of [Chikaku, Inc.](https://www.chikaku.co.jp/).
