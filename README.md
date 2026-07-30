ARCHITECTURE:

(VOTERS)                                                     (VIEWERS)
   │                                                            │
   ▼                                                            ▼
[ NGINX ]                                                   [ RESULT ]
   │                                                            │
   ▼                                                            │ (Reads)
[ VOTE ] (Replicas: 2)                                          │
   │                                                            │
   ▼ (Writes)                                                   ▼
[ REDIS ] ◄────── (Reads) ────── [ WORKER ] ───── (Writes) ─► [ DB ]
 (Queue)                        (Processor)                 (Postgres)
