# Installing the Business API Ecosystem on Kubernetes with Helm

The following describes how to setup a full instance of the Business API Ecosystem (BAE). This includes the 
BAE itself, as well as the required databases and an Identity Provider (keyrock)

This repository provides examples of the [Helm values files](./values) which show the minimum configuration 
parameters to be set. Adapt these for your setup before proceeding with the instructions.

The helm chart with all possible configuration values can be found here:
* [Sources](https://github.com/FIWARE/helm-charts/tree/main/charts/business-api-ecosystem)
* [Artifact HUB](https://artifacthub.io/packages/helm/fiware/business-api-ecosystem)



## Preparation

Prepare Helm repositories
```shell
helm repo add t3n https://storage.googleapis.com/t3n-helm-charts
helm repo add elastic https://helm.elastic.co
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add fiware https://fiware.github.io/helm-charts/
helm repo update
```

We will assume that all components will be deployed within the namespace `marketplace`.
```shell
kubectl create ns marketplace
```

In the following we assume that you have control of the domain `domain.org`. Furthermore we assume 
that the externally available components will be accessible on the following subdomains:
* Keyrock IdM: https://keyrock.domain.org
* BAE UI (Logic Proxy): https://marketplace.domain.org



## Databases

First modify the corresponding values files according to your needs and then deploy the required databases MongoDB, MySQL and elasticsearch using `helm`. 
```shell
# Deploy elasticsearch
helm install -f ./values/values-elastic.yml --namespace marketplace elasticsearch elastic/elasticsearch --version 7.5.1

# Deploy MySQL:
helm install -f ./values/values-mysql.yml --namespace marketplace mysql t3n/mysql --version 0.1.0

# Deploy MongoDB
helm install -f ./values/values-mongodb.yml --namespace marketplace mongodb bitnami/mongodb
```



## Identity Provider (Keyrock)

Modify the Keyrock values file according to your needs and deploy the Keyrock Identity Provider. 
Make sure to setup an Ingress or OpenShift route in the values file for external 
access of the UI (e.g. https://keyrock.domain.org).
```shell
# Deploy Keyrock
helm install -f ./values/values-keyrock.yml --namespace marketplace keyrock fiware/keyrock
```

In a browser open the Keyrock UI (e.g. https://keyrock.domain.org) and login with the admin credentials provided in 
the values file.

Create an application for the Business API Ecosystem (Marketplace):
* Set the URL of the Marketplace UI (e.g. https://marketplace.domain.org)
* Set the Callback URL for the OAuth2 Authorization Code flow (e.g. https://marketplace.domain.org/auth/fiware/callback)
* Enable the 'Authorization Code' and 'Refresh Token' grant types

On the next screen, create the following roles:
* admin
* seller
* customer
* orgAdmin

After finishing the creation of the application, note down the shown OAuth2 ClientID and ClientSecret.



## Business API Ecosystem (Marketplace)

Finally install the Business API Ecosystem. Make sure to setup an Ingress or OpenShift route in the values file for external 
access of the Marketplace UI (e.g. https://marketplace.domain.org). Furthermore adapt the configuration options for 
the databases, elasticsearch and Keyrock instances which have been setup before.

In this values file you will also find an example on how to enable a PVC for plugin installation in the charging backend
and theme deployment in the logic proxy. An example of how to build an image for a theme can be
found [here](https://github.com/i4Trust/bae-i4trust-theme).
```shell
# Deploy BAE
helm install -f ./values/values-marketplace.yml --namespace marketplace bae fiware/business-api-ecosystem
```

---
**NOTE**

In order to avoid conflicts with too long container names (max. 63 characters), a short release name should be chosen.

---

The deployment of all components will take some time. When the logic proxy component has been deployed and changed to the running state, 
you can access the marketplace UI via the browser (e.g. http://marketplace.domain.org).

Before logging in, you might need to setup some users within Keyrock and assign corresponding roles (e.g. customer or seller).



## Additional information

### Auth scheme of MongoDB

The Charging Backend of the BAE includes an older MongoDB client which uses an 
outdated auth schema.

Per default, the MongoDB 3.6 installed with this instructions is not using this 
older auth schema. In order to change the auth schema after the databases and users 
have been created, perform the steps in [ChangeMongoDBAuth.md](./ChangeMongoDBAuth.md).

This is required to properly process product orders in the BAE.
