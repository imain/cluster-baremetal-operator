# cluster-baremetal-operator

Introduction

OpenShift supports multiple cloud platform types like AWS, GCP, Azure etc. In addition to these public cloud platforms, OpenShift can also be used to set up a fully functional OpenShift cluster with just Baremetal servers which are either pre-provisioned or are fully provisioned using OpenShift’s Baremetal IPI solution.
The CBO fits into this specific solution by deploying on an OpenShit control plane all the necessary components required to take an unprovisioned server to a fully functional worker node ready to run OpenShift compute workloads. This second level operator was written based on enhancement :
https://github.com/openshift/enhancements/blob/master/enhancements/baremetal/an-slo-for-baremetal.md for details about this operator.

What does it do?

Cluster-baremetal-operator (CBO) is designed to be an OpenShfit Operator that is responsible for deploying all components required to provision Baremetal servers into worker nodes that join an OpenShift cluster. 
The components that have knowledge of how to connect and boot a Baremetal Server are encapsulated in an upstream K8s project called metal3.io. The CBO is responsible for making sure that the metal3 deployment consisting of the baremetal-operator (BMO) and Ironic containers is always running on one of the mater nodes within the OpenShift cluster.
The CBO is also responsible for listening to OpenShift updates to resources that it is watching and take appropriate action. The CBO reports on its own state via the “baremetal” clusterOperator resources as is required by every OpenShift operator.
The CBO reads, validates and passes information provided in the Provisioning Config Resource (CR) and passes this information to the metal3 deployment. It also creates Secrets that containers within the metal3 deployment use to communicate with each other. Currently, only one copy of the Provisioning CR exists per OpenShift Cluster so all worker nodes would be provisioned using the same configuration.


When is CBO active?

CBO runs on all platform types supported by OpenShift but will perform its above mentioned tasks only when the platform type is BareMetal. The “baremetal” ClusterOperator displays the current state of the CBO when running on a BareMetal platform and as “Disabled” in all Platform types. 
When the CBO is running on the BareMetal platform, it manages the metal3 deployment and will continue communicating its state using the “baremetal” ClusterOperator.
CBO is considered a second level operator (SLO) in OpenShift parlance. What that means is that another OpenShift operator is responsible for deploying CBO. In this case, the Cluster Version Operator (CVO) is responsible for deploying its SLOs at a specific run level that is coded into the manifests of that operator.
The OpenShift Installer is responsible for deploying the control plane but does not wait for CVO to complete deployment of the CBO. The CVO, also running on the control plane, is completely responsible for running the CBO on one master node at a time. The worker deployment is completely handled by metal3 which in turn deployed by CBO as mentioned earlier.

What are its inputs?

The CBO needs information about the network to which the baremetal servers are connected and where it can find the image required to boot the servers. This information is provided by the Provisioning CR. CBO watches this resource and passes this information to the metal3 deployment after validating its contents.
The Provisioning CR with details about the config items can be found here. 

What are its outputs?

If not already created by an external entity, the metal3 deployment and its associated Secrets are the result of the CBO. The CBO also creates an image-cache Daemonset that assists the metal3 deployment by downloading the image provided in the Provisioning CR and making it available locally on each master node so that the metal3 deployment would be able to access a local copy of the image while trying to boot a baremetal server.

CBO reports its own state using the “baremetal” CO as mentioned earlier. It is also designed to provide alerts and metrics regarding its own deployment. It is also capable of reporting metrics gathered by BMO regarding the baremetal servers being provisioned. These metrics can then be scraped by Prometheus and can be viewed on the Prometheus dashboard.

CBO will also start having to report on its own health. That feature currently does not exist and is expected to be implemented in the near future.

