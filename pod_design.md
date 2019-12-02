# POD DESIGN


## Labels and Selectors

### Listing and filtering pods

#### `--show-labels` displays all labels under a LABELS column
```
kubectl get po --show-labels
```

#### `--selector, -l` filter results by labels in key:value format
```
kubectl get po --selector app=App1
kubectl get po -l app=App1
```

#### `--label-columns, -L` displays labels and its values in a separate column
```
kubectl get po --label-columns=app,release
kubectl get po -L app,release
```

#### `--field-selector` filters results by fields using comparison operators
```
kubectl get po --field-selector=status.phase!=Running,spec.restartPolicy=Always
```

### Labeling Pods

#### `--overwrite` overwrites if label already exists on the resource
```
kubectl label po nginx2 app=App2 --overwrite
```

#### `-` appended after the label key removes the label from the resource
```
kubectl label po <pod1-name> <pod2-name> <pod3-name> app-
```
> Note: `--overwrite` is not required

#### `--resource-version` Update pod 'foo' only if the resource is unchanged from version 1
```
kubectl label po foo status=unhealthy --resource-version=1
```


## Annotations

### Annotating Pods

#### `--overwrite` overwrites if annotation already exists on the resource
```
kubectl annotate po <pod1-name> <pod2-name> description='my description' --overwrite
```

#### `-` appended after the annotation key removes the annotation from the resource
```
kubectl annotate po <pod1-name> <pod2-name> description-
```

#### `--resource-version` Update pod 'foo' only if the resource is unchanged from version 1
```
kubectl annotate po foo description='my frontend running nginx' --resource-version=1
```


## Rolling Updates and Deployment Rollbacks

### Strategies

#### Recreate Strategy

Recreate deletes the resources of the current deployment before deploying the updated resources resulting to service downtime.

#### Rolling Updates Strategy

Rolling Updates deletes ands updates the current deployment's resources one by one so there is no service downtime.

### Handling Rollouts

#### creating and updating deployments
```
kubectl create -f deployment-definition.yaml
kubectl apply -f deployment-definition.yml
kubectl set image deploy/myapp-deployment nginx=nginx:1.9.1
```
> Note: refer to [Deployment Command Line Reference](./core_concepts.md#deployment-command-line-reference-sheet) for details.

#### `status` to check the rollout status
```
kubectl rollout status deployment/myapp-deployment
```

#### `history` to check the rollout history
```
kubectl rollout history deployment/myapp-deployment
```

#### `history --revision` to view history details, including podTemplate of the revision specified of a specific revision
```
kubectl rollout history deployment/myapp-deployment --revision=3
```

#### `pause` and `resume` ongoing rollouts
```
kubectl rollout pause deploy nginx
kubectl rollout resume deploy nginx
```

#### `undo` to rollback (`--to-revision` to specify to a specific revision)
```
kubectl rollout undo deployment/myapp-deployment --to-revision=2
```

## Jobs and Cronjobs

### Restart Policies

- Always, 
- onFailure, 
- Never, 

```
apiVersion: v1
kind: Pod
metadata:
  name: math-pod
spec:
  containers:
  - name: math-add
    image: ubuntu
    command: ['expr', '3', '+', '2']
  restartPolicy: Never
```
