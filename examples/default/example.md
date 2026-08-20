<img src="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI2NCIgaGVpZ2h0PSI2NCI+PHJlY3Qgd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0iIzBkM2I2NiIvPjx0ZXh0IHg9IjMyIiB5PSIzOCIgZm9udC1mYW1pbHk9IkFyaWFsIiBmb250LXNpemU9IjIyIiBmaWxsPSIjZmFmYmZjIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIj5MUzwvdGV4dD48L3N2Zz4=" class="logo-top-right" alt="Company Logo Placeholder" />

# Lumina Stream Architecture

This document provides a technical overview of the Lumina Stream application, detailing the configuration steps, API design, and schema definitions.

## 1. System Requirements

Before initializing the local environment, you must ensure that your system meets the baseline configuration standards. 

### 1.1 Dependencies

The following internal tools must be present on your host machine:
* Python 3.10+
* Docker Engine 24.0.0 or higher
* Valid TLS certificates in your local keystore

> #### Security Notice
> Never commit your `.env` file to version control. All API keys and secrets should be injected at runtime via your CI/CD pipeline variables.

## 2. Network Configuration

The service routes all incoming traffic through a centralized gateway. You can define specific overrides in your `routing.conf` file using standard proxy directives.

### 2.1 Example Routing Block

To bypass the standard auth middleware for public assets, append the following rule. Notice the use of the `Cache-Control` header.

<div class="codehilite">
<pre><code>location /public/ {
    proxy_pass http://cdn_backend;
    proxy_set_header Host $host;
    add_header Cache-Control "public, max-age=31536000";
}</code></pre>
</div>

## 3. Data Schema Mappings

When interacting with the ingestion endpoints, payloads are automatically serialized and typed against the database schema.

| JSON Primitive | Database Column | Nullable | Index Type |
|---|---|---|---|
| `uuid` | `UUID PRIMARY KEY` | False | B-Tree |
| `timestamp` | `TIMESTAMP WITH TIME ZONE` | False | Brin |
| `metadata` | `JSONB` | True | GIN |
| `is_active` | `BOOLEAN` | False | None |

### 3.1 Field Validations

All text fields are sanitized upon entry. If a payload violates the schema, the API will respond with an HTTP `400 Bad Request` and return a standard error trace.
