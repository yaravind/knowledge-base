# Pipeline Execution

What does “Jobs access OneLake using Fabric identity propagation, not storage keys” mean?
In Microsoft Fabric, Spark jobs never authenticate to OneLake using storage account keys, SAS tokens, or user‑managed secrets.

Instead, Fabric automatically propagates identity and permissions from the user, service principal, or pipeline that started the job, all the way down to OneLake.

Step‑by‑step explanation
A user or pipeline starts a Notebook or SJD

This could be:
A human user (Entra ID identity)
A pipeline using a service principal or managed identity
Fabric captures the caller’s identity

Fabric knows who initiated the workload
This identity is validated via Microsoft Entra ID
Fabric propagates that identity into the Spark session

When the Spark session is created:
The driver node runs under a Fabric‑managed identity context
Worker nodes inherit the same security context
Spark accesses OneLake using Fabric‑managed authentication

Fabric translates identity permissions into secure, short‑lived access
Spark executors can read/write OneLake without secrets
No connection strings or keys appear in code or configs
OneLake enforces authorization

Access is governed by:
Workspace and item permissions
Lakehouse / Warehouse security
Folder‑level ACLs (where applicable)
If the caller doesn’t have permission, the Spark job fails — even though the compute exists.

What does NOT happen (important)
✅ No storage account keys

✅ No SAS tokens in notebooks

✅ No secrets in Key Vault

✅ No manual token handling in Spark configs

This is fundamentally different from:

Self‑managed Spark on ADLS Gen2
Databricks with storage keys or OAuth configs
Why this matters
Security

Eliminates long‑lived secrets
Reduces risk of credential leakage
Governance

Centralized permission management
Auditable access through Fabric and Purview
Developer experience

No authentication code
No environment‑specific secrets
Same code works in dev, test, and prod
Simple analogy (for training)
Think of Fabric like a secured office building:

You scan your badge at the entrance (Notebook or Pipeline)
Fabric escorts you through the building (Spark session)
Every door you open (OneLake access) checks your badge, not a shared master key
One‑line summary (for slides)
Fabric uses identity‑based access to OneLake, automatically enforcing the caller’s permissions — no storage keys required.