This project will not be developed any more, because the overlap of the basic ideas with the much further developed [Argo CD](https://github.com/argoproj/argo-cd) is too high. Please check out Argo CD - it's really neat.

# Wheel GitOps Agent
[![Build Status](https://travis-ci.com/wheel-sh/wheel-agent.svg?branch=master)](https://travis-ci.com/wheel-sh/wheel-agent)

<img src="https://github.com/wheel-sh/wheel-agent/raw/master/img/wheel.png" width="128">

---

The Wheel GitOps Agent enables the administration of Kubernetes Resources like namespaces and its content through a Git repository. The main idea is to combine resource templates (currently OpenShift templates) with its parameters to describe the desired state of a namespace in Git. Wheel establishes a clear structure for organizing the configurations of the desired cluster state. The agent reacts to changes in the cluster or in the Git repository. This provides a simple and traceable way to manage a cluster.

The current Kubernetes target distribution is OpenShift/OKD.

The project maturity is pre-alpha.

---


## Getting Started

These instructions will get you the Wheel GitOps Agent up and running on your OpenShift cluster for development and testing purposes. 

### Prerequisites

You need a running OpenShift cluster where you have cluster-admin permissions. 

If you don't have a cluster, you can use the following options to set up a working one:
* Minishift (provides an OpenShift Cluster locally) https://docs.okd.io/latest/minishift/getting-started/installing.html
* All-in-one cluster (How to get a 'real' cluster on one server running) https://blog.openshift.com/openshift-all-in-one-aio-for-labs-and-fun/


### Installing

First the necessary Git config repository should be created. Here is an example repository that can be used to start for now:

https://github.com/wheel-sh/demo-cluster-config

Just make a fork/copy of this repository.

If possible, the agent should be deployed in its own namespace.

```
oc new-project wheel
```

To deploy the agent, the provided OpenShift template can be used. This creates the following resources:

* DeploymentConfig for the Agent
* Service to enable traffic to the agents REST API to react to Git hooks
* Route to expose the service
* ServiceAccount under which the agent runs
* ClusterRoleBinding to the roles self-access-reviewer, self-provisioner, system:basic-user.
 
```
oc process -f https://raw.githubusercontent.com/wheel-sh/wheel-agent/master/openshift/wheel-agent.yaml \
-p AGENT_NAMESPACE=wheel \
-p CONFIG_REPOSITORY_URL=https://github.com/wheel-sh/demo-cluster-config.git
-p CONFIG_REPOSITORY_BRANCH=master
-p WHEEL_IMAGE=registry.cloud.nikio.io/cicd/wheel-agent:latest \
| oc apply -f -
```

In this example, the parameters must be adjusted accordingly if necessary. The referenced container image is currently in a private registry, but can also be pulled anonymously. Later, the image will also be made available on Docker Hub. However, it is easily possible to build the image yourself.

After the Wheel Agent has been deployed, a Git Hook should be set up for push events. To optain the required hook URL of the agent, execute the following command in the agents namespace:

```
echo "https://$(oc get route wheel-agent --no-headers -o custom-columns=HOST:.spec.host)/git-hook"
```

You can then set up a push hook for this URL in the configuration repository. This makes changes in the repository active in the cluster within a few seconds. 

### Build

To build the Wheel Agent a JDK8, Maven and Docker is needed. To build the container image simply clone this repository and run:

```
mvn package
docker build -t wheel-agent .
```

## Built With

* [Maven](https://maven.apache.org/) - Dependency Management
* [Spring Boot](https://spring.io/projects/spring-boot)
* [JGit](https://www.eclipse.org/jgit/) - As Git API
* [Fabric8 Kubernetes Client](https://github.com/fabric8io/kubernetes-client) - For cluster authentication. 
* [Lombok](https://projectlombok.org/) - To reduce boilerplate code

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on the code of conduct, and the process for submitting pull requests.

## Versioning

[SemVer](http://semver.org/) is used for versioning. 

## Authors

* **Nikolas Philips** - *Creator* - [niiku](https://github.com/niiku)

See also the list of [contributors](https://github.com/wheel-sh/wheel-agent/contributors) who participated in this project.

## License

This project is licensed under the Apache License 2.0 License - see the [LICENSE](https://github.com/wheel-sh/wheel-agent/blob/master/LICENSE) file for details

