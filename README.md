# SuperGrid

New base for the product formerly called SuperAtlas. Home: supergrid.com.

## Why the rename

- **Atlas** is a strong name, but it is very "AI naming". Models suggest it constantly as a brand, and hackathons are full of `Atlas()` and `()Atlas` projects.
- **Grid** fits what the thing is: the core of each part is the power plant, everything else hangs off the power grid, and it is built to be extended.

## Docs

- [SUPERGRID.md](SUPERGRID.md): product, thesis, areas to build, open questions.
- [BUSINESS-MODEL.md](BUSINESS-MODEL.md): license, pricing, monetization.
- [TENANCY.md](TENANCY.md): accounts, workspaces, sites, hosts, open-core split.

These were copied from `superatlas-app` on 2026-10-29 with the name swapped. The originals there are untouched; the hackathon build still lives in that folder.

## Folders

- `marketing/`: supergrid.com
- `operator/`: the control plane (operator software, see TENANCY.md §6)
- `workspace/`: the workspace template that is forked per workspace (TENANCY.md §8)
- `dev/`: local dev bridge and tooling
