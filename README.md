# OSP-K8S-Operators VA Automation

## Configure

1. Pick a VA from the [architecture repo](https://github.com/openstack-k8s-operators/architecture/tree/main/examples/va) and copy its various `*values.yaml`'s `.data` sections into a directory in [files](files) such that:
   - The directory has a meaningful name (referred to as `VA_KEY`
     below), e.g. `hci`
   - A sub-directory (referred to as `CM_NAME`) exists for each values file which matches the `ConfigMap` name in the [architecture automation](https://github.com/openstack-k8s-operators/architecture/blob/main/automation/vars/default.yaml) file.
   - For example the `*values.yaml` files' `.data` sections from [HCI VA](https://github.com/openstack-k8s-operators/architecture/tree/main/examples/va/hci) would be placed in [files/hci](files/hci) under the sub-directories created via the command `mkdir -p files/hci/{network-values,edpm-values,service-values,edpm-values-post-ceph}`.

2. Modify the various `files/VA_KEY/CM_NAME/{service-values.yaml,values.yaml}` files to match the configuration needs of your environment.

3. Use the [architecture automation](https://github.com/openstack-k8s-operators/architecture/blob/main/automation/vars/default.yaml) file, or create a variation of it, which has a variable structure like so to enumerate the deployment procedure for your customized version of the VA:

```
vas:
  <VA name/key that must match files/VA_KEY, i.e. hci>:
    repo: <architecture repo, i.e. https://github.com/openstack-k8s-operators/architecture.git>
    branch: <repo branch, i.e. main>
    stages:
    # stage 1
    - path: <path to stage 1 directory in arch repo, i.e. examples/va/hci/control-plane/nncp>
      validation: <CLI command to validate stage finished, i.e. oc wait nncp --for jsonpath='{.status.conditions[0].reason}'=SuccessfullyConfigured --timeout=60s">
      # values files needed for stage 1
      values:
      - name: <the name of the ConfigMap (CM_NAME) in this particular *values.yaml file in the arch repo, e.g. network-values>
        # your customized `*values.yaml` data for this particular file used in this stage of this VA
        src_file: <the name of the files/VA_KEY/CM_NAME/*values.yaml in this repo, i.e. as you modified in step 2 above>
      build_output: <the name of the file created by kustomize build, e.g. if `build_output=foo.yaml`, then `kustomize build -o foo.yaml ... ` will be run>
      # add any other entries for other *values.yaml files needed for stage 1

    # stages 2..N
    # repeat entries as per above for each additional stage
```

## Run

```
ansible-playbook -i inventory.yml -e va=hci playbook.yml
```

## Exmained Staged CRs

After running the playbook observe the `va_repo` directory. All
of the example values files will be updated based on the content
in [files](files) and staged CRs will have been created by
`kustomize`.

```
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   examples/va/hci/control-plane/nncp/values.yaml
	modified:   examples/va/hci/edpm-pre-ceph/values.yaml
	modified:   examples/va/hci/service-values.yaml
	modified:   examples/va/hci/values.yaml

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	examples/va/hci/control-plane/nncp/staged_crs.yaml
	examples/va/hci/control-plane/staged_crs.yaml
	examples/va/hci/edpm-pre-ceph/staged_crs.yaml
	examples/va/hci/staged_crs.yaml

no changes added to commit (use "git add" and/or "git commit -a")
$
```
