# Cloud Resources

Companies increasingly store data in cloud object storage — AWS S3 buckets, Azure blobs/file storage, and Google Cloud Storage. These are frequently **misconfigured to allow public access**, making them a high-value passive enumeration target. As with domain information, much of this can be discovered without touching the company's own infrastructure.

## Useful Third-Party Providers

### domain.glass

[domain.glass](https://domain.glass/) reveals a lot about a company's infrastructure. As a useful side effect, it also surfaces security measures already in place — for example, a Cloudflare security status of "Safe" tells you a gateway-layer protection (Layer 2) exists and should be noted.

### GrayHatWarfare

[GrayHatWarfare](https://buckets.grayhatwarfare.com/) indexes publicly exposed cloud storage across AWS, Azure, and GCP. You can:

- Search and discover open buckets across all three major providers.
- Sort and filter results **by file format**.
- Passively confirm what files live in storage you found via other means (e.g. Google dorking).

> **Workflow:** Once you've identified candidate buckets (often through Google searches or leaked URLs in a company's web assets), pivot to GrayHatWarfare to passively enumerate their contents before deciding whether to access them directly.

## Why Cloud Storage Matters

| Provider | Storage Service | Notes |
| --- | --- | --- |
| AWS | S3 buckets | Most commonly misconfigured; predictable naming (`company-backups`, `company-dev`). |
| Azure | Blob & File storage | Azure **File storage uses SMB**, so it can be enumerated with familiar SMB tooling. |
| GCP | Cloud Storage | Similar exposure risks to S3. |

The combination of predictable bucket naming and lax default permissions means a company's backups, source code, configuration files, or credentials may be sitting in a world-readable bucket — discoverable entirely through passive recon.
