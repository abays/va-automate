# OSP-K8S-Operators VA Automation

## Configure

1. Pick a VA from the [architecture repo](https://github.com/openstack-k8s-operators/architecture/tree/main/examples/va)
and copy its various `*values.yaml`'s `.data` sections into a directory in [files](files) and give that directory
a meaningful name (referred to as `VA_KEY` below).  For example, take the `*values.yaml` files' `.data` sections from 
[HCI VA](https://github.com/openstack-k8s-operators/architecture/tree/main/examples/va/hci)
and place them in [files/hci](files/hci).
2. Name the files you create in [files](files) however you wish, but it is advised to pick names that correspond
to the "stage" of the VA to which those values pertain.  For example, if you take a networking `values.yaml` from
[HCI VA](https://github.com/openstack-k8s-operators/architecture/tree/main/examples/va/hci), then you might call the
file `network-values.yaml`.
3. Modify the various `files/VA_KEY/*values.yaml` files to match the configuration needs of your environment.
4. In [vars/default.yaml](vars/default.yaml), create a variable structure like so to enumerate the deployment
procedure for your customized version of the VA:
```
vas:
  <VA name/key that must match files/VA_KEY, i.e. hci>:
    repo: <architecture repo, i.e. https://github.com/openstack-k8s-operators/architecture.git>
    branch: <repo branch, i.e. main>
    stages:
    # stage 1
    - path: <path to stage 1 directory in arch repo, i.e. examples/va/hci/control-plane/nncp>
      validation: <CLI command to validate stage finished, i.e. oc wait nncp --for jsonpath='{.status.conditions[0].reason}'=SuccessfullyConfigured --timeout=60">
      # values files needed for stage 1
      values:
      - name: <the name of the ConfigMap in this particular *values.yaml file in the arch repo, i.e. network-values>
        # your customized `*values.yaml` data for this particular file used in this stage of this VA
        src_file: <the name of the files/VA_KEY/*values.yaml in this repo, i.e. as you chose in step 2 above>
        # file to write out to <path>/<dest_file> after processing <src_file> for use with the VA kustomize automation
        dest_file: <the original file name of the *values.yaml file for this stage in the arch repo, i.e. values.yaml>
      # add any other entries for other *values.yaml files needed for stage 1

    # stages 2..N
    # repeat entries as per above for each additional stage
```

## Run

```
ansible-playbook -i inventory.yml -e va=hci playbook.yml
```
