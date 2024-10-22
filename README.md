# demo-kueue
## Demo of Native Kubernetes Job Queuing 

### Prerequisites
- Openshift AI >= 2.11 with Distributed Workloads enabled.

#### Admin Workflow

##### 1. Set up RBAC
As a `cluster-admin` user,  create a new project and an `admin` and 
a `kueue-batch-user-role` role binding for a user in that project.

```bash
USER=bkoz
PROJECT=kueue-demo
oc new-project $PROJECT
oc adm policy add-role-to-user admin $USER -n $PROJECT
oc adm policy add-role-to-user kueue-batch-user-role $USER -n $PROJECT
```

##### 2. Create the Kueue resources.
The `cluster-admin` creates a `clusterqueue`, `resourceflavor` and a `localqueue`.

```bash
oc create -f kueues.yaml
```

##### 3. Submit workloads.

- 10 jobs (workloads) are submitted.
- Each job requires that (3) pods run in parallel.
- Each pod requires 1 cpu and 200MB of memory.
- The ResourceFlavor cpu quota = 9
- This means only 3 jobs are permitted to run simultaneously.
- Watch kueue manage this.
- As workloads are completed (~30 seconds) new workloads are admitted.

You could also [watch this demo video](https://people.redhat.com/bkozdemb/downloads/kueue-demo-01.m4v).

```bash
for i in `seq 10`; do oc create -f job.yaml; done
```

##### 4. Observe the pending and admitted workloads.

```bash
watch oc get localqueue
```

```
NAME           CLUSTERQUEUE    PENDING WORKLOADS   ADMITTED WORKLOADS
team-a-queue   cluster-queue   7                   3
```

Wait for the workloads to complete.

##### 5. Clean up
```bash
oc delete jobs --all
oc delete pod --field-selector=status.phase==Succeeded
oc delete -f kueues.yaml
```