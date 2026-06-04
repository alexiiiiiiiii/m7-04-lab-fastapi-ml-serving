# Architectural Decisions

## 1. Versioning
We selected path-based versioning (`/v1/`) over header-based versioning to maximize developer ergonomics and guarantee seamless routing by API gateways and reverse proxies. Path-based versioning is highly visible in logs and simplifies client configuration across diverse integration tools like Postman, curl, and native SDKs without requiring custom header handling.

## 2. Batch Ordering and Partial Failures
When `predict-batch` is called with 32 items and one image is corrupt, the API returns a `200 OK` status with a keyed JSON response object preserving the user-supplied identifiers. Instead of failing the entire batch, the entry corresponding to the corrupted image contains a detailed structured `Error` object, while the successful classifications contain their respective `PredictResponse` data. This prevents partial inputs from blocking the entire pipeline and avoids relying on fragile array index alignment for mapping inputs to outputs.

## 3. Async Lifecycle
An asynchronous job transitions through the lifecycle states of `queued` → `running` → `completed` or `failed`. Clients retrieve results from the `GET /v1/predictions/{job_id}` endpoint, which returns a `202 Accepted` while the job is `queued` or `running`, and a `200 OK` with the schema once the job is in a terminal state. Completed and failed job results are retained in high-performance storage for 24 hours to balance operational debuggability with storage cost constraints, after which they are permanently deleted.
