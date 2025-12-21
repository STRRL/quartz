# Re: Go Ahead Self host PostgresSQL

Origin: <https://pierce.dev/notes/go-ahead-self-host-postgres#user-content-fn-6>

## Overview

I am developing [supabase-operator](https://github.com/strrl/supabase-operator), there some best practice mentioned in this post, and could be integrated in the supabase-operator project

## Take Away

Setup pgbouncer pls, the writer also propose a auto config program: <https://github.com/piercefreeman/autopg/tree/main/autopgpool>

Also another awesome project, pgtune: <https://pgtune.leopard.in.ua/>, <https://github.com/le0pard/pgtune>

### Routine Tasks

```
- Weekly tasks (10 minutes):
  - Check backup verification (automated, just reviewing alerts)
  - Review slow query logs
  - Check disk space trends
- Monthly tasks (30 minutes):
  - Apply Postgres security updates
  - Review and rotate backup retention
  - Capacity planning based on growth trends
- Quarterly tasks (optional) (2 hours):
  - Update monitoring dashboards
  - Review and optimize configuration parameters
  - Test disaster recovery procedures
```

### Configs

Memory:

```
The key parameters:

shared_buffers: Start around 25 % of RAM; modern PG happily uses tens of GB.
effective_cache_size: Set to 75% of system RAM (this tells Postgres how much memory the OS will use for caching)
work_mem: Be conservative here. Set it to total RAM / max_connections / 2, or use a fixed value like 32MB
maintenance_work_mem: Can be generous (1-2GB), only used during VACUUM and index operations
```

Connection:

```
# Connection settings
max_connections = 200
shared_preload_libraries = 'pg_stat_statements'
log_connections = on
log_disconnections = on
```

Storage:

```
# Storage optimization for NVMe
random_page_cost = 1.1                 # Down from default 4.0
seq_page_cost = 1.0                    # Keep at default
effective_io_concurrency = 200         # Up from default 1

# WAL settings
wal_level = replica                     # Enable streaming replication
max_wal_size = 2GB                     # Allow larger checkpoints
min_wal_size = 1GB                     # Prevent excessive recycling
checkpoint_completion_target = 0.9      # Spread checkpoint I/O over 90% of interval

```

