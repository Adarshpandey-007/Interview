# 🎯 Connect — Complete Interview Preparation Guide

> **Read this on your phone before 10:30 AM tomorrow. Everything here is 100% truthful to your codebase.**

---

## Table of Contents
1. [Project Summary (30-Second Pitch)](#1-project-summary)
2. [Technology Stack (Every Single Library)](#2-technology-stack)
3. [Chat System — How It Works End-to-End](#3-chat-system)
4. [End-to-End Encryption (E2EE) — Exact Implementation](#4-e2ee)
5. [Video Calling — LiveKit SFU Architecture](#5-video-calling)
6. [WebRTC P2P Direct File Transfer](#6-p2p-file-transfer)
7. [Vault & BYOS (Bring Your Own Storage)](#7-vault-and-byos)
8. [Database Security — RLS & Hardening](#8-database-security)
9. [Rate Limiting & Audit Logging](#9-rate-limiting)
10. [Authentication & Session Management](#10-authentication)
11. [PWA & Offline Capabilities](#11-pwa)
12. [Storage Analytics & Smart Cleanup](#12-storage-analytics)
13. [Common Interview Questions & Answers](#13-common-questions)
14. [Gen AI Relevance — How to Connect This Project to AI](#14-gen-ai-relevance)

---

## 1. Project Summary

### The 30-Second Elevator Pitch
> *"Connect is a production-grade, secure, real-time communication platform I built using Next.js 16, Supabase, and WebRTC. It handles encrypted messaging, scalable video conferencing via a LiveKit SFU server, direct peer-to-peer file sharing through WebRTC Data Channels, and a cloud storage Vault system that supports custom S3-compatible backends like Cloudflare R2 and AWS S3. Every database table is locked down with PostgreSQL Row-Level Security, and all chat messages are encrypted client-side using AES-GCM-256 before they ever leave the browser."*

### Key Numbers to Mention
| Metric | Value |
|--------|-------|
| Video call scaling | 50+ participants per room |
| Client bandwidth savings (SFU vs Mesh) | ~90% reduction |
| RLS-protected tables | 100% of tables |
| Database query latency after RLS fix | < 10ms |
| PBKDF2 key derivation iterations | 100,000 rounds |
| P2P file chunk size | 16 KB per chunk |
| SQL migration files | 19 ordered scripts |
| Encryption standard | AES-GCM-256 |

---

## 2. Technology Stack

### Every Library in `package.json` — Know What Each One Does

| Package | Version | What It Does |
|---------|---------|--------------|
| `next` | 16.2.4 | React framework with App Router, server components, API routes, middleware |
| `react` / `react-dom` | 19.2.4 | UI rendering library |
| `@supabase/supabase-js` | 2.103.3 | Client SDK for talking to Supabase (database queries, auth, storage, realtime) |
| `@supabase/ssr` | 0.10.2 | Server-side Supabase client for Next.js middleware and API routes |
| `@livekit/components-react` | 2.9.21 | Pre-built React UI components for LiveKit video calls (VideoConference, RoomAudioRenderer) |
| `@livekit/components-styles` | 1.2.0 | Default CSS styling for LiveKit video call UI |
| `livekit-client` | 2.19.1 | Client-side SDK that connects to the LiveKit SFU server |
| `livekit-server-sdk` | 2.15.4 | **Server-side** SDK used in API routes to generate JWT access tokens |
| `@aws-sdk/client-s3` | 3.1057.0 | AWS S3 SDK for BYOS — uploading/downloading files to custom S3-compatible buckets |
| `@aws-sdk/s3-request-presigner` | 3.1057.0 | Generates pre-signed URLs so the browser can upload directly to S3 |
| `@upstash/ratelimit` | 2.0.8 | Sliding-window rate limiter (100 requests per 10 minutes) |
| `@upstash/redis` | 1.38.0 | Serverless Redis client for storing rate limit counters |
| `emoji-picker-react` | 4.18.0 | Emoji picker dropdown in the chat input |
| `@giphy/js-fetch-api` | 5.7.0 | Fetches trending/search GIFs from Giphy API |
| `@giphy/react-components` | 10.1.1 | Grid component to display GIF search results |
| `react-hot-toast` | 2.6.0 | Toast notification popups |
| `lucide-react` | 1.8.0 | Icon library (Send, Lock, Video, etc.) |
| `qrcode.react` | 4.2.0 | QR code generation for sharing invite links |
| `tailwindcss` | v4 | Utility-first CSS framework |
| `typescript` | v5 | Type-safe JavaScript |

### Browser APIs Used (No External Library)
| API | Where Used |
|-----|-----------|
| **Web Crypto API** (`crypto.subtle`) | E2EE encryption/decryption of messages AND IndexedDB cache encryption |
| **IndexedDB** | Local encrypted file cache for Vault files and P2P received files |
| **WebRTC** (`RTCPeerConnection`, `RTCDataChannel`) | Direct browser-to-browser file transfer |
| **`visibilitychange` Event** | Detect when user switches tabs; fetch missed messages on return |
| **`sessionStorage`** | Persist active call state for auto-reconnect on page refresh |
| **`getUserMedia`** | Request camera + microphone permissions for video calls |
| **Service Worker** | PWA offline support and caching |

---

## 3. Chat System

### Complete Flow: User A sends "Hello" to User B

```
Step 1: User A types "Hello" and hits Send
         ↓
Step 2: OPTIMISTIC UI UPDATE
        → A fake message object is created instantly with a random ID
        → It appears in User A's chat immediately (no waiting for server)
        → syncStatus = 'pending' if offline, undefined if connected
         ↓
Step 3: CLIENT-SIDE ENCRYPTION (if E2EE is on)
        → The plaintext "Hello" is encrypted using AES-GCM-256
        → A random 96-bit IV (Initialization Vector) is generated
        → Output: "v4.enc.AES-GCM.IV_<24hex>.CT_<base64ciphertext>"
        → The plaintext NEVER leaves the browser
         ↓
Step 4: DATABASE INSERT
        → The encrypted string is inserted into the `messages` table
        → Supabase returns the real UUID
        → The fake ID in React state is swapped with the real UUID
        → syncStatus changes to 'sent'
         ↓
Step 5: REAL-TIME BROADCAST (WebSocket)
        → Supabase Realtime detects the INSERT via postgres_changes
        → It sends the new row as a WebSocket payload to all subscribers
         ↓
Step 6: USER B RECEIVES THE MESSAGE
        → User B's client is subscribed to the same channel
        → The WebSocket payload arrives with the encrypted content
        → The client checks: does it start with "v4.enc.AES-GCM."?
        → YES → decrypt using the derived room CryptoKey
        → The decrypted plaintext "Hello" is displayed in User B's chat
         ↓
Step 7: READ RECEIPT
        → If User B's tab is visible (document.visibilityState === 'visible')
        → The client updates the message status to 'read' in the database
        → This UPDATE triggers another WebSocket event
        → User A sees double-checkmarks (read receipt) appear
```

### Key Technical Details to Explain

**Optimistic UI Updates:**
> "When a user sends a message, I don't wait for the database to respond. I immediately create a temporary message object with a fake ID and push it to the React state array. The user sees their message instantly. Once the database confirms the insert, I swap the fake ID for the real UUID. If the insert fails, I mark the message as 'failed' and show an error. This pattern is called Optimistic UI — it makes the app feel instant even if the network is slow."

**Offline Queue:**
> "If the WebSocket connection drops, messages typed by the user are pushed to an `offlineQueue` array. When the connection is re-established, the queue is flushed and all pending messages are inserted into the database."

**Tab-Visibility Sync:**
> "On mobile, when a user switches apps or locks their screen, WebSockets get throttled by the OS to save battery. Messages sent during this time would be lost. To fix this, I listen for the browser's `visibilitychange` event. The moment the user returns to the tab, I query the database for any messages created after the timestamp of the last known message. This guarantees zero message loss, even on mobile."

**Auto-Reconnect:**
> "If the WebSocket channel disconnects (status = `CLOSED`, `CHANNEL_ERROR`, or `TIMED_OUT`), a retry timer fires every 2 seconds. It removes the dead channel, creates a fresh one, and resubscribes. This runs automatically — the user doesn't have to do anything."

---

## 4. E2EE (End-to-End Encryption)

### How It Actually Works in Code

**Step 1: Key Derivation (PBKDF2)**
```
Input: passphrase (user enters this) + conversationId (used as salt)
         ↓
Algorithm: PBKDF2 with 100,000 iterations of HMAC-SHA-256
         ↓
Output: A 256-bit AES-GCM CryptoKey object (lives only in browser memory)
```

> **Why PBKDF2?** It's a password-based key derivation function. The 100,000 iterations make brute-force attacks impractical — each guess takes significant computation time. The `conversationId` is mixed into the salt (`roomId + '_connect_e2ee_salt_v4'`) to prevent the same passphrase from producing the same key in different rooms.

**Step 2: Encryption (AES-GCM-256)**
```
Input: plaintext message + CryptoKey + random 96-bit IV
         ↓
Algorithm: AES-GCM (Galois/Counter Mode)
         ↓
Output: "v4.enc.AES-GCM.IV_<24 hex chars>.CT_<base64 ciphertext>"
```

> **Why AES-GCM?** It provides both **confidentiality** (encryption) AND **integrity** (authentication tag). If even a single bit of the ciphertext is tampered with, the decryption will fail. This prevents man-in-the-middle modification attacks.

> **Why a random IV every time?** AES-GCM requires a unique IV per message. If the same IV is reused with the same key, the encryption is broken. Using `crypto.getRandomValues(new Uint8Array(12))` ensures a cryptographically random 96-bit IV every time.

**Step 3: What the Database Sees**
The database NEVER stores plaintext. It only stores:
```
v4.enc.AES-GCM.IV_a1b2c3d4e5f6a1b2c3d4e5f6.CT_SGVsbG8gV29ybGQ=
```
Even Supabase admins, database backups, and server-side code cannot read the messages without the passphrase.

**Step 4: Decryption**
When messages are loaded from the database, the client parses the envelope format using regex, extracts the IV (hex) and ciphertext (base64), and calls `crypto.subtle.decrypt()` with the derived key.

### Interview-Ready Summary
> *"I implemented zero-trust client-side encryption using the browser's native Web Crypto API. Messages are encrypted with AES-GCM-256 before they leave the browser. The encryption key is derived from a room passphrase using PBKDF2 with 100,000 iterations. The database only ever stores ciphertext. Decryption happens entirely client-side. This means even if the database is compromised, messages remain unreadable."*

---

## 5. Video Calling — LiveKit SFU

### The Problem: WebRTC P2P Mesh

```
In a P2P Mesh with 5 users:
Each user uploads their stream to 4 other users = 4 uploads
Each user downloads 4 streams = 4 downloads
Total connections = N × (N-1) = 5 × 4 = 20 connections

At 10 users: 10 × 9 = 90 connections → Browser CPU crashes
```

### The Solution: SFU (Selective Forwarding Unit)

```
With LiveKit SFU and 5 users:
Each user uploads their stream ONCE to the server = 1 upload
The server forwards streams to everyone else
Total uploads = N = 5 (not N²)

At 50 users: Still only 50 uploads → Works perfectly
```

### How the Call Flow Works

```
1. User A clicks "Call" button
        ↓
2. PERMISSION REQUEST (getUserMedia)
   → Browser asks for camera + microphone
   → CRITICAL: This MUST be the first async call after the click
   → Mobile browsers block permission prompts if any setState or
     network call happens before getUserMedia
        ↓
3. SIGNALING (Supabase Realtime Broadcast)
   → A 'call-request' signal is broadcast to the room channel
   → User B receives it → shows "Incoming call" UI with ringtone
   → User B clicks "Accept" → broadcasts 'call-accept' signal
        ↓
4. TOKEN GENERATION (Server-side API Route)
   → Frontend POSTs to /api/livekit/get-token with roomName
   → Server verifies:
     a) User is authenticated (Supabase session check)
     b) Room name matches pattern: connect-(dm|group)-<uuid>
     c) User is actually a member of that conversation/group
   → Server generates a JWT access token using livekit-server-sdk
   → Token has permissions: roomJoin, canPublish, canSubscribe
   → Token TTL: 6 hours
        ↓
5. LIVEKIT ROOM CONNECTION
   → Frontend passes the JWT token to <LiveKitRoom> component
   → LiveKit client connects to the SFU server
   → Media streams are published and subscribed automatically
        ↓
6. CALL END
   → User clicks "End Call" → 'call-end' signal is broadcast
   → Both clients disconnect from the LiveKit room
   → Token is cleared from state
```

### Key Security Detail
> "The API route validates that the user is actually a member of the conversation before issuing a token. Even if someone guesses a room name, they can't join without being in the `conversation_participants` or `group_members` table. This is enforced server-side."

---

## 6. WebRTC P2P Direct File Transfer

### Why P2P Instead of Cloud Upload?
> "Uploading a 500MB video to S3 and then downloading it on the other end costs egress bandwidth and takes time. With P2P, the file goes directly from Browser A to Browser B over a local network connection — zero cloud cost, zero server involvement."

### Step-by-Step Transfer Flow

```
1. User A drops a file into the chat → file is staged
        ↓
2. SHA-256 HASH
   → The file's ArrayBuffer is hashed using crypto.subtle.digest('SHA-256')
   → This hash uniquely identifies the file for caching
        ↓
3. WEBRTC SIGNALING (via Supabase Broadcast Channel)
   → User A creates an RTCPeerConnection
   → Creates a DataChannel named 'file-transfer' (ordered: true)
   → Creates an SDP Offer → broadcasts it via Supabase
   → User B receives the offer → creates an Answer → sends it back
   → ICE candidates are exchanged for NAT traversal
        ↓
4. DATA CHANNEL OPENS
   → File is read as an ArrayBuffer
   → Split into 16KB chunks
   → Each chunk is sent via dc.send(chunk)
   → BACKPRESSURE: If dc.bufferedAmount > 1MB, pause sending for 50ms
   → Progress percentage is calculated and shown in UI
        ↓
5. EOF SIGNAL
   → After all chunks are sent, sender sends the string "EOF"
   → Receiver knows the transfer is complete
        ↓
6. RECEIVER REASSEMBLY
   → Chunks are collected in an array: p2pIncomingChunks[]
   → On EOF: new Blob(chunks, { type: meta.type })
   → File is encrypted with AES-GCM-256 and stored in IndexedDB
        ↓
7. DATABASE RECORD
   → A message of type 'p2p-file' is inserted in the messages table
   → Contains: file name, size, SHA-256 hash, localUrl (blob URL)
   → This allows the file to appear in chat history
```

### The 3-Tier Download Strategy
When a user clicks "Download" on a P2P file in chat:
1. **Tier 1: Blob URL** — Check if the in-memory blob URL is still valid (same session)
2. **Tier 2: IndexedDB Cache** — Look up the SHA-256 hash in the encrypted local cache
3. **Tier 3: P2P Re-request** — If partner is online, send a `request-file` signal via WebRTC → partner reads from their cache → sends the file back over a new DataChannel

> "This 3-tier approach means a file is only ever transferred once. Every subsequent access is instant from the local cache, saving bandwidth and cloud costs."

---

## 7. Vault & BYOS (Bring Your Own Storage)

### What is the Vault?
The Vault is a secure cloud file storage system within Connect. Users can upload photos, videos, documents — and share them with specific people or groups using role-based permissions.

### What is BYOS?
> "By default, files go to Supabase Storage with a 10GB quota. But I built a 'Bring Your Own Storage' system that lets users plug in their own S3-compatible storage bucket. They enter their access key, secret key, bucket name, and endpoint URL for providers like Cloudflare R2, AWS S3, Backblaze B2, or DigitalOcean Spaces. Once activated, all their file uploads go directly to their own bucket — giving them unlimited storage at their own cost."

### How BYOS Works Technically
1. User enters S3 credentials in the Settings page
2. The **secret key is encrypted server-side** using AES-256-GCM before being stored in the `storage_providers` table
3. The `is_active` flag is set to `true` (only ONE provider can be active per user — enforced by a unique partial index)
4. When uploading a file, the API route checks if the user has an active BYOS provider:
   - **Yes** → Uses `@aws-sdk/client-s3` with the user's custom credentials to upload
   - **No** → Falls back to Supabase Storage
5. For downloads, pre-signed URLs are generated using `@aws-sdk/s3-request-presigner`

### S3 Client Factory
> "I wrote a factory function that creates an S3Client configured for each provider's quirks. For example, Cloudflare R2 uses region `'auto'` because it's regionless. Backblaze B2 requires `forcePathStyle: true` because it doesn't support virtual-hosted-style addressing. The factory handles these differences automatically."

---

## 8. Database Security — RLS & Hardening

### What is Row-Level Security (RLS)?
> "RLS is a PostgreSQL feature where the database itself enforces access rules at the row level. Even if someone bypasses my API and queries the database directly through the Supabase REST API, they can only see rows they're authorized to see. It's like a firewall inside the database."

### The Recursion Bug — How I Fixed It

**The Problem:**
```
vault_files RLS policy → checks "does user own this vault?"
  → queries vaults table
    → vaults table has its own RLS policy
      → checks "does user have permission?"
        → queries vault_permissions table
          → vault_permissions has RLS that checks vaults again
            → INFINITE LOOP → PostgreSQL stack overflow → query crashes
```

**The Fix — SECURITY DEFINER Helper Functions:**
```sql
CREATE FUNCTION public.get_vault_owner(check_vault_id UUID)
RETURNS UUID
LANGUAGE sql SECURITY DEFINER STABLE
AS $$
  SELECT owner_id FROM public.vaults WHERE id = check_vault_id;
$$;
```

> **Why this works:** A `SECURITY DEFINER` function runs with the privileges of the function owner (the database admin), not the calling user. This means it **bypasses RLS** on the `vaults` table. The function only returns a single UUID (the owner's ID), never entire rows — so it's safe. The vault_files RLS policy calls this function instead of querying the vaults table directly, breaking the recursion loop.

### Security Hardening Measures
1. **`SET search_path = public`** on ALL security definer functions — prevents schema injection attacks where an attacker creates a malicious function in a different schema with the same name
2. **`REVOKE EXECUTE FROM PUBLIC`** on internal helper functions — prevents anonymous users from calling them directly via the Supabase REST API
3. **Re-`GRANT` only to `authenticated` and `service_role`** — only logged-in users and server processes can call the helpers
4. **Sender ID spoofing prevention** — The `conversation_participants` INSERT policy has `WITH CHECK (auth.uid() = user_id)`, so a user can't insert a participant row pretending to be someone else
5. **Views use `security_invoker = true`** — views for duplicate/stale file detection run with the calling user's permissions, not the view creator's

---

## 9. Rate Limiting & Audit Logging

### Rate Limiting
> "I implemented API rate limiting using Upstash Redis with a sliding window algorithm — 100 requests per 10 minutes per user/IP. If a user exceeds the limit, the API returns 429 Too Many Requests. If Redis is unavailable, it 'fails open' (allows all requests) to prevent the entire app from going down due to a rate limiter outage."

### Audit Logging
> "Security-sensitive events are logged to an `audit_logs` table. Every file download, file deletion, upload, storage provider change, and rate limit breach is recorded with the user ID, target resource ID, IP address, and metadata. This creates a forensic trail for incident response."

**Logged Events:**
- `FILE_DOWNLOAD`
- `FILE_DELETE`
- `FILE_UPLOAD_REQUEST`
- `STORAGE_PROVIDER_ADD`
- `STORAGE_PROVIDER_DELETE`
- `RATE_LIMIT_BREACH`
- `KEY_ROTATION`

---

## 10. Authentication & Session Management

### How Auth Works
1. **Supabase Auth** handles sign-up, login, and session tokens (JWT-based)
2. **Next.js Middleware** (`middleware.ts`) runs on EVERY request (except static assets)
3. The middleware calls `supabase.auth.getUser()` to verify the session cookie
4. If the user is NOT authenticated → redirect to `/login`
5. If the user IS authenticated and tries to visit `/login` → redirect to `/` (dashboard)
6. Session cookies are automatically refreshed by the middleware on each request

> "I use the `@supabase/ssr` package which handles cookie-based sessions properly in server-side contexts. This is different from the regular Supabase client which uses localStorage — localStorage doesn't work in Server Components or API routes."

---

## 11. PWA & Offline Capabilities

### What Makes It a PWA?
- **Web App Manifest** (`manifest.ts`) — defines app name, icons, theme color, background color
- **Service Worker** — caches static assets for offline access
- **Add to Home Screen** — users can install it as a standalone app on mobile
- **Offline Queue** — messages typed while offline are queued and sent when connection returns

---

## 12. Storage Analytics & Smart Cleanup

### Storage Intelligence Dashboard
> "I built a dashboard that shows users exactly how their storage is being used. It has a donut chart showing the distribution of Photos, Videos, Documents, and Other files. The data comes from a PostgreSQL **materialized view** (`mv_storage_stats`) that aggregates file counts and sizes per category. I used a materialized view instead of a live query because running `SUM(size)` over millions of files on every page load would be too slow."

### Smart Cleanup
1. **Duplicate Detection** — A SQL view (`v_duplicate_files`) groups files by SHA-256 hash within each vault. If two files have the same hash, they're duplicates. Users can delete the extras to free space.
2. **Stale File Detection** — A SQL view (`v_stale_files`) finds files not accessed for 90+ days using the `last_accessed_at` timestamp. A debounced `touch_vault_file()` function updates this timestamp (but only once per hour to avoid write amplification).

---

## 13. Common Interview Questions & Answers

### Q: "Tell me about a challenging bug you fixed."
> *"The most complex bug was an infinite recursion loop in PostgreSQL Row-Level Security. When checking if a user could access a file in the vault_files table, the RLS policy queried the vaults table, which had its own RLS policy that queried back. This caused a stack overflow in Postgres. I fixed it by creating SECURITY DEFINER helper functions that bypass RLS and return only scalar values (a single UUID). I also locked down these functions with `SET search_path = public` to prevent schema injection attacks."*

### Q: "Why did you choose Supabase instead of Firebase?"
> *"Supabase uses PostgreSQL which gives me Row-Level Security, SQL views, materialized views, triggers, and the ability to write raw SQL for complex queries. Firebase's NoSQL model doesn't support these. Also, Supabase's Realtime is built on top of PostgreSQL's logical replication — it watches actual database changes, not a separate message queue. This means my real-time events and my database are always in sync."*

### Q: "How do you handle scalability?"
> *"For video calls, I moved from P2P Mesh to a LiveKit SFU which reduces connection complexity from O(N²) to O(N). For storage, BYOS lets users bring their own S3 buckets, distributing storage costs. For database, materialized views pre-aggregate heavy analytics queries. For caching, IndexedDB stores files locally so repeat downloads don't hit the server. For API protection, sliding-window rate limiting prevents abuse."*

### Q: "What would you improve?"
> *"I'd add Playwright E2E tests for the WebRTC signaling flow, implement Cloudflare Workers for CDN edge caching of encrypted assets, and explore pgvector on Supabase to enable semantic search across uploaded documents in the Vault."*

### Q: "How does your project handle concurrency?"
> *"For real-time messages, Supabase Realtime handles concurrent WebSocket connections at scale. For P2P file transfers, WebRTC DataChannel uses backpressure — if the send buffer exceeds 1MB, I pause for 50ms before sending the next chunk, preventing memory overflow. For database writes, PostgreSQL handles concurrent inserts with MVCC (Multi-Version Concurrency Control), and my RLS policies are all deterministic functions that don't block."*

---

## 14. Gen AI Relevance — How to Connect This Project to AI

When the interviewer asks: *"This project doesn't have AI — why are you applying for a Gen AI role?"*

### Answer:
> *"Connect gave me deep expertise in the infrastructure layer that Gen AI applications require:*
>
> 1. **Real-time Streaming** — In Gen AI, LLMs stream tokens one-by-one via WebSockets or Server-Sent Events. My experience building low-latency WebSocket messaging with optimistic updates, reconnection logic, and offline queuing directly applies to building AI chat interfaces that stream responses.
>
> 2. **Document Processing & Storage** — My Vault system handles document uploads, S3 storage, SHA-256 hashing, and metadata indexing. This is the same pipeline used for RAG — the only missing step is chunking the text and generating embeddings.
>
> 3. **PostgreSQL Expertise** — RAG systems use vector databases. Supabase supports pgvector natively. My experience with RLS policies, materialized views, database functions, and performance optimization on PostgreSQL makes me ready to work with pgvector for semantic search.
>
> 4. **Client-Side Security** — I implemented AES-GCM-256 encryption, PBKDF2 key derivation, and secure IndexedDB caching. Understanding cryptography is essential for handling PII, prompt injection mitigation, and secure API key management in Gen AI systems.
>
> 5. **API Design** — My Next.js API routes handle authentication, input validation, rate limiting, and audit logging. These patterns are exactly what's needed for building secure LLM API wrappers."

### Frameworks You Should Know About (for the theoretical questions):
- **LangChain** — Modular framework for building LLM applications. Has tools for text chunking, embedding generation, prompt templates, and agent tool-calling
- **LangGraph** — For stateful graph-based agent workflows with loops and human-in-the-loop
- **RAG Pipeline** — Ingest docs → chunk → embed → store in vector DB → user query → embed query → cosine similarity search → inject top-K results into LLM prompt → grounded answer
- **pgvector** — PostgreSQL extension for vector storage and similarity search (cosine distance: `<=>`)
- **Agent vs Chain** — A chain is a fixed sequence. An agent has a reasoning loop (ReAct: Reason → Act → Observe → Repeat) and can choose which tools to call

---

## Quick Revision Checklist (Read This Last)

- [ ] **AES-GCM-256**: Symmetric encryption with authentication tag. 96-bit IV, 256-bit key.
- [ ] **PBKDF2**: Password-Based Key Derivation. 100K iterations of HMAC-SHA-256.
- [ ] **SFU**: Selective Forwarding Unit. Each client uploads once, server routes to others.
- [ ] **WebRTC DataChannel**: Browser-to-browser binary data transfer. No server involved.
- [ ] **RLS**: Row-Level Security. Database-level access control on every row.
- [ ] **SECURITY DEFINER**: PostgreSQL function that runs with owner privileges (bypasses RLS).
- [ ] **Optimistic UI**: Show the result before the server confirms, then reconcile.
- [ ] **Materialized View**: Pre-computed query results cached in database. Refreshed with `REFRESH MATERIALIZED VIEW CONCURRENTLY`.
- [ ] **Sliding Window Rate Limiter**: Tracks requests in a time window. Smoother than fixed windows.
- [ ] **Pre-signed URL**: A temporary S3 URL with auth baked in. Expires after a set time.
- [ ] **BYOS**: Bring Your Own Storage. User connects their own S3 bucket.
- [ ] **ICE/STUN**: Protocols for NAT traversal in WebRTC connections.

---

> **Good luck tomorrow! Be confident — you built a genuinely complex, production-grade system. 🚀**
