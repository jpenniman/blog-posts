---
layout: post
title: Rule of Modularity
date: 2025-12-04
image: /blog/images/01-rule-of-modularity.png
author: Jason M Penniman
excerpt: "Rule of Modularity: Write simple parts connected by clean interfaces."
tags:
- dotnet
- C#
- software design
- component-based design
---

> Write simple parts connected by clean interfaces.

The Rule of Modularity is a design principle that encourages us to break a system into small, self‑contained pieces—often called modules or components—and connect those pieces through well‑defined, minimal interfaces.

## Why it matters

- **Understandability** – When each part does just one thing and its responsibilities are clear, it’s easier for developers (or anyone reading the code, including AI agents) to grasp what the system does.
- **Maintainability** – Changes confined to a single module rarely ripple through the rest of the codebase, so fixing bugs, adding features, or changing behavior becomes less risky.
- **Reusability** – A cleanly isolated module can be reused in other projects or contexts because it doesn’t depend on hidden internal details of the surrounding system.
- **Testability** – Small, independent units are straightforward to unit‑test; you can verify a module’s behavior in isolation without needing the whole application running.

## What “simple parts” look like

- **Focused responsibility** – Each module should have a single, well‑scoped purpose.
- **Limited size** – Keep the amount of code, number of functions, or lines of logic modest enough that a developer can read the entire module quickly.
  Obviously, this is subjective. The idea is not to impose constraint on size, but to prevent things from growing out of control to the point where
  the code is no longer maintainable, safe to change, or easy to reason over. If the code is hard to reason over, it is a good indication it is
  getting too big and it is time to refactor.

  This helps AI coding agents too. It keeps the context small and allows the model to reason over the entire component/module.
- **Encapsulation** – Hide internal data structures and helper functions behind the module’s public surface. When implementation details leak, they cause big balls of mud.

## What “clean interfaces” mean

- **Explicit contracts** – Define clearly what inputs a module expects and what outputs it produces (e.g., function signatures, API endpoints, message formats).
- **Minimal coupling** – The interface should expose only what’s necessary; extra parameters or dependencies increase the chance that changes in one module affect another.
- **Stable versioning** – Treat the interface as a contract that other modules rely on; changes to it should be deliberate and, if possible, backward‑compatible.

## Practical tips

- **Start with a clear boundary:** Before writing code, sketch the responsibilities and the data each module will exchange.
  I like to use TDD for this. It allows me to design and get a feel for how the interface will be used.
- **Prefer composition over inheritance:** Combine simple modules rather than building deep class hierarchies that tightly bind implementations.
- **Document the interface:** Even a short comment describing expected inputs/outputs can prevent accidental misuse.
- **Refactor aggressively:** When a module starts growing beyond a manageable size, extract logical sub‑parts into new modules with their own clean interfaces.

## Conclusion

By consistently applying the Rule of Modularity, you end up with systems that are easier to reason about, evolve, and scale—whether you’re working on a tiny script or a massive distributed system.

Cheers!
