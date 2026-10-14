# Sophia

Sophia is an experimental desktop for Linux, built on an idea from Plan 9: every client gets its own view of the system, and the server checks every step it takes. Engine owns the input devices and the display. The window manager and the shell run as separate processes, so a crash in either one doesn't take your session down.

X11 is a full frontend, with an X server written from scratch in Rust. Everything else talks to Sophia over one shared 9P2000.L core. Each role keeps its own endpoint, file tree and permissions; sharing the code doesn't mean sharing the permissions.

## The shape

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

Engine commits each window's geometry and pixels together. The window manager proposes layout and focus from anonymous nodes and never sees titles, process IDs or pixels. Shells draw panels, launchers and switchers through their own interfaces, and they get window metadata only with separate permission.

Administration and the metadata broker still use the old socket protocol. The 9P application frontend and the shared contract between frontends and Engine are planned, not built; the drawing doesn't show them as finished.

## What Sophia borrows from Plan 9

A namespace is everything a connection can reach. The session builds each client's socket directory and discovery file from trusted policy, and every endpoint serves its own tree and checks what the client may do. A client proves its identity when it connects, and that identity is fixed for the connection. Knowing a path, an attach name or holding an open file never grants permission by itself. X11 keeps its own admission checks.

The roles share file operations, not a record format. Small commands and discovery use lines of text; window-manager and shell traffic keeps compact binary records. Clients speak 9P directly. Kernel mounts aren't a security boundary, and Sophia doesn't use Plan 9's draw protocol.

Portals are the only way between namespaces. The portal service decides whether a transfer may happen, and the receiving frontend carries it out. A 9P client gets one object bound into its tree, and revoking the grant makes its open files stale. An X client gets ordinary X semantics. File descriptors travel on a separate channel, never inside the 9P stream.

Screen capture is the first portal: a frame record and the screenshot bytes for one named output. It works on both physical displays today, and locking the screen or changing a display revokes old grants on time. Mirrored displays are still being tested. A capture grant never lets anyone inject input. Agents use the same interfaces as every other client; Sophia gives them no special powers.

Wayland is possible but off the roadmap. Someone could write a compatibility layer as a client of the planned 9P application frontend, once that frontend exists.

## Repositories

- **[sophia](https://github.com/sophia-org/sophia):** Engine, the Rust X server, the shared 9P core, portals and session supervision. The [roadmap](https://github.com/sophia-org/sophia/blob/master/docs/strategic-roadmap.md) tracks the remaining work.
- **[sophia-website](https://github.com/sophia-org/sophia-website):** The source for [sophia.gg](https://sophia.gg), its documentation and blog.
- **[hagia](https://github.com/sophia-org/hagia):** The reference window manager, a tiling manager written in Nim that never sees what it's tiling. It uses Sophia's window-manager files over 9P.
- **[narthex](https://github.com/sophia-org/narthex):** A reference shell that describes panels for Engine to draw and reserves screen space through Sophia's shell files.

## Build on Sophia

Window managers and shells run outside Engine, so you can write them in any language without taking over composition or input. If a supervised window manager or shell exits, Engine keeps the last frame on screen while the session restarts it.

Start with [Building on Sophia](https://sophia.gg/docs/building-on-sophia/) and the [architecture map](https://sophia.gg/docs/architecture/). The [one-core decision](https://github.com/sophia-org/sophia/blob/master/docs/notes/decisions/zsx0tk4k-serve-every-public-role-from-one-9p-core-with-namespaces-as-composed-trees-and-portals-as-binds.md) explains where Sophia is headed; what's built today is a separate question, answered on the site's status section.
