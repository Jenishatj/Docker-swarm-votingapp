ARCHITECTURE:

```text
                            [ Docker Swarm Overlay Network ]

(VOTERS)                                                             (VIEWERS)
   │                                                                     │
   ▼                                                                     ▼
[ NGINX ]                                                            [ RESULT ]
   │                                                                     │
   ▼                                                                     │ 
[ VOTE ] (Replicas: 2)                                                   │ (Reads)
   │                                                                     │
   ▼ (Writes)                                                            ▼
[ REDIS ] ◄────── (Reads) ────── [ WORKER ] ───── (Writes) ─────────► [ DB ] 
 (Queue)                         (Processor)                          (Postgres)
`````

CONCEPTUAL QUESTIONS:

1.	What is the difference between a master and a worker?

Master maintains the cluster state, handle routing, and dispatch tasks to other nodes. Workers strictly execute the tasks (containers) assigned to them by the managers and report back on their status; they have no authority over the cluster itself.

2. What are the two join tokens for and why are they different?
   
* 	Worker Token: Grants a node permission to join strictly as a worker, meaning it can only receive and execute container tasks assigned to it by the manager.
* 	Manager Token: Grants a node full administrative rights, allowing it to vote on cluster state, modify configurations, route traffic, and schedule tasks.
The reason they are different is to uphold the security principle of least privilege. If an attacker compromises a worker node and steals its join token, they can only use it to add more worker nodes to the cluster. They cannot use that token to a manager status which protects the cluster’s core.

3. What happens to the cluster if the manager node dies?
   
* Single-Manager Cluster: If our only manager dies, the "control plane" of the cluster goes down. We can no longer deploy new services, update existing ones, or add/remove nodes. However, the "data plane" remains — existing containers running on your worker nodes will continue to run normally. The workers just will not receive any new instructions until a manager is restored.
* Multi-Manager Cluster: If we have multiple managers (e.g., 3 or 5), the cluster will automatically elect a new leader and continue functioning with zero interruptions, provided the remaining managers still hold "quorum."

4. What is Quorum?

Quorum is the strict majority of manager nodes that must be online and communicating with each other to make decisions or change the cluster's state.


OBSERVATIONS:

1. The Rolling Update

* Action: Injected new environment variables to change the voting options from "Cats vs. Dogs" to "android vs. iphone".
* Observation: Swarm did not take the application offline. Based on the update_config in the Compose file, it methodically shut down the old replicas and spun up the new replicas one by one. The update was verified via an incognito browser, proving zero-downtime deployment.

2. The Rollback

* Action: Executed the --rollback command on the vote service.
* Observation: Swarm instantly recognized the previous state of the service and rolled back the tasks using the exact same rolling-update methodology. The application gracefully reverted to its previous configuration without human intervention in the YAML file.

3. Node Failure

* Action: Changed the availability of worker_1 (which hosted the pinned database) to drain.
* Observation:
* Swarm immediately detected the node loss and successfully evacuated stateless containers (like the vote app and worker), rescheduling them onto healthy nodes in seconds.
* Critical Finding: Because the db service was strictly constrained to worker_1 via the label db=true, Swarm could not reschedule it. The database remained in a pending/offline state until worker_1 was brought back online. This proves that strict placement constraints override Swarm's default high-availability rescheduling behavior.



