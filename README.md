# Bhoomi 2.0

**A proposal for a national land-information and land-trading platform that describes land as area, not perimeter.**

Bhoomi 2.0 represents land as a hierarchy of small, uniquely identified area units called **ALUs (Automated Land Units)**. Land records from multiple government sources are attached to these units, so a parcel can be looked up in one place and, in later stages, traded digitally.

> **Status: concept with working prototypes.** This repository is the overview of the idea. The code is in the sibling repositories listed below. All demo records are fictional, and nothing here is a legal title system. See [Limitations](#limitations).

Developed by Team Bace Hackers. It originated as a Smart India Hackathon (SIH) problem statement and was later extended for Kritikal Hacks 2026.

## Problem

1. **Land is described by boundaries.** Landmark- and perimeter-based descriptions are imprecise and can change over time. Land is an area, but records capture its outline.
2. **Land data is scattered.** Government survey data is not consolidated. Retrieving basic details can require visiting around seven websites, each with its own login and OTP, which pushes non-technical users toward intermediaries.
3. **Land is hard to trade.** The process is slow, paper-based and opaque.

## Core Idea: ALUs

The smallest unit, the **ALU, is defined as 0.1 m²** (roughly one floor tile). The approach is analogous to the *dx* of a Riemann sum: divide an area into the smallest unit and aggregate upward. This is an organising principle, not a new mathematical claim.

| Level | Logical cell | Area |
|---|---|---:|
| 1 | 1000 m × 1000 m | 1 km² |
| 2 | 100 m × 100 m | 10,000 m² |
| 3 | 10 m × 10 m | 100 m² |
| 4 | 1 m × 1 m | 1 m² |
| 5 (ALU) | 0.1 m × 1 m | 0.1 m² |

Every cell has a deterministic ID with parent and ancestor relationships. Properties can be attached at any level; a parent inherits uniform properties of its children. The 0.1 m² level is a **logical indexing unit**, not a claim that the underlying survey is accurate to 10 cm.

## Computing at National Scale

India covers roughly 3.29 million km², which is on the order of 33 trillion ALUs at 0.1 m² each. To avoid computing all of them, the design uses a **coarse-to-fine, multi-round** method: resolve large uniform blocks (rivers, forests) first, split only mixed cells in later rounds, and handle only boundaries at the finest level.

Design targets (not measured at national scale):

| Stage | Expected share of work settled |
|---|---|
| First round (water, forest, obvious blocks) | about 20% |
| By the 100 m level | about 50% |
| By the 10 m level | about 90% |
| Data remaining at the finest level | about 5–10% |

## AI with Human Review (Planned)

- AI assigns land features and data points, with a human supervisor. The plan is to use YOLO-family vision models.
- Each parcel is described by **21 data points drawn from 7 ministries**.
- Each AI decision carries a confidence level: green (confident), yellow (review), red (uncertain). Low-confidence cases go to a human reviewer. The working estimate is roughly 80% automated and 20% supervised.

## Product Principles

- Mostly static, pre-computed content for low serving cost.
- Lazy loading: detail loads as the user zooms in (the prototype API caps each response at 1,500 cells).
- One place instead of seven portals.
- Rural access by dial code (planned for later stages).

## Business Model (Assumptions)

- Free viewing of land records for citizens.
- Premium analysis tools for professionals (regional change over time, price history, comparisons).
- Trading as a later stage, with a nominal transaction fee. The working assumption is about 0.00018% per trade, to be validated.

**Hypothesis (unproven):** land trade is infrequent mainly because trading is difficult. The planning figure of roughly 50× more transactions is an assumption, not a measured result.

## Trust, Legal and Regulation

The principal obstacles are legal rather than technical: KYC and ownership verification (proposed through banks), a dedicated land-market regulator comparable in role to SEBI, and a phased rollout beginning with large organised participants.

## Open Research Question

How should the value of a *group* of ALUs relate to the values of the individual ALUs within it?

## Technology Stack

**Built in the prototypes**

| Layer | Technology |
|---|---|
| Marketplace frontend | React 19, TypeScript, Vite, MapLibre GL, ethers |
| WebGIS | Leaflet, OpenStreetMap basemap |
| Backend | Python, FastAPI |
| Spatial database | PostgreSQL 16 + PostGIS; GeoPandas, Shapely, PyProj |
| Evidence collection | TinyFish web agent; Fastify service that gathers and hashes the 21-field evidence sheet |
| Trade rules | Six checks: `LISTED`, `BUNDLE_INTACT`, `KYC_WALLET`, `ZONING_OK`, `FUNDS_IN_ESCROW`, `TRADE_MATCH` |
| Settlement | Solidity escrow, Hardhat, Sepolia testnet, MetaMask |
| Boundary data | Government of India BharatMapService (Survey of India) |

**Planned:** YOLO-based feature detection on satellite imagery, batch computation on cloud GPUs, static pre-computed map tiles, import of authorised state land records (e.g. BhuNaksha), a human review queue, a dial-code/voice gateway, and bank-grade KYC.

## Related Repositories

| Repository | Contents |
|---|---|
| [Kritikal_Hacks_Bace_Hackers](https://github.com/prathambansaliasofficial-ai/Kritikal_Hacks_Bace_Hackers) | Land Market prototype: React frontend, FastAPI + PostGIS backend, TinyFish middleware, testnet escrow |
| [Kritikal-Hacks-26 (WebGIS)](https://github.com/yashbaghel03-debug/Kritikal-Hacks-26-Bace-Hackers-Re-imagining-how-land-use-to-Trade) | WebGIS and ALU grid on Indian boundary data; 21-category data pipeline |

## Roadmap

- [x] ALU hierarchy and zoom-dependent grid
- [x] Land-market prototype with rule engine and testnet escrow
- [x] Evidence collection and review workflow (prototype)
- [ ] Real cadastral parcel geometry from authorised sources
- [ ] AI feature detection and confidence scoring at scale
- [ ] Pre-computed tiles and public deployment
- [ ] Real KYC and ownership verification
- [ ] Rural dial-code access
- [ ] Pilot with institutional users

## Limitations

- Demo parcels, owners and evidence are fictional.
- Escrow runs on a testnet only; a blockchain receipt is not a legal transfer of title.
- KYC in the prototype is a stub.
- The efficiency figures, fee level and "50×" idea are estimates and hypotheses.
- Real cadastral polygons are state-specific and must come from authorised sources.

## Team

Team Bace Hackers: Pratham Bansal ([@prathambansaliasofficial-ai](https://github.com/prathambansaliasofficial-ai)) and Yash Baghel ([@yashbaghel03-debug](https://github.com/yashbaghel03-debug)).

[Add: your specific contribution.]
