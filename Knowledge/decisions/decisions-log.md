# Architecture Decisions Log

| Date | Project | Decision | Rationale | Status |
|---|---|---|---|---|
| 2025-04 | Oracle Proxy | nginx reverse proxy on me-jeddah-1 | ZATCA API requires Saudi IP; France VPS routed through Oracle Cloud free tier | Active |
| 2025-04 | Central POS | WPF + .NET 8 + SQLite + SignalR | Desktop-first, offline-capable, real-time sync for multi-merchant | Active |
| 2025-04 | ZATCA Portal | ASP.NET Core 6 MVC, multi-tenant, JWT | Reusable compliance layer across multiple BC clients | Active |
