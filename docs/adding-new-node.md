# Adding New Worker Node

## Adding the node in k8s

In order to add a new worker node, we follow the steps as outlined by the kubespray project.
Lets assume we are adding one new worker node: `computegpu001.p40.example.com` and add to relevant sections.

1. Add the node to your ansible inventory file
```shell
   vim /etc/genestack/inventory/openstack-flex-inventory.ini
```

2. Ensure hostname is correctly set and hosts file has 127.0.0.1 entry

3. Run scale.yaml to add the node to your cluster
```shell
   ansible-playbook -i /etc/genestack/inventory/openstack-flex-inventory.yaml scale.yml --limit computegpu001.p40.example.com --become
```

Once step 3 competes succesfully, validate that the node is up and running in the cluster
```shell
   kubectl get nodes | grep computegpu001.p40.example.com
```

## Adding the node in openstack

Once the node is added in k8s cluster, adding the node to openstack service is simply a matter of labeling the node with the right
labels and annotations.

1. Export the nodes to add
```shell
   export NODES='computegpu001.p40.example.com'
```

2. For compute node add the following labels
```shell
   # Label the openstack compute nodes
   kubectl label node computegpu001.p40.example.com openstack-compute-node=enabled

   # With OVN we need the compute nodes to be "network" nodes as well. While they will be configured for networking, they wont be gateways.
   kubectl label node computegpu001.p40.example.com openstack-network-node=enabled
```

3. Add the right annotations to the node
```shell
   kubectl annotate \
        nodes \
        ${NODES} \
        ovn.openstack.org/int_bridge='br-int'

   kubectl annotate \
        nodes \
        ${NODES} \
        ovn.openstack.org/bridges='br-ex'

   kubectl annotate \
        nodes \
        ${NODES} \
        ovn.openstack.org/ports='br-ex:bond1'

   kubectl annotate \
        nodes \
        ${NODES} \
        ovn.openstack.org/mappings='physnet1:br-ex'

   kubectl annotate \
        nodes \
        ${NODES} \
        ovn.openstack.org/availability_zones='nova'
```

4. Verify all the services are up and running
```shell
   kubectl get pods -n openstack -o wide | grep "computegpu"
```

At this point the compute node should be up and running and your `openstack` cli command should list the compute node under hosts.