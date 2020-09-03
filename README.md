# Packages [![Auto Update Status](https://github.com/cloudposse/packages/workflows/auto-update/badge.svg)](https://github.com/cloudposse/packages/actions?query=workflow%3Aauto-update) [![Latest Release](https://img.shields.io/github/release/cloudposse/packages.svg)](https://github.com/cloudposse/packages/releases/latest) [![Slack Community](https://slack.cloudposse.com/badge.svg)](https://slack.cloudposse.com)

[![README Header][readme_header_img]][readme_header_link]

[![Cloud Posse][logo]](https://cpco.io/homepage)

<!--




  ** DO NOT EDIT THIS FILE
  **
  ** This file was automatically generated by the `build-harness`.
  ** 1) Make all changes to `README.yaml`
  ** 2) Run `make init` (you only need to do this once)
  ** 3) Run`make readme` to rebuild this file.
  **
  ** (We maintain HUNDREDS of open source projects. This is how we maintain our sanity.)
  **





-->

Cloud Posse distribution of awesome apps.


---

This project is part of our comprehensive ["SweetOps"](https://cpco.io/sweetops) approach towards DevOps.
[<img align="right" title="Share via Email" src="https://docs.cloudposse.com/images/ionicons/ios-email-outline-2.0.1-16x16-999999.svg"/>][share_email]
[<img align="right" title="Share on Google+" src="https://docs.cloudposse.com/images/ionicons/social-googleplus-outline-2.0.1-16x16-999999.svg" />][share_googleplus]
[<img align="right" title="Share on Facebook" src="https://docs.cloudposse.com/images/ionicons/social-facebook-outline-2.0.1-16x16-999999.svg" />][share_facebook]
[<img align="right" title="Share on Reddit" src="https://docs.cloudposse.com/images/ionicons/social-reddit-outline-2.0.1-16x16-999999.svg" />][share_reddit]
[<img align="right" title="Share on LinkedIn" src="https://docs.cloudposse.com/images/ionicons/social-linkedin-outline-2.0.1-16x16-999999.svg" />][share_linkedin]
[<img align="right" title="Share on Twitter" src="https://docs.cloudposse.com/images/ionicons/social-twitter-outline-2.0.1-16x16-999999.svg" />][share_twitter]




It's 100% Open Source and licensed under the [APACHE2](LICENSE).












## Introduction


Use this repo to easily install releases of popular Open Source apps. We provide a few ways to use it.

1. **Make Based Installer.** This installer works regardless of your OS and distribution. It downloads packages directly from their GitHub source repos and installs them to your `INSTALL_PATH`. 
2. **Alpine Linux Packages.** Use our Alpine repository to install prebuilt packages that use the original source binary (where possible) from the maintainers' official GitHub repo releases.
3. **Docker Image.** Use our docker image as a base-image or as part of a multi-stage docker build. The docker image always distributes the latest linux binaries for `x86_64` architectures.

See examples below for usage.

**Is one of our packages out of date?**

Open up an [issue](https://github.com/cloudposse/packages/issues) or submit a PR (*preferred*). We'll review quickly!

## Usage




### Alpine Repository (recommended)

A public Alpine repository is provided by [Cloud Posse](https://cloudposse.com). The repository is hosted on Amazon S3 and fronted by [CloudFlare's CDN](http://cloudflare.com) with end-to-end TLS. This ensures insane availability with DDoS mitigation and low-cost hosting. Using this alpine repository is ultimately more reliable than depending on [GitHub for availability](https://twitter.com/githubstatus) and provides an easier way to manage dependencies pinned at multiple versions. 

The repository itself is managed using [`alpinist`](https://github.com/cloudposse/alpinist), which takes care of the heavy lifting of building repository indexes. You can self-host your own Alpine repository using this strategy.

### Configure the alpine repository:

#### The Easy Way

We provide a bootstrap script to configure the alpine repository for your version of alpine. 

```
curl -sSL https://apk.cloudposse.com/install.sh | sh
```
__NOTE__: Requires `bash` and `curl` to run:

#### For Docker

Add the following to your `Dockerfile` near the top.
```
# Install the cloudposse alpine repository
ADD https://apk.cloudposse.com/ops@cloudposse.com.rsa.pub /etc/apk/keys/
RUN echo "@cloudposse https://apk.cloudposse.com/3.11/vendor" >> /etc/apk/repositories
```
__NOTE__: we support alpine `3.7`, `3.8`, `3.9`, `3.10`, and `3.11` packages at this time

### Installing Alpine Packages

When adding packages, we recommend using `apk add --update $package` to update the repository index before installing packages.

Simply install any package as normal:
```
apk add gomplate
```

But we recommend that you use version pinning:
```
apk add gomplate==3.0.0-r0
```

And maybe even repository pinning, so you know that you get our versions:
```
apk add gomplate@cloudposse==3.0.0-r0
```

### Makefile Interface

The `Makefile` interface works on OSX and Linux. It's a great way to distribute binaries in an OS-agnostic way which does not depend on a package manager (e.g. no `brew` or `apt-get`). 

This method is ideal for [local development environments](https://docs.cloudposse.com/local-dev-environments/) (which is how we use it) where you need the dependencies installed natively for your OS/architecture, such as installing a package on OSX.

See all available packages:
```
make -C install help
```

Install everything...
```
make -C install all
```

Install specific packages:
```
make -C install aws-vault chamber
```

Install to a specific folder:
```
make -C install aws-vault INSTALL_PATH=/usr/bin
```

Uninstall a specific package
```
make -C uninstall yq
```




## Examples

### Docker Multi-stage Build

Add this to a `Dockerfile` to install packages using a multi-stage build process:
```
FROM cloudposse/packages:latest AS packages

COPY --from=packages /packages/bin/kubectl /usr/local/bin/
```

### Docker with Git Clone

Or... add this to a `Dockerfile` to easily install packages on-demand:
```
RUN git clone --depth=1 -b master https://github.com/cloudposse/packages.git /packages && \
    rm -rf /packages/.git && \
    make -C /packages/install kubectl
```

### Makefile Inclusion

Sometimes it's necessary to install some binary dependencies when building projects. For example, we frequently 
rely on `gomplate` or `helm` to build chart packages.

Here's a stub you can include into a `Makefile` to make it easier to install binary dependencies.

```
export PACKAGES_VERSION ?= master
export PACKAGES_PATH ?= packages/
export INSTALL_PATH ?= $(PACKAGES_PATH)/bin

## Install packages
packages/install:
        @if [ ! -d $(PACKAGES_PATH) ]; then \
          echo "Installing packages $(PACKAGES_VERSION)..."; \
          rm -rf $(PACKAGES_PATH); \
          git clone --depth=1 -b $(PACKAGES_VERSION) https://github.com/cloudposse/packages.git $(PACKAGES_PATH); \
          rm -rf $(PACKAGES_PATH)/.git; \
        fi

## Install package (e.g. helm, helmfile, kubectl)
packages/install/%: packages/install
        @make -C $(PACKAGES_PATH)/install $(subst packages/install/,,$@)

## Uninstall package (e.g. helm, helmfile, kubectl)
packages/uninstall/%:
        @make -C $(PACKAGES_PATH)/uninstall $(subst packages/uninstall/,,$@)
```

### Contributing Additional Packages
In addition to following the Contributing section, the following steps can be used to add new packages for review (via a PR).
1. Clone an existing, similar, package within the vendors directory. Name the new folder with the same name as the binary package being installed.
2. At a minimum, update the `VERSION`, `DESCRIPTION`, and `Makefile` to reflect the binary being installed. Ensure that a test task exist in the package Makefile.
3. Test the install and ensure that it downloads and runs as expected (`make -C install <your_package> INSTALL_PATH=/tmp`)
4. Test the apk build (see below)
5. Update the `README.md` (`make init readme/deps readme`)

### Testing apk builds

To validate that a new package will build into an apk you can use the following steps;

```bash
make docker/build/apk/shell
make -C vendor/<appname> apk
# Some temp build files in the volume mount set user/group to nobody/nobody for apk building.
# It is easier to remove them while within the docker container.
rm -rf ./tmp/build.*
exit
```



## Package Build Status
| Build Status | Version | Description |
| ------------ | ------- | ----------- |
![amtool](https://github.com/cloudposse/packages/workflows/amtool/badge.svg?branch=master) | 0.21.0     | Tool for interacting with the Alertmanager API
![argocd](https://github.com/cloudposse/packages/workflows/argocd/badge.svg?branch=master) | 1.7.2      | Declarative GitOpts for Kubernetes
![assume-role](https://github.com/cloudposse/packages/workflows/assume-role/badge.svg?branch=master) | 0.3.2      | Easily assume AWS roles in your terminal.
![atlantis](https://github.com/cloudposse/packages/workflows/atlantis/badge.svg?branch=master) | 0.15.0     | Terraform For Teams
![awless](https://github.com/cloudposse/packages/workflows/awless/badge.svg?branch=master) | 0.1.11     | A Mighty CLI for AWS
![aws-iam-authenticator](https://github.com/cloudposse/packages/workflows/aws-iam-authenticator/badge.svg?branch=master) | 0.5.1      | A tool to use AWS IAM credentials to authenticate to a Kubernetes cluster
![aws-okta](https://github.com/cloudposse/packages/workflows/aws-okta/badge.svg?branch=master) | 0.19.4     | aws-okta allows users to authenticate with AWS using Okta credentials
![aws-vault](https://github.com/cloudposse/packages/workflows/aws-vault/badge.svg?branch=master) | 5.4.4      | A vault for securely storing and accessing AWS credentials in development environments
![cfssl](https://github.com/cloudposse/packages/workflows/cfssl/badge.svg?branch=master) | 1.4.1      | Cloudflare's PKI and TLS toolkit
![cfssljson](https://github.com/cloudposse/packages/workflows/cfssljson/badge.svg?branch=master) | 1.4.1      | Cloudflare's PKI and TLS toolkit json parser
![chamber](https://github.com/cloudposse/packages/workflows/chamber/badge.svg?branch=master) | 2.8.2      | CLI for managing secrets
![cli53](https://github.com/cloudposse/packages/workflows/cli53/badge.svg?branch=master) | 0.8.17     | Command line tool for Amazon Route 53
![cloudflared](https://github.com/cloudposse/packages/workflows/cloudflared/badge.svg?branch=master) | 2020.3.2   | Argo Tunnel client
![cloudposse-atlantis](https://github.com/cloudposse/packages/workflows/cloudposse-atlantis/badge.svg?branch=master) | 0.9.0.3    | Terraform For Teams, enhanced by Cloud Posse
![codefresh](https://github.com/cloudposse/packages/workflows/codefresh/badge.svg?branch=master) | 0.72.1     | Codefresh CLI
![consul](https://github.com/cloudposse/packages/workflows/consul/badge.svg?branch=master) | 1.8.3      | Hashicorp consul
![ctop](https://github.com/cloudposse/packages/workflows/ctop/badge.svg?branch=master) | 0.7.3      | Top-like interface for container metrics
![direnv](https://github.com/cloudposse/packages/workflows/direnv/badge.svg?branch=master) | 2.21.3     | Unclutter your .profile
![doctl](https://github.com/cloudposse/packages/workflows/doctl/badge.svg?branch=master) | 1.46.0     | A command line tool for DigitalOcean services
![duffle](https://github.com/cloudposse/packages/workflows/duffle/badge.svg?branch=master) | 0.3.5b1    | CNAB installer
![emailcli](https://github.com/cloudposse/packages/workflows/emailcli/badge.svg?branch=master) | 1.0.3      | Command line email sending client written in Go.
![fargate](https://github.com/cloudposse/packages/workflows/fargate/badge.svg?branch=master) | 0.3.2      | CLI for AWS Fargate
![fetch](https://github.com/cloudposse/packages/workflows/fetch/badge.svg?branch=master) | 0.3.10     | fetch makes it easy to download files, folders, and release assets from a specific public git commit, branch, or tag
![figurine](https://github.com/cloudposse/packages/workflows/figurine/badge.svg?branch=master) | 1.0.1      | Print your name in style
![fzf](https://github.com/cloudposse/packages/workflows/fzf/badge.svg?branch=master) | 0.22.0     | A command-line fuzzy finder
![gh](https://github.com/cloudposse/packages/workflows/gh/badge.svg?branch=master) | 0.11.1     | The GitHub CLI
![ghr](https://github.com/cloudposse/packages/workflows/ghr/badge.svg?branch=master) | 0.13.0     | Upload multiple artifacts to GitHub Releases in parallel
![github-commenter](https://github.com/cloudposse/packages/workflows/github-commenter/badge.svg?branch=master) | 0.7.0      | Command line utility for creating GitHub comments on Commits, Pull Request Reviews or Issues
![github-release](https://github.com/cloudposse/packages/workflows/github-release/badge.svg?branch=master) | 0.8.1      | Commandline app to create and edit releases on Github (and upload artifacts)
![github-status-updater](https://github.com/cloudposse/packages/workflows/github-status-updater/badge.svg?branch=master) | 0.5.0      | Command line utility for updating GitHub commit statuses and enabling required status checks for pull requests
![gitleaks](https://github.com/cloudposse/packages/workflows/gitleaks/badge.svg?branch=master) | 1.2.0      | Audit git repos for secrets 🔑
![gomplate](https://github.com/cloudposse/packages/workflows/gomplate/badge.svg?branch=master) | 3.8.0      | A flexible commandline tool for template rendering. Supports lots of local and remote datasources.
![gonsul](https://github.com/cloudposse/packages/workflows/gonsul/badge.svg?branch=master) | 0.2.1      | A stand-alone alternative to git2consul 
![goofys](https://github.com/cloudposse/packages/workflows/goofys/badge.svg?branch=master) | 0.24.0     | a high-performance, POSIX-ish Amazon S3 file system written in Go
![gosu](https://github.com/cloudposse/packages/workflows/gosu/badge.svg?branch=master) | 1.12       | Simple Go-based setuid+setgid+setgroups+exec
![gotop](https://github.com/cloudposse/packages/workflows/gotop/badge.svg?branch=master) | 3.0.0      | A terminal based graphical activity monitor inspired by gtop and vtop
![helm](https://github.com/cloudposse/packages/workflows/helm/badge.svg?branch=master) | 3.3.1      | The Kubernetes Package Manager
![helm2](https://github.com/cloudposse/packages/workflows/helm2/badge.svg?branch=master) | 2.16.10    | The Kubernetes Package Manager
![helm3](https://github.com/cloudposse/packages/workflows/helm3/badge.svg?branch=master) | 3.3.1      | The Kubernetes Package Manager
![helmfile](https://github.com/cloudposse/packages/workflows/helmfile/badge.svg?branch=master) | 0.126.2    | Deploy Kubernetes Helm Charts
![htmltest](https://github.com/cloudposse/packages/workflows/htmltest/badge.svg?branch=master) | 0.13.0     | :white_check_mark: Test generated HTML for problems
![hugo](https://github.com/cloudposse/packages/workflows/hugo/badge.svg?branch=master) | 0.74.3     | The world’s fastest framework for building websites.
![jp](https://github.com/cloudposse/packages/workflows/jp/badge.svg?branch=master) | 0.1.3      | Command line interface to JMESPath
![jq](https://github.com/cloudposse/packages/workflows/jq/badge.svg?branch=master) | 1.6        | Command-line JSON processor
![json2hcl](https://github.com/cloudposse/packages/workflows/json2hcl/badge.svg?branch=master) | 0.0.6      | Convert JSON to HCL, and vice versa
![jx](https://github.com/cloudposse/packages/workflows/jx/badge.svg?branch=master) | 2.1.127    | Jenkins-X
![k3d](https://github.com/cloudposse/packages/workflows/k3d/badge.svg?branch=master) | 3.0.1      | Little helper to run Rancher Lab's k3s in Docker
![k6](https://github.com/cloudposse/packages/workflows/k6/badge.svg?branch=master) | 0.27.1     | A modern load testing tool, using Go and JavaScript - https://k6.io
![k9s](https://github.com/cloudposse/packages/workflows/k9s/badge.svg?branch=master) | 0.21.7     | Kubernetes CLI To Manage Your Clusters In Style
![katafygio](https://github.com/cloudposse/packages/workflows/katafygio/badge.svg?branch=master) | 0.8.1      | K8s continuous backup to git
![kfctl](https://github.com/cloudposse/packages/workflows/kfctl/badge.svg?branch=master) | 1.0        | Machine Learning Toolkit for Kubernetes
![kind](https://github.com/cloudposse/packages/workflows/kind/badge.svg?branch=master) | 0.8.1      | A tool for running local Kubernetes clusters using Docker
![kops](https://github.com/cloudposse/packages/workflows/kops/badge.svg?branch=master) | 1.18.0     | Kubernetes Operations (kops) - Production Grade K8s Installation, Upgrades, and Management
![kops-1.12](https://github.com/cloudposse/packages/workflows/kops-1.12/badge.svg?branch=master) | 1.12.3     | Kubernetes Operations (kops) - Production Grade K8s Installation, Upgrades, and Management
![krew](https://github.com/cloudposse/packages/workflows/krew/badge.svg?branch=master) | 0.4.0      | Kubectl plugin manager
![kubecron](https://github.com/cloudposse/packages/workflows/kubecron/badge.svg?branch=master) | 1.0.2      | Utilities to manage kubernetes cronjobs. Run a CronJob manually for test purposes. Suspend/unsuspend a CronJob
![kubectl](https://github.com/cloudposse/packages/workflows/kubectl/badge.svg?branch=master) | 1.19.0     | Production-Grade Container Scheduling and Management
![kubectl-1.13](https://github.com/cloudposse/packages/workflows/kubectl-1.13/badge.svg?branch=master) | 1.13.12    | Production-Grade Container Scheduling and Management (v1.13)
![kubectl-1.14](https://github.com/cloudposse/packages/workflows/kubectl-1.14/badge.svg?branch=master) | 1.14.10    | Production-Grade Container Scheduling and Management (v1.14)
![kubectl-1.15](https://github.com/cloudposse/packages/workflows/kubectl-1.15/badge.svg?branch=master) | 1.15.12    | Production-Grade Container Scheduling and Management (v1.15)
![kubectl-1.16](https://github.com/cloudposse/packages/workflows/kubectl-1.16/badge.svg?branch=master) | 1.16.14    | Production-Grade Container Scheduling and Management (v1.16)
![kubectl-1.17](https://github.com/cloudposse/packages/workflows/kubectl-1.17/badge.svg?branch=master) | 1.17.11    | Production-Grade Container Scheduling and Management (v1.17)
![kubectl-1.18](https://github.com/cloudposse/packages/workflows/kubectl-1.18/badge.svg?branch=master) | 1.18.8     | Production-Grade Container Scheduling and Management (v1.18)
![kubectl-1.19](https://github.com/cloudposse/packages/workflows/kubectl-1.19/badge.svg?branch=master) | 1.19.0     | Production-Grade Container Scheduling and Management (v1.19)
![kubectx](https://github.com/cloudposse/packages/workflows/kubectx/badge.svg?branch=master) | 0.9.1      | Switch faster between clusters and namespaces in kubectl
![kubens](https://github.com/cloudposse/packages/workflows/kubens/badge.svg?branch=master) | 0.9.1      | Switch faster between clusters and namespaces in kubectl
![kubeval](https://github.com/cloudposse/packages/workflows/kubeval/badge.svg?branch=master) | 0.15.0     | Validate your Kubernetes configuration files, supports multiple Kubernetes versions
![lazydocker](https://github.com/cloudposse/packages/workflows/lazydocker/badge.svg?branch=master) | 0.9        | The lazier way to manage everything docker
![lectl](https://github.com/cloudposse/packages/workflows/lectl/badge.svg?branch=master) | 0.21       | Script to check issued certificates by Let's Encrypt on CTL (Certificate Transparency Log) using https://crt.sh
![minikube](https://github.com/cloudposse/packages/workflows/minikube/badge.svg?branch=master) | 1.12.3     | Run Kubernetes locally
![misspell](https://github.com/cloudposse/packages/workflows/misspell/badge.svg?branch=master) | 0.3.4      | Correct commonly misspelled English words in source files
![nomad](https://github.com/cloudposse/packages/workflows/nomad/badge.svg?branch=master) | 0.12.3     | Hashicorp nomad
![pack](https://github.com/cloudposse/packages/workflows/pack/badge.svg?branch=master) | 0.13.1     | Create cloud native Buildpacks
![packer](https://github.com/cloudposse/packages/workflows/packer/badge.svg?branch=master) | 1.6.2      | Packer is a tool for creating identical machine images for multiple platforms from a single source configuration.
![pandoc](https://github.com/cloudposse/packages/workflows/pandoc/badge.svg?branch=master) | 2.10.1     | Universal markup converter
![pgmetrics](https://github.com/cloudposse/packages/workflows/pgmetrics/badge.svg?branch=master) | 1.9.3      | Postgres metrics
![pluto](https://github.com/cloudposse/packages/workflows/pluto/badge.svg?branch=master) | 3.4.1      | A cli tool to help discover deprecated apiVersions in Kubernetes
![popeye](https://github.com/cloudposse/packages/workflows/popeye/badge.svg?branch=master) | 0.8.10     | A Kubernetes cluster resource sanitizer
![promtool](https://github.com/cloudposse/packages/workflows/promtool/badge.svg?branch=master) | 2.20.1     | Prometheus CLI tool
![rakkess](https://github.com/cloudposse/packages/workflows/rakkess/badge.svg?branch=master) | 0.4.5      | Review Access - kubectl plugin to show an access matrix for all available resources
![rancher](https://github.com/cloudposse/packages/workflows/rancher/badge.svg?branch=master) | 2.4.6      | Rancher CLI
![rbac-lookup](https://github.com/cloudposse/packages/workflows/rbac-lookup/badge.svg?branch=master) | 0.6.1      | Find Kubernetes roles and cluster roles bound to any user, service account, or group name.
![retry](https://github.com/cloudposse/packages/workflows/retry/badge.svg?branch=master) | 3.3.0      | ♻️ Functional mechanism based on channels to perform actions repetitively until successful.
![saml2aws](https://github.com/cloudposse/packages/workflows/saml2aws/badge.svg?branch=master) | 2.27.0     | CLI tool which enables you to login and retrieve AWS temporary credentials using a SAML IDP
![scenery](https://github.com/cloudposse/packages/workflows/scenery/badge.svg?branch=master) | 0.1.5      | A Terraform plan output prettifier
![sentinel](https://github.com/cloudposse/packages/workflows/sentinel/badge.svg?branch=master) | 0.14.2     | Hashicorp sentinel
![sentry-cli](https://github.com/cloudposse/packages/workflows/sentry-cli/badge.svg?branch=master) | 1.55.2     | A command line utility to work with Sentry.
![shellcheck](https://github.com/cloudposse/packages/workflows/shellcheck/badge.svg?branch=master) | 0.7.1      | ShellCheck, a static analysis tool for shell scripts
![shfmt](https://github.com/cloudposse/packages/workflows/shfmt/badge.svg?branch=master) | 3.1.2      | A shell parser, formatter and interpreter (POSIX/Bash/mksh)
![slack-notifier](https://github.com/cloudposse/packages/workflows/slack-notifier/badge.svg?branch=master) | 0.3.0      | Command line utility to send messages with attachments to Slack channels via Incoming Webhooks
![sops](https://github.com/cloudposse/packages/workflows/sops/badge.svg?branch=master) | 3.6.0      | Secrets management stinks, use some sops!
![stern](https://github.com/cloudposse/packages/workflows/stern/badge.svg?branch=master) | 1.11.0     | ⎈ Multi pod and container log tailing for Kubernetes
![sudosh](https://github.com/cloudposse/packages/workflows/sudosh/badge.svg?branch=master) | 0.3.0      | Shell wrapper to run a login shell with `sudo` as the current user for the purpose of audit logging
![teleport](https://github.com/cloudposse/packages/workflows/teleport/badge.svg?branch=master) | 4.2.11     | Privileged access management for elastic infrastructure.
![terraform](https://github.com/cloudposse/packages/workflows/terraform/badge.svg?branch=master) | 0.13.1     | Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.
![terraform-0.11](https://github.com/cloudposse/packages/workflows/terraform-0.11/badge.svg?branch=master) | 0.11.14    | Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.
![terraform-0.12](https://github.com/cloudposse/packages/workflows/terraform-0.12/badge.svg?branch=master) | 0.12.29    | Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.
![terraform-0.13](https://github.com/cloudposse/packages/workflows/terraform-0.13/badge.svg?branch=master) | 0.13.1     | Terraform is a tool for building, changing, and combining infrastructure safely and efficiently.
![terraform-docs](https://github.com/cloudposse/packages/workflows/terraform-docs/badge.svg?branch=master) | 0.9.1      | Generate docs from terraform modules
![terraform_0.11](https://github.com/cloudposse/packages/workflows/terraform_0.11/badge.svg?branch=master) | 0.11.14    | Terraform (Deprecated package. Use terraform-0.11 instead)
![terraform_0.12](https://github.com/cloudposse/packages/workflows/terraform_0.12/badge.svg?branch=master) | 0.12.29    | Terraform (Deprecated package. Use terraform-0.12 instead)
![terraform_0.13](https://github.com/cloudposse/packages/workflows/terraform_0.13/badge.svg?branch=master) | 0.13.1     | Terraform (Deprecated package. Use terraform-0.13 instead)
![terragrunt](https://github.com/cloudposse/packages/workflows/terragrunt/badge.svg?branch=master) | 0.23.38    | Terragrunt is a thin wrapper for Terraform that provides extra tools for working with multiple Terraform modules.
![terrahelp](https://github.com/cloudposse/packages/workflows/terrahelp/badge.svg?branch=master) | 0.7.4      | Terrahelp is as a command line utility that provides useful tricks like masking of terraform output.
![tfenv](https://github.com/cloudposse/packages/workflows/tfenv/badge.svg?branch=master) | 0.4.0      | Transform environment variables for use with Terraform (e.g. `HOSTNAME` ⇨ `TF_VAR_hostname`)
![tfmask](https://github.com/cloudposse/packages/workflows/tfmask/badge.svg?branch=master) | 0.7.0      | Terraform utility to mask select output from `terraform plan` and `terraform apply`
![thanos](https://github.com/cloudposse/packages/workflows/thanos/badge.svg?branch=master) | 0.14.0     | Highly available Prometheus setup with long term storage capabilities. CNCF Sandbox project.
![trivy](https://github.com/cloudposse/packages/workflows/trivy/badge.svg?branch=master) | 0.11.0     | A Simple and Comprehensive Vulnerability Scanner for Containers, Suitable for CI
![variant](https://github.com/cloudposse/packages/workflows/variant/badge.svg?branch=master) | 0.36.5     | Variant is a Universal CLI tool that works like a task runner
![variant2](https://github.com/cloudposse/packages/workflows/variant2/badge.svg?branch=master) | 0.33.0     | Second major version of Variant, a Universal CLI tool that works like a task runner
![vault](https://github.com/cloudposse/packages/workflows/vault/badge.svg?branch=master) | 1.5.3      | Hashicorp vault
![venona](https://github.com/cloudposse/packages/workflows/venona/badge.svg?branch=master) | 0.32.1     | Codefresh runtime-environment agent
![vert](https://github.com/cloudposse/packages/workflows/vert/badge.svg?branch=master) | 0.1.0      | Simple CLI for comparing two or more versions
![yq](https://github.com/cloudposse/packages/workflows/yq/badge.svg?branch=master) | 3.3.2      | yq is a portable command-line YAML processor



## Share the Love

Like this project? Please give it a ★ on [our GitHub](https://github.com/cloudposse/packages)! (it helps us **a lot**)

Are you using this project or any of our other projects? Consider [leaving a testimonial][testimonial]. =)


## Related Projects

Check out these related projects.

- [build-harness](https://github.com/cloudposse/build-harness) - Collection of Makefiles to facilitate building Golang projects, Dockerfiles, Helm charts, and more
- [geodesic](https://github.com/cloudposse/geodesic) - Geodesic is the fastest way to get up and running with a rock solid, production grade cloud platform built on strictly Open Source tools.



## Help

**Got a question?** We got answers.

File a GitHub [issue](https://github.com/cloudposse/packages/issues), send us an [email][email] or join our [Slack Community][slack].

[![README Commercial Support][readme_commercial_support_img]][readme_commercial_support_link]

## DevOps Accelerator for Startups


We are a [**DevOps Accelerator**][commercial_support]. We'll help you build your cloud infrastructure from the ground up so you can own it. Then we'll show you how to operate it and stick around for as long as you need us.

[![Learn More](https://img.shields.io/badge/learn%20more-success.svg?style=for-the-badge)][commercial_support]

Work directly with our team of DevOps experts via email, slack, and video conferencing.

We deliver 10x the value for a fraction of the cost of a full-time engineer. Our track record is not even funny. If you want things done right and you need it done FAST, then we're your best bet.

- **Reference Architecture.** You'll get everything you need from the ground up built using 100% infrastructure as code.
- **Release Engineering.** You'll have end-to-end CI/CD with unlimited staging environments.
- **Site Reliability Engineering.** You'll have total visibility into your apps and microservices.
- **Security Baseline.** You'll have built-in governance with accountability and audit logs for all changes.
- **GitOps.** You'll be able to operate your infrastructure via Pull Requests.
- **Training.** You'll receive hands-on training so your team can operate what we build.
- **Questions.** You'll have a direct line of communication between our teams via a Shared Slack channel.
- **Troubleshooting.** You'll get help to triage when things aren't working.
- **Code Reviews.** You'll receive constructive feedback on Pull Requests.
- **Bug Fixes.** We'll rapidly work with you to fix any bugs in our projects.

## Slack Community

Join our [Open Source Community][slack] on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

## Discourse Forums

Participate in our [Discourse Forums][discourse]. Here you'll find answers to commonly asked questions. Most questions will be related to the enormous number of projects we support on our GitHub. Come here to collaborate on answers, find solutions, and get ideas about the products and services we value. It only takes a minute to get started! Just sign in with SSO using your GitHub account.

## Newsletter

Sign up for [our newsletter][newsletter] that covers everything on our technology radar.  Receive updates on what we're up to on GitHub as well as awesome new projects we discover.

## Office Hours

[Join us every Wednesday via Zoom][office_hours] for our weekly "Lunch & Learn" sessions. It's **FREE** for everyone!

[![zoom](https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png")][office_hours]

## Contributing

### Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/packages/issues) to report any bugs or file feature requests.

### Developing

If you are interested in being a contributor and want to get involved in developing this project or [help out](https://cpco.io/help-out) with our other projects, we would love to hear from you! Shoot us an [email][email].

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.

 1. **Fork** the repo on GitHub
 2. **Clone** the project to your own machine
 3. **Commit** changes to your own branch
 4. **Push** your work back up to your fork
 5. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!


## Copyright

Copyright © 2017-2020 [Cloud Posse, LLC](https://cpco.io/copyright)



## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

```text
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```









## Trademarks

All other trademarks referenced herein are the property of their respective owners.

## About

This project is maintained and funded by [Cloud Posse, LLC][website]. Like it? Please let us know by [leaving a testimonial][testimonial]!

[![Cloud Posse][logo]][website]

We're a [DevOps Professional Services][hire] company based in Los Angeles, CA. We ❤️  [Open Source Software][we_love_open_source].

We offer [paid support][commercial_support] on all of our projects.

Check out [our other projects][github], [follow us on twitter][twitter], [apply for a job][jobs], or [hire us][hire] to help with your cloud strategy and implementation.



### Contributors

|  [![Erik Osterman][osterman_avatar]][osterman_homepage]<br/>[Erik Osterman][osterman_homepage] | [![Igor Rodionov][goruha_avatar]][goruha_homepage]<br/>[Igor Rodionov][goruha_homepage] | [![Andriy Knysh][aknysh_avatar]][aknysh_homepage]<br/>[Andriy Knysh][aknysh_homepage] |
|---|---|---|

  [osterman_homepage]: https://github.com/osterman
  [osterman_avatar]: https://img.cloudposse.com/150x150/https://github.com/osterman.png
  [goruha_homepage]: https://github.com/goruha
  [goruha_avatar]: https://img.cloudposse.com/150x150/https://github.com/goruha.png
  [aknysh_homepage]: https://github.com/aknysh
  [aknysh_avatar]: https://img.cloudposse.com/150x150/https://github.com/aknysh.png

[![README Footer][readme_footer_img]][readme_footer_link]
[![Beacon][beacon]][website]

  [logo]: https://cloudposse.com/logo-300x69.svg
  [docs]: https://cpco.io/docs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=docs
  [website]: https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=website
  [github]: https://cpco.io/github?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=github
  [jobs]: https://cpco.io/jobs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=jobs
  [hire]: https://cpco.io/hire?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=hire
  [slack]: https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=slack
  [linkedin]: https://cpco.io/linkedin?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=linkedin
  [twitter]: https://cpco.io/twitter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=twitter
  [testimonial]: https://cpco.io/leave-testimonial?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=testimonial
  [office_hours]: https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=office_hours
  [newsletter]: https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=newsletter
  [discourse]: https://ask.sweetops.com/?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=discourse
  [email]: https://cpco.io/email?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=email
  [commercial_support]: https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=commercial_support
  [we_love_open_source]: https://cpco.io/we-love-open-source?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=we_love_open_source
  [terraform_modules]: https://cpco.io/terraform-modules?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=terraform_modules
  [readme_header_img]: https://cloudposse.com/readme/header/img
  [readme_header_link]: https://cloudposse.com/readme/header/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=readme_header_link
  [readme_footer_img]: https://cloudposse.com/readme/footer/img
  [readme_footer_link]: https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=readme_footer_link
  [readme_commercial_support_img]: https://cloudposse.com/readme/commercial-support/img
  [readme_commercial_support_link]: https://cloudposse.com/readme/commercial-support/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/packages&utm_content=readme_commercial_support_link
  [share_twitter]: https://twitter.com/intent/tweet/?text=Packages&url=https://github.com/cloudposse/packages
  [share_linkedin]: https://www.linkedin.com/shareArticle?mini=true&title=Packages&url=https://github.com/cloudposse/packages
  [share_reddit]: https://reddit.com/submit/?url=https://github.com/cloudposse/packages
  [share_facebook]: https://facebook.com/sharer/sharer.php?u=https://github.com/cloudposse/packages
  [share_googleplus]: https://plus.google.com/share?url=https://github.com/cloudposse/packages
  [share_email]: mailto:?subject=Packages&body=https://github.com/cloudposse/packages
  [beacon]: https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/packages?pixel&cs=github&cm=readme&an=packages
