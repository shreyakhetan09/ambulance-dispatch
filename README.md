# Speedy Siren

Emergency ambulance dispatch: given a caller’s location on a city grid, find
the **nearest hospital** and the **lowest-cost route** to it, then visualize
the path. The backend runs Dijkstra on a weighted street grid; the React
frontend books the ride, estimates intra-city fare, and draws the route.

<p>
  <img alt="React" src="https://img.shields.io/badge/React-61DAFB?style=flat&logo=react&logoColor=black" />
  <img alt="Node.js" src="https://img.shields.io/badge/Node.js-339933?style=flat&logo=nodedotjs&logoColor=white" />
  <img alt="Express" src="https://img.shields.io/badge/Express-000000?style=flat&logo=express&logoColor=white" />
  <img alt="Dijkstra" src="https://img.shields.io/badge/Routing-Dijkstra-0A66C2?style=flat" />
  <img alt="Bootstrap" src="https://img.shields.io/badge/Bootstrap-7952B3?style=flat&logo=bootstrap&logoColor=white" />
</p>

In a real emergency the clock starts before the wheels do. Speedy Siren
answers the first operational question: *which hospital can reach this
intersection fastest, and which streets should the ambulance take?*

![Speedy Siren landing](https://user-images.githubusercontent.com/79561540/229322202-265cada4-4dba-470c-9490-23cdb4f67e9c.png)

---

## Why it matters

Calling a hospital and hoping an ambulance is free is a phone tree, not a
system. This project treats the city as a **weighted graph**: every block
is a node, every street is an edge with a travel cost, and hospitals sit at
known coordinates. One request returns the winning hospital, the path, and
the total cost — the same idea used in live dispatch, reduced to a demo you
can run locally.

## What it does

- **Book an ambulance** from the landing page — enter a grid row and column
  for the caller.
- **Nearest-hospital search** — Dijkstra from the caller to every hospital
  in `server/hopitals.json`; the lowest-cost destination wins.
- **Route visualization** — a graph view (`/graph`) highlights the chosen
  path on the city grid.
- **Intra-city fare estimate** — a simple cost readout for the selected
  trip.
- **Contact / WhatsApp** — a public contact path so a bystander can reach
  dispatch without installing a native app.

## How routing works

```mermaid
flowchart LR
    C["Caller location<br/>row, column"] --> API["POST /find-hospital"]
    H["Hospital catalogue<br/>grid x,y + ambulance count"] --> API
    API --> G["Build 9×9 street graph<br/>random edge weights"]
    G --> D["Dijkstra from caller<br/>to every hospital"]
    D --> W["Lowest-cost hospital + path"]
    W --> UI["React map + fare"]
```

```mermaid
sequenceDiagram
    participant User as Caller
    participant Web as React app
    participant API as Express API
    participant Graph as Street graph
    User->>Web: Enter row / column
    Web->>API: POST /find-hospital
    API->>Graph: Dijkstra(start, each hospital)
    Graph-->>API: distance + path per hospital
    API-->>Web: winning hospital, path, cost
    Web-->>User: Route on /graph + fare
```

Edge weights stand in for traffic. In this demo they are sampled per run;
the algorithm is the same if those weights come from a live traffic feed.

## Architecture

```
speedy-siren-app/
├── client/                 React 18 + React Router + Bootstrap
│   ├── src/pages/Home      landing, booking, about
│   ├── src/pages/Graph     route visualization (react-graph-vis)
│   └── src/components      fare calculator, contact, WhatsApp
└── server/                 Express on port 3002
    ├── index.js            Graph + Dijkstra + /find-hospital
    └── hopitals.json       hospital name, grid cell, ambulance count
```

| Layer | Choice |
| ----- | ------ |
| UI | React 18, React Router, Bootstrap |
| Routing API | Express, CORS |
| Pathfinding | Dijkstra on an undirected weighted grid |
| Map | `react-graph-vis` |

## Quickstart

The API and the UI run as two processes.

```bash
# 1. routing API
cd speedy-siren-app/server
npm install
npm start          # http://localhost:3002

# 2. web app (second terminal)
cd speedy-siren-app/client
npm install
npm start          # http://localhost:3000
```

Open the home page, book an ambulance with a grid coordinate, then open
`/graph` to see the selected path.

`GET /test` on the API should return `{ "message": "Server is working" }`.

## Screenshots

![Problem framing](https://user-images.githubusercontent.com/79561540/229323609-db16516f-439d-45af-8ca3-9d7e5f97c18b.png)
![Booking UI](https://user-images.githubusercontent.com/79561540/229323302-66a25a9c-c4ce-4996-bd29-065751990cd3.png)
![Service detail](https://user-images.githubusercontent.com/79561540/229323341-01851c55-a123-4827-acbb-c5e19a71f044.png)
![Route graph](https://user-images.githubusercontent.com/79561540/229334211-f53b148f-1361-4c22-b184-ff813a38c785.png)
![Fare and contact](https://user-images.githubusercontent.com/79561540/229323286-82b27ccc-1bd3-48a7-afbf-ffc0d39597f1.png)

This started as a team hackathon build. The fork keeps the original product
name and adds a project-level README so the dispatch idea is obvious from
the repo page.
