# Deploy a Kubernetes cluster to AWS and Install Gitlab on the cluster

## Pre-requisites

### Install dependencies

```
$ brew install kops kubernetes-cli kubernetes-helm
$ curl --output /usr/local/bin/gitlab-runner https://gitlab-ci-multi-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-ci-multi-runner-darwin-amd64
$ chmod +x /usr/local/bin/gitlab-runner
```

## Installation

### Kubernetes

*Create the cluster*

```
$ kops create cluster \
    --node-count 3 \
    --zones $(ZONE) \
    --master-zones $(ZONE) \
    --dns-zone $(DNS_ZONE) \
    --node-size m4.large \
    --master-size t2.medium \
    $(NAME)
```

*Install the Kubernetes Dashboard*

```
$ kubectl create -f https://git.io/kube-dashboard
```

You can reach the dashboard by running `kubectl proxy`.

## GitLab

*Install the GitLab chart:*

```
$ helm repo add gitlab https://charts.gitlab.io
$ helm init
$ helm install --namespace gitlab --name gitlab -f gitlab-config.yaml gitlab/gitlab
```

*Install the GitLab runner chart:*

```
$ helm install \
    --namespace gitlab \
    --name gitlab-runner \
    -f gitlab-runner.yml \
    gitlab/gitlab-runner
```

*Register the Docker runner:*

```
$ gitlab-runner register -n \
    --url http://gitlab.dq.berryphillips.com/ \
    --registration-token $(RUNNER_TOKEN) \
    --executor docker \
    --description "Docker Runner" \
    --docker-image "docker:latest" \
    --docker-volumes /var/run/docker.sock:/var/run/docker.sock
```
