# TAP Supply Chain With Ability to specify a different location in the registry then the default

## Why do this

Today, in TAP when you configure the build, full or iterate clusters in which images will be built, you define the registry, certificate and image repository within the registry in which to store the different OCI artifacts. these artifacts include the Container image built by the supply chain, as well as the deliverable manifests if using the RegistryOps approach and not doing GitOps.

While using a single harbor project can work for many customers, in many environments, seperation of projects for example in harbor is critical as it allows for better security boundrys and allows giving each team access to only the repositories they need.

Currently in TAP this is not configurable at the workload level and we need to create a custom supply chain, and a few custom templates to make this setting configurable.

## Implementing the custom supply chain
### Requirments

* this has only been tested in internet accessible environments. in Air Gapped deployments, your mileage may vary
* you still need to have the OOTB-Templates package installed as we are only creating some forked templates where needed to make it more configurable.

## How to do this
### Create the needed templates

The next step is to create the cartographer resources with the adapted code to allow configurable image repositories

```bash
kubectl apply  -f kpack-template.yaml
kubectl apply  -f kaniko-template.yaml
kubectl apply  -f delivery-cluster-config-template.yaml
kubectl apply  -f deliverable-template.yaml
kubectl apply  -f config-writer.yaml
kubectl apply  -f config-writer-template.yaml
```

### Creating the custom supply chain
The first step is to fill in the values.yaml file in this repo based on your environment. These can be the same values as you have supplied for the ootb_supply_chain section in your tap installation values.

Once you have set the values in the file, we can now render and apply the supply chain manifest:
```bash
ytt -f values.yaml -f supply-chain.yaml | kubectl apply -f -
```

### Testing out the new supply chain

***Make sure to change the value for the image_repository parameter in the workload yaml to something relevant in your environment***
<br>
To test out the new supply chain, we can use the example workload in this repo
```bash
tanzu apps workload apply -f example/workload.yaml
```