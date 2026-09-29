# HireBench

A competitive-programming contest platform with its own sandboxed code judge.

> **Status: work in progress.** The app is scaffolded and deployed; the features below are being built now. See the roadmap for what's done.

## What it will do

- **Contests:** admins create timed contests from a problem set (Markdown statements, time/memory limits, sample and hidden tests).
- **Judge:** participants submit C++ or Python from a browser editor (Monaco). A separate worker compiles and runs the code in a locked-down Docker container and returns a verdict: AC / WA / TLE / RE / CE.
- **Live standings:** ICPC-style, sorted by problems solved, then penalty (solve time + 20 min per wrong attempt).
- **Profiles:** GitHub sign-in, plus a verified Codeforces handle. To verify, you submit a compile error to a given CF problem, which is then checked through the Codeforces API. The profile then shows your CF rating.

## Architecture

```
Browser ──▶ Next.js app (Vercel) ──▶ Postgres (Neon)
  Monaco editor,       UI + API routes,        ▲  submissions table
  verdict polling      Auth.js, Zod            │  doubles as the job queue
                                               │
                         Judge worker (Node/TS on a Linux VPS)
                         claims a job ──▶ docker run (sandboxed) ──▶ writes verdict
```

**Key design decisions**
- **Postgres as the job queue.** Workers claim jobs with `SELECT … FOR UPDATE SKIP LOCKED`, so there's no Redis to run, and several workers can run safely side by side.
- **User code never runs on the web server.** It only runs inside the sandbox on the worker machine.
- **Sandbox limits per run:** `--network none`, memory cap with swap disabled, `--cpus 1`, `--pids-limit` (stops fork bombs), read-only filesystem plus a small tmpfs, a non-root user, a wall-clock kill timeout and an output size cap. Docker isolation is weaker than a dedicated sandbox such as `isolate` or gVisor; moving to one of those is on the roadmap.

## Stack

Next.js (App Router) · TypeScript · Tailwind · PostgreSQL (Neon) · Drizzle ORM · Auth.js · Docker · Vitest · GitHub Actions

## Roadmap

- [x] Project setup and deployment (Vercel)
- [ ] GitHub sign-in and database schema
- [ ] Problem editor and contest builder
- [ ] Judge worker with Docker sandbox
- [ ] Contest page: editor, Run/Submit, live verdicts
- [ ] Live ICPC-style standings
- [ ] Codeforces handle verification on profiles
- [ ] Worker deployed on a VPS; seeded demo contest
- [ ] Tests and CI

## Running locally

```bash
npm install
npm run dev
```
Then open http://localhost:3000.
