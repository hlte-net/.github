# HLTE.net System Documentation

## System Architecture Diagram

```
+---------------------+    Content     +------------------+     Email Content
|                     |  Highlighting  |                  |     ------------->
|  Browser Extension  | -------------> |                  |
|                     |                |                  |        +-------------------------+
+---------------------+                |                  |        |                         |
                                       |  Elixir Daemon   | <------+  Email (SNS/SES/S3)     |
                                       |                  |        |                         |
                                       |                  |        +-------------------------+
                                       |                  |
+---------------------+                |                  |
|                     |  Liked Posts   |                  |
|  Bluesky Account    | -------------> |                  |
|                     |                +--------+---------+
+---------------------+                         |
                                                | Content Events
                                                | (Redis Stream)
                                                v
                                     +-------------------+
                                     |                   |
                                     |  Persistence      |
                                     |  Service          |
                                     |                   |
                                     +--------+----------+
                                              |
                                              | URL Rendering
                                              | Requests
                                              v
                                     +-------------------+
                                     |                   |      +-----------------+
                                     |  Cloudflare       |      |                 |
                                     |  Worker           +----->+  R2 Storage     |
                                     |                   |      |  (PDFs/Images)  |
                                     +-------------------+      |                 |
                                                                +-----------------+

         Content Access Flow
         -------------------

 +----------+      HTTP API       +---------------+
 |          |  <-------------->   |               |
 |  Users   |                     |  Elixir       |
 |          |  <---------------   |  Daemon       |
 +----------+   Content Results   |               |
                                  +---------------+
```

## Architectural Overview

HLTE (Highlight) is a distributed system for capturing, storing, processing, and retrieving web content across multiple sources. Here's how the components work together:

### Core Components

1. **Elixir Daemon** (Primary Backend)
   - Main application server handling content processing, storage, and retrieval
   - Uses SQLite for content and tags storage
   - Provides HTTP API endpoints for content submission/retrieval
   - Publishes events to Redis for asynchronous processing
   - Supports email ingestion via AWS SNS/SES

2. **Persistence Service**
   - Watches for new entries via Redis streams
   - Stores content as JSON files for long-term archival
   - Organizes content in a structured metadata system

3. **Cloudflare Persistence Worker**
   - Captures web content as PDFs and screenshots
   - Runs on Cloudflare Workers platform
   - Stores rendered content in Cloudflare R2 storage
   - Provides API for content rendering

4. **Bluesky Likes Integration**
   - Monitors a Bluesky account for liked posts
   - Automatically captures and processes liked content
   - Submits content to the HLTE daemon
   - Uses Redis for session storage

5. **Browser Extension**
   - Allows users to highlight and capture web content
   - Provides UI for content submission and retrieval
   - Communicates with daemon API endpoints

### Data Flow

1. **Content Capture** occurs through:
   - Manual highlighting via browser extension
   - Automatic capture of Bluesky liked posts
   - Email submission of URLs

2. **Processing** handled by the Elixir daemon:
   - Content validation and normalization
   - Storage in SQLite database
   - Event publishing to Redis

3. **Persistence** managed by:
   - Persistence service storing JSON metadata
   - Cloudflare worker creating PDF/screenshot captures

4. **Retrieval** available through:
   - Browser extension UI
   - Direct API calls
   - Statistical dashboard

## Usage Information

### Setting Up the Elixir Daemon

```bash
cd elixir-daemon
mix deps.get
mix run --no-halt
```

**Configuration** (config/config.exs or environment variables):
- `HLTE_REDIS_URL`: Redis connection URL
- `HLTE_SNS_WHITELIST_JSON`: Email whitelist for SNS ingestion
- Default port: 56555

### Configuring Persistence Services

```bash
cd persistence
./run.sh
```

Requires Redis configured to match Elixir daemon settings.

### Setting Up Bluesky Integration

**Environment Variables**:
- `BLUESKY_IDENT`: Bluesky username
- `BLUESKY_PASS`: Bluesky password
- `REDIS_URL`: Redis connection URL
- `HLTE_USER_KEY`: HLTE authentication key
- `HLTE_USER_URL`: HLTE daemon URL
- `BSKYHLTE_POLLING_FREQ_SECONDS`: Polling frequency (default: 67)

### Deploying Cloudflare Worker

Configure in wrangler.toml:
- R2 bucket settings
- KV namespace for authentication tokens
- Worker environment variables

### Using the System

1. **Browser Extension**:
   - Install the extension in your browser
   - Configure with your HLTE key and daemon URL
   - Highlight text on web pages to capture content
   - Access saved content through the extension

2. **Bluesky Integration**:
   - Configure with your Bluesky account
   - Like posts on Bluesky to automatically capture them
   - Content is stored in your HLTE system

3. **Email Capture**:
   - Send emails to your configured address with URLs
   - System automatically processes and stores content
   - Receive confirmation emails

4. **API Direct Access**:
   - Submit content: POST to `/save` endpoint with content data
   - Retrieve content: GET from `/entries` with appropriate query parameters
   - All requests require authentication via key

### Security Considerations

- Authentication uses a cryptographic key system
- Key storage permissions are strictly controlled (400)
- Redis namespacing uses key hashing for isolation
- Cloudflare worker uses token-based authentication

This system provides a comprehensive content capture, storage, and retrieval solution with multiple input methods and persistence strategies, allowing for flexible personal knowledge management across various platforms.

---

This document was produced by [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview).
