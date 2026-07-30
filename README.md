ARCHITECTURE:

```text
                            [ Docker Swarm Overlay Network ]

(VOTERS)                                                             (VIEWERS)
   │                                                                     │
   ▼                                                                     ▼
[ NGINX ]                                                            [ RESULT ]
   │                                                                     │
   ▼                                                                     │ 
[ VOTE ] (Replicas: 2)                                                   │ (Reads)                                                     │
   │                                                                     │
   ▼ (Writes)                                                            ▼
[ REDIS ] ◄────── (Reads) ────── [ WORKER ] ───── (Writes) ─────────► [ DB ] 
 (Queue)                         (Processor)                          (Postgres)
