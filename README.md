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

OBESERVATIONS:

1. The Rolling Update
Action: Injected new environment variables to change the voting options from "Cats vs. Dogs" to "android vs. iphone".
Observation: Swarm did not take the application offline. Based on the update_config in the Compose file, it methodically shut down the old replicas and spun up the new replicas one by one. The update was verified via an incognito browser, proving zero-downtime deployment.

3. The Rollback
Action: Executed the --rollback command on the vote service.
Observation: Swarm instantly recognized the previous state of the service and rolled back the tasks using the exact same rolling-update methodology. The application gracefully reverted to its previous configuration without human intervention in the YAML file.

5. Node Failure
Action: Changed the availability of worker_1 (which hosted the pinned database) to drain.
Observation:
•	Swarm immediately detected the node loss and successfully evacuated stateless containers (like the vote app and worker), rescheduling them onto healthy nodes in seconds.
•	Critical Finding: Because the db service was strictly constrained to worker_1 via the label db=true, Swarm could not reschedule it. The database remained in a pending/offline state until worker_1 was brought back online. This proves that strict placement constraints override Swarm's default high-availability rescheduling behavior.



