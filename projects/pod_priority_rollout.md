# Implement PodPriorities to reduce toil

Our software release cycle usually triggered a restart of all kubernetes Deployments and StatefulSets to pick up the latest container image version. This restart would normally result in several critical pods getting stuck in NotReady state due to insufficient resources on the worker nodes, even though there was sufficient capacity on the whole cluster. but the scheduler did not have enough context to ensure all pods would get scheduled. Resolving this would require manual operator intervention: an SRE would manually kill some non-critical pods to allow the critical pods to be scheduled. For any computer science nerds, this was an instance of the [bin-packing problem.](https://en.wikipedia.org/wiki/Bin_packing_problem).

## Task

Automate the manual steps taken by the operator to resolve the pod scheduling issue.

## Actions

I used PodPriority and Preemption policies to provide the scheduler with enough context to resolve the scheduling issues. To do this, I worked with the development team to classify each service into an appropriate priority level: high, medium or low. Services with high priority, such as user login or pods serving the web front end, could preempt lower priority services. I tested the eviction behavior to ensure it functioned as we intended.

## Result 

Reduced manual operator toil. Captured the desired application state in code.  Improved release reliability and reduced deployment delays.