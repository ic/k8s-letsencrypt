# Kubernetes + Let's Encrypt Setup Automation

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


## Ideas for improvements

Ideas for possible issues and feature requests. The appear here, as there is no plan at this point. Perhaps if there is popular demand! Then, please go ahead and register as GitHub issues.

* `setup` does not select the target cluster. Basically need to add: `gcloud config set container/cluster <target cluster name>`
* Remove dependency on the Gcloud SDK, by using the APIs directly.
* Accept more configuration parameter to completely plug into an existing cluster (e.g. specify the name of the Ingress to secure, etc.).

## Acknowledgements

This piece is based on prior work:

* @sjenning [early work](https://github.com/sjenning/kube-nginx-letsencrypt) on a container running `certbot` for Let's Encrypt.
* @thejsj [similar work](https://github.com/thejsj/kubernetes-letsencrypt-demo) on Kubernetes + Let's Encrypt, with the accompanying [blog post](https://runnable.com/blog/how-to-use-lets-encrypt-on-kubernetes).

These earlier works guided me, but did not work entirely. Perhaps they became outdated with newer version of Kubernetes or Docker. Another reason is that you can see in `setup` that the procedure is in two stages. Anyway, I hope this repository helps somewhere.

Work done with the support of [Chikaku, Inc.](https://www.chikaku.co.jp/).
