# Sophia Org

We are building a graphical desktop environment that respects modularity, visual integrity, and user security. 

Traditional display systems are monolithic. They force display drivers, rendering, spatial layouts, input routing, and shell interfaces into a single, un-sandboxed address space. If one component falters, your entire session dies.

Sophia decomposes the modern desktop. We treat the graphics stack as a visual pipeline of specialized, single-responsibility processes communicating over clean, versioned IPC boundaries. We preserve X11’s flexible application model through isolated, virtualized namespaces while establishing strict, transaction-driven visual commits.

Our work is hosted across several focused repositories.

---

## The Visual Pipeline

```text
                  [ sandboxed clients ]
                            │
                            ▼ (classic X11 socket)
                  ┌───────────────────┐
                  │ X SERVER FRONTEND │  <-- sophia-x-authority (Rust)
                  └─────────┬─────────┘
                            │
                            ▼ (anonymous transactions)
                  ┌───────────────────┐
                  │   SOPHIA ENGINE   │  <-- sophia (Rust)
                  └──────┬─────┬──────┘
                         │     ▲
     [ sophia_wm_v1 ]    │     │      [ sophia_shell_v1 ]
    opaque layout nodes  ▼     ▼      descriptors & reservations
                  ┌───────┐   ┌───────┐
                  │ CODES │   │ CODES │  <-- decoupled policy edges
                  │  WM   │   │ SHELL │      (e.g., Hagia, Narthex)
                  └───────┘   └───────┘
```

The Sophia ecosystem is divided into mechanism, protocol translation, and policy:

1.  **Mechanism (The Engine):** Holds visual and input authority. It drives the hardware (KMS/DRM, libinput) and commits visual frames atomically. It has no understanding of window managers or application-level protocols.
2.  **Translation (The Frontend):** Resolves client protocols. It translates application commands into anonymous surface transactions for the Engine.
3.  **Policy (WM & Shell):** Legislates layout and interface. Policy processes are completely decoupled, run in their own address spaces, and operate on opaque layout nodes without seeing application metadata or titles.

---

## Core Repositories

*   **[`sophia`](https://github.com/sophia-org/sophia):** The core display server system. Includes the unopinionated `sophia-engine`, the secure Rust-based X11 protocol translator (`sophia-x-authority`), the portal broker, and the session supervisor.
*   **[`sophia-website`](https://github.com/sophia-org/sophia-website):** The source code, technical journals, and architectural specs for our home at **[sophia.gg](https://sophia.gg)**.
*   **[`hagia`](https://github.com/sophia-org/hagia):** The reference blind tiling window manager client. Written in Nim, it communicates over the metadata-blind `sophia_wm_v1` wire to proposal-test layout states.
*   **[`narthex`](https://github.com/sophia-org/narthex):** Our reference confined shell interface client. It speaks `sophia_shell_v1` to reserve work-area spans and specify securely rendered status bars and panels.

---

## Our Engineering Philosophy

We design and implement our components under a few unyielding rules:

*   **The One Rule of Confinement:** Mechanism (Engine) owns composed pixels. Policy clients (WM and Shell) decide layout, but they draw nothing, or they draw blind. Portals brokered by user consent are the only way data crosses isolated namespace boundaries.
*   **Fault Isolation:** If a policy edge (the Window Manager or Shell) crashes, the display server continues to run, holding active windows in place on the display, while the session supervisor restarts the policy client instantly.
*   **Ecosystem Pluralism:** While the unyielding visual kernel is written in Rust for memory safety, you can build window managers or shell panels in Nim, Zig, Python, Rust, or C simply by speaking our versioned IPC wire protocols.

---

## Get Involved

We invite you to explore our documentation at [sophia.gg/docs](https://sophia.gg/docs/), download the repositories, and help us refine this modular architecture.
