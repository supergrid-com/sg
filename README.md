# SuperGrid

New base for the product formerly called SuperAtlas. Home: supergrid.com.

## Why the rename

- **Atlas** is a strong name, but it is very "AI naming". Models suggest it constantly as a brand, and hackathons are full of `Atlas()` and `()Atlas` projects.
- **Grid** fits what the thing is: the core of each part is the power plant, everything else hangs off the power grid, and it is built to be extended.

## Docs

Planning, discovery and reasoning docs live in `private/`, which is gitignored and never committed. Public docs for each part live next to the code in that part's folder.

## Folders

- `marketing/`: supergrid.com
- `operator/`: the control plane, the operator software that runs many workspaces
- `workspace/`: the workspace template, forked once per workspace and then owned by its owner
- `dev/`: local dev bridge and tooling

## Licensing

This repository is source-available. The default license is the Functional Source License, [FSL-1.1-MIT](LICENSE.md): use it, modify it, self-host it, build for clients with it. Offering it as a competing hosted product needs an operator license. Each version becomes MIT two years after release.

Directories override the default where they say so:

| Path | License |
|---|---|
| Root, `workspace/`, `operator/`, `dev/` | FSL-1.1-MIT |
| SDK and plugin API packages (each carries its own file) | MIT |
| `marketing/` code | FSL-1.1-MIT |
| `marketing/` content: brand, copy, graphics, video | All rights reserved |
| The SuperGrid name and logo | Not licensed, see [TRADEMARK.md](TRADEMARK.md) |

Signup, plans, billing, metering, the operator host and the partner console are developed outside this repository under a commercial license.
