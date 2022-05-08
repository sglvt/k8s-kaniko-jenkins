
# Overview

The following steps will use the `values.yaml` and `config.json.template` files from this repository in order to set up a Jenkins server running in Kubernetes, and configuring a Kaniko pod template (Jenkins agent) for it.

The value file is based on the [default value file of the Jenkins chart](https://github.com/helm/charts/blob/master/stable/jenkins/values.yaml),

# Steps
## 1 - Create the namespace
First - create a new namespace, it will make the configuration and subsequent cleanup easier.
```
kubectl create ns kaniko-demo
```
## 2 - Create the registry secret

This is required to push to a registry (Dockerhub in this case), have a look [here](https://github.com/GoogleContainerTools/kaniko#pushing-to-different-registries) for more info.

You'll need to update the `user` and `token` placeholders before executing the following commands.

```
export BASE64_CREDENTIALS=$(echo -n "<user>:<token>"| base64) && \
  curl -sL https://raw.githubusercontent.com/sglvt/k8s-kaniko-jenkins/master/config.json.template \
  | envsubst > config.json && \
  kubectl -n kaniko-demo create secret generic kaniko-secret --from-file=config.json
```

## 3 - Install the Jenkins chart

Optionally, you can have a look at the rendered template using `helm template`, which is a good idea before installing any chart.

The value file is customized to create a pod template with 2 containers: jnlp(Jenkins agent), and kaniko (see the [podTemplates](https://github.com/sglvt/k8s-kaniko-jenkins/blob/master/values.yaml#L557) section - resource requests/limits can be tweaked).

If you want to make further changes, first download the file and update the `--values` argument to point to the local values.yaml file.

To install the chart:

```
helm repo add jenkins https://charts.jenkins.io
helm repo update
export CHART_VERSION=$(helm search repo -o json | jq -r .[0].version)
helm upgrade -i demo jenkins/jenkins \
  --namespace kaniko-demo \
  --version ${CHART_VERSION} \
  --values https://raw.githubusercontent.com/sglvt/k8s-kaniko-jenkins/master/values.yaml \
  --set namespaceOverride=kaniko-demo
```

Jenkins is now deploying, and will soon be accessible with a configuration which supports building container images using Kaniko.

## Additional info
The value file was saved using
```
export CHART_VERSION=$(helm search repo -o json | jq -r .[0].version)
helm show values jenkins/jenkins --version ${CHART_VERSION} > values.${CHART_VERSION}.yaml
```
and edited afterwards