# Sophia

Sophia is an experimental desktop system shaped by Plan 9: a shared file protocol, a namespace composed for each identity, and explicit grants between namespaces. Engine owns composition and physical input. Separate processes handle window layout and shell UI.

X11 remains a first-class frontend, with a native authority written from scratch in Rust. Sophia's native interfaces use a shared 9P2000.L core. Each role keeps its own endpoint, file tree and authority; sharing the implementation does not merge their permissions.

## The Shape

```text
  X11 clients              9P clients: WM · shells · output
        │                  lock · factotum · inspection · portals
        ▼                              │
  ┌────────────────┐          ┌────────────────────┐
  │ X AUTHORITY    │          │ SHARED 9P CORE     │
  │ Native Rust    │          │ 9P2000.L           │
  │ Built from     │          │ Separate exports   │
  │ scratch        │          │ and endpoints      │
  └───────┬────────┘          └────────┬───────────┘
          │     admitted transactions, │
          │     proposals and commands │
          └──────────────┬─────────────┘
                         ▼
                  SESSION OWNERS
              admission · routing · policy
                         │
                         ▼
                   SOPHIA ENGINE
        input · scene · commits · rendering · scanout
                         │
                         ▼
                     DRM / KMS
```

Engine commits window geometry and pixels together. The window manager proposes layout and focus from opaque nodes; it receives no titles, process IDs or pixels. Shells draw panels, launchers and switchers through their own admitted interfaces, with separately authorized metadata.

Administration and the metadata broker still await migration from the legacy socket envelope. The 9P application authority and a common protocol-authority contract are also planned; they are not shown as completed paths above.

## What Sophia Borrows from Plan 9

A namespace describes what an admitted connection can reach. Session composes its socket directory and discovery file from trusted policy. Each endpoint serves its own tree and checks the client's authority. Authenticated attach fixes identity for the admitted launch paths; a path, attach name or open handle cannot create permission. X11 retains its own admission checks.

Roles share file operations, not one record format. Small controls and discovery can use text; window-manager and shell traffic keeps compact binary records. Core clients connect directly over 9P. Kernel mounts are not an authority mechanism, and Sophia does not adopt Plan 9's draw protocol.

Portals authorize transfers between namespaces. The decision owner checks bounded facts; the recipient's frontend executes the transfer. A 9P recipient receives a grant-bound object in its tree. Revocation makes retained handles stale. X recipients use X semantics. Descriptors belong on a separate channel, never inside the 9P stream.

Capture is the first output bind: a frame record and bounded screenshot bytes, authorized for one named output. It is implemented but remains under physical and performance qualification. Capture grants imply no permission to inject input. Agents use the same admitted interfaces as other clients; Sophia adds no agent-specific authority.

Wayland is possible but off the roadmap. A third party could build a compatibility layer as a client of the planned 9P application authority, carrying its clients as separate identities. That extension path depends on the application authority landing.

## Core Repositories

- **[sophia](https://github.com/sophia-org/sophia):** Engine, the Rust X11 authority, the shared 9P core, portal services and Session supervision. The [roadmap](https://github.com/sophia-org/sophia/blob/master/docs/strategic-roadmap.md) tracks the remaining work.
- **[sophia-website](https://github.com/sophia-org/sophia-website):** The source for [sophia.gg](https://sophia.gg), its documentation and development posts.
- **[hagia](https://github.com/sophia-org/hagia):** The reference blind tiling window manager, written in Nim and using Sophia's WM files over 9P.
- **[narthex](https://github.com/sophia-org/narthex):** A reference descriptor shell, using Sophia's shell files to describe panels and reserve work areas.

## Build on Sophia

Window managers and shells run outside Engine. Their protocols let clients use languages other than Rust without taking over composition or physical input. If a supervised WM or shell exits, Engine retains the committed scene while Session handles recovery.

Start with [Building on Sophia](https://sophia.gg/docs/) and the [architecture](https://sophia.gg/docs/architecture/). The [one-core decision](https://github.com/sophia-org/sophia/blob/master/docs/notes/decisions/zsx0tk4k-serve-every-public-role-from-one-9p-core-with-namespaces-as-composed-trees-and-portals-as-binds.md) explains the direction; current implementation and acceptance remain separate from that target.
