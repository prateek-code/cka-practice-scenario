# 1. Kubernetes Deployment Rollout History Command

The `kubectl rollout history (TYPE NAME | TYPE/NAME) [flags] [options]` command is used to view the history of deployments and their revisions.

## Examples

  ### View the rollout history of a deployment
  `kubectl rollout history deployment <deployment-name>`

  ### View the details of daemonset revision 3
  `kubectl rollout history daemonset <daemonset-name> --revision=3`

# 2. Kubernetes Deployment Status Command

The `kubectl rollout status (TYPE NAME | TYPE/NAME) [flags] [options]` command is used to view the status of deployments and their revisions.

## Examples

  ### View the rollout status of a deployment
  `kubectl rollout status deployment <deployment-name>`

  ### View the details of daemonset revision 3
  `kubectl rollout status daemonset <daemonset-name> --revision=3`
