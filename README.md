# Bhoomi 2.0

**Land is area, not perimeter.**

Bhoomi 2.0 is a proposal for a national land-information and land-trading platform. Instead of describing land by fuzzy boundary lines, it describes land as a hierarchy of tiny, uniquely identified area units called **ALUs (Automated Land Units)**. Land records from many government sources are attached to those units, so anyone can look up a piece of land in one place and, eventually, trade it in seconds.

> **Status: vision plus working prototypes.** This repository is the overview of the idea. The running code lives in two sibling repositories (see [Repositories](#repositories)). All demo records are fictional, and nothing here is a legal title system. See [Honest limits](#honest-limits).

Built by team **Bace Hackers**. Originally a Smart India Hackathon (SIH) problem statement, later extended for Kritikal Hacks 2026.

---

## The problem

1. **Land is described by boundaries, not by area.** Traditionally a plot is described by landmarks, such as "20 acres between this tree and the river bank". Landmarks move, weather changes them, and nobody can say exactly where the land is. GPS coordinates helped, but they still describe the *perimeter*. Land is really an *area*, and we keep trying to capture it by its outline.
2. **Land data exists but is scattered.** Governments have spent enormous sums on land surveys, but the data is not formally structured. Finding basic details means visiting around seven different websites, each with its own login and its own OTP. For a farmer or any non-technical person this is very hard, so people fall back on local revenue officials (patwaris) and intermediaries, who often cannot help ("the server is down").
3. **Land is hard to trade.** The process is slow, paper-heavy and opaque, which leaves room for corruption and keeps many owners and buyers out of the market.

## The core idea: ALUs

We define the **smallest unit of area, the ALU, as 0.1 m²**, roughly the size of one floor tile. This plays the same role as *dx* in a **Riemann sum**: divide an area into the smallest unit, then add everything up. We borrow the idea of Riemann integration as an organising principle; we are not claiming a new theorem.

ALUs are organised in a hierarchy, and every cell has a deterministic ID with parent and ancestor relationships:

| Level | Logical cell | Area |
|---|---|---:|
| 1 | 1000 m x 1000 m | 1 km² |
| 2 | 100 m x 100 m | 10,000 m² |
| 3 | 10 m x 10 m | 100 m² |
| 4 | 1 m x 1 m | 1 m² |
| 5 (ALU) | 0.1 m x 1 m | 0.1 m² |

Key properties:

- **Properties live at any level.** A parent cell automatically inherits the properties of its children when they are uniform (for example, a 1 km block that is entirely river).
- **Small, acceptable error.** At the finest level, the possible error is about one tile of land. Informal surveys we ran suggested that people are comfortable with this margin.
- **Grouping creates new value.** A single ALU is unique, but a group of ALUs forms a solid chunk that can have a different trade value from the sum of its parts.

> The 0.1 m² level is a **logical indexing unit**. It is not a claim that the underlying cadastral survey is accurate to 10 cm.

## Computing at national scale

India has roughly 3.29 million km² of area. At 0.1 m² per ALU that is on the order of **33 trillion ALUs**. Computing every one of them naively would be a tremendous amount of work and would need huge server capacity.

Our method avoids this with a **coarse-to-fine, multi-round** process:

1. **Round 1: large blocks (1 km² to 100 km²).** Capture whole areas and settle the easy ones first, such as rivers and forests.
2. **Later rounds: smaller cells.** Cells already resolved are *not* processed again. Only mixed cells, where several things meet, are split further.
3. **Final rounds: boundaries only.** At the finest level we only handle the boundaries between regions, which is a tiny fraction of the total area.

Our working estimates (design targets, not yet measured at national scale):

| Stage | Share of the work expected to be settled |
|---|---|
| First round (water, forest, obvious blocks) | about 20% |
| By the 100 m level | about 50% |
| By the 10 m level | about 90%, after computing only millions of cells |
| Data left to store at the finest level | about 5 to 10% |

The result is that computing cost drops by orders of magnitude compared with filling a 33-trillion-cell grid.

## AI with a human in the loop

- **AI does the bulk of the assignment.** It identifies land features and assigns data points to land. The plan is YOLO-family vision models for feature detection, with the process governed by a human supervisor.
- **21 data points across 7 ministries.** Each parcel is described by a fixed set of 21 fields gathered from seven government sources.
- **Confidence scores.** Every AI decision carries a confidence level: **green** (sure), **yellow** (check), **red** (not sure). Low-confidence cases go to a human.
- **Roughly 80% automated, 20% supervised**, which reduces manual effort to about a fifth of a fully manual process.
- **AI is used once.** The heavy AI work is a one-time computation (a capital cost). After that, operating the system mainly means serving a largely static website.

## Product principles

- **Mostly static by design.** Around 80% of what users see is static or pre-computed, so it is cheap and fast to serve. Dynamic elements exist but are the minority.
- **Lazy loading.** Only what is on screen is loaded, and more detail loads as the user zooms in. The browser never builds a whole-country grid (the prototype API caps each response at 1,500 cells).
- **One place instead of seven portals.** No repeated logins or OTPs just to read land details.
- **Rural access by phone (planned).** In later stages, villagers could see their land details by dialling a short code, with no smartphone or internet needed.

## Business model

- **Free for citizens.** Viewing land records is free. This is the freemium base.
- **Premium tools for professionals.** Analysis tools for real-estate players: pick a region, see how land has changed over time, price history, comparisons.
- **Trading (later stage).** Stock-market-style features: analyse land, see price history, and buy through the portal in seconds.
- **Nominal transaction fee.** Our assumption is a very small fee per trade, around **0.00018%**, similar in spirit to what stock exchanges charge. This number is a working assumption to be validated.

**Our hypothesis (unproven):** land trade is rare in India not because people do not want to trade, but because trading is hard. When India moved its stock market to electronic trading, participation grew enormously. If land trade becomes as easy, we expect far more trades, possibly an order of magnitude or more (we have used ~50x as a planning figure), which could let governments lower stamp duty and still collect the same or more revenue.

## Trust, legal and regulation

The biggest hurdles are legal rather than technical:

- Users must complete **KYC and land-ownership verification** (proposed to be done through banks) before they can trade.
- A dedicated **regulator for land markets**, similar in role to SEBI for securities, would be needed.
- Strategy: start with large, organised players (real-estate firms, institutions), then widen access.
- Compared with the stock market, which has a limited number of listed companies, land has vastly more participants, so both technical and legal complexity are higher. The expected payoff is a more transparent system with much less room for corruption.

## Open research question

How should the value of a *group* of ALUs relate to the values of the individual ALUs inside it? Treating every ALU as unique, while recognising that a bundle trades differently, looks like a research problem in its own right and could support a academic paper.

## Technology stack

### Built in the prototypes

| Layer | Technology |
|---|---|
| Frontend (marketplace) | React 19, TypeScript, Vite, MapLibre GL, ethers (wallet) |
| Frontend (WebGIS) | Leaflet with an OpenStreetMap basemap |
| Backend / API | Python, FastAPI |
| Spatial database | PostgreSQL 16 with PostGIS; GeoPandas, Shapely, PyProj |
| Evidence collection | TinyFish web agent; Fastify (Node.js) service that gathers and hashes the 21-field evidence sheet |
| Trade rules | Six fixed checks in the backend: `LISTED`, `BUNDLE_INTACT`, `KYC_WALLET`, `ZONING_OK`, `FUNDS_IN_ESCROW`, `TRADE_MATCH` |
| Settlement | Solidity escrow contract, Hardhat, Sepolia testnet, MetaMask |
| Geographic data | Government of India BharatMapService (Survey of India administrative boundaries) |
| Packaging | Docker Compose (PostGIS), Makefile and npm scripts |

### Planned for the full vision

| Need | Planned approach |
|---|---|
| Land-feature detection | YOLO-family vision models on satellite and aerial imagery |
| One-time national computation | Batch processing on GPU-capable cloud servers, run coarse-to-fine |
| Fast, cheap serving | Pre-computed map tiles served statically with lazy loading |
| Real parcel geometry | Import of authorised state land-record data (for example BhuNaksha) |
| Human review | Confidence-scored review queue for yellow and red cases |
| Rural access | Dial-code (USSD-style) or voice gateway |
| Real identity and ownership checks | Bank-grade KYC integration |

## Repositories

| Repository | What it contains |
|---|---|
| **[Bhoomi2.0](https://github.com/prathambansaliasofficial-ai/Bhoomi2.0)** (this repo) | Overview of the vision, method and business |
| **[Kritikal_Hacks_Bace_Hackers](https://github.com/prathambansaliasofficial-ai/Kritikal_Hacks_Bace_Hackers)** | Land Market prototype: React frontend, FastAPI + PostGIS backend, TinyFish middleware, testnet escrow |
| **[Kritikal-Hacks-26-Bace-Hackers-Re-imagining-how-land-use-to-Trade](https://github.com/yashbaghel03-debug/Kritikal-Hacks-26-Bace-Hackers-Re-imagining-how-land-use-to-Trade)** | WebGIS and ALU grid on real Indian boundary data, plus the 21-category data pipeline |

## Roadmap

- [x] ALU hierarchy and zoom-dependent grid
- [x] Land-market prototype with rule engine and testnet escrow
- [x] Evidence collection and review workflow (prototype)
- [ ] Import real cadastral parcel geometry from authorised sources
- [ ] AI feature detection (YOLO) and confidence scoring at scale
- [ ] Pre-computed tiles and public deployment
- [ ] Real KYC and ownership verification
- [ ] Rural dial-code access
- [ ] Pilot with institutional users

## Honest limits

- Demo parcels, owners and evidence are **fictional fixtures**, not official government records.
- The escrow runs on a **testnet only**; a blockchain receipt is **not** a legal transfer of title.
- KYC in the prototype is a stub, not real identity verification.
- The efficiency figures (20% / 50% / 90%), the fee level and the "50x more trade" idea are **estimates and hypotheses**, not measured results.
- Real cadastral polygons are state-specific and must come from authorised sources, never from guessed boundaries.

## Team

Team Bace Hackers: Pratham Bansal ([@prathambansaliasofficial-ai](https://github.com/prathambansaliasofficial-ai)) and [@yashbaghel03-debug](https://github.com/yashbaghel03-debug).
