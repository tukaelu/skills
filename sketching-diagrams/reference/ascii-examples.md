# ASCII Art Examples for Mermaid Diagrams

Sketch style examples for each diagram type. The goal of ASCII art is not perfect representation —
it's a structural checkpoint: "Does this look right?" Use real node names from the user's content.

---

## Flowchart (flowchart)

```
[Start]
  |
  v
[Input validation] --NG--> [Show error] --> [End]
  |
 OK
  |
  v
[Data processing]
  |
  v
[Condition A?] --No--> [Process B]
  |                         |
 Yes                        |
  |                         |
  v                         v
[Process A] ---------->[Output result] --> [End]
```

---

## Sequence Diagram (sequenceDiagram)

```
User           API Server      Database
  |                |               |
  |--POST /login-->|               |
  |                |--SELECT user->|
  |                |<--user record-|
  |<--200 + token--|               |
  |                |               |
  |--GET /data---->|               |
  |  (Authorization: Bearer token) |
  |                |--SELECT data->|
  |                |<--data rows---|
  |<--200 + JSON---|               |
```

---

## Class Diagram (classDiagram)

```
┌───────────────┐            ┌───────────────┐
│   <<abstract>>│            │               │
│    Animal     │<──extends──│      Dog      │
├───────────────┤            ├───────────────┤
│ +name: string │            │ +breed: string│
│ +age: int     │            ├───────────────┤
├───────────────┤            │ +bark(): void │
│ +speak(): str │            └───────────────┘
└───────────────┘
        ^
        │ extends
        │
┌───────────────┐
│      Cat      │
├───────────────┤
│ +indoor: bool │
├───────────────┤
│ +meow(): void │
└───────────────┘
```

---

## ER Diagram (erDiagram)

```
┌──────────────┐        ┌──────────────┐        ┌──────────────┐
│    users     │        │    orders    │        │ order_items  │
│──────────────│        │──────────────│        │──────────────│
│ id (PK)      │1      N│ id (PK)      │1      N│ id (PK)      │
│ name         │───────>│ user_id (FK) │───────>│ order_id (FK)│
│ email        │        │ total_amount │        │ product_id   │
│ created_at   │        │ status       │        │ quantity     │
└──────────────┘        │ created_at   │        │ price        │
                        └──────────────┘        └──────────────┘
```

---

## State Diagram (stateDiagram-v2)

```
        ┌──────────────────────────────┐
        │                              │
        v                              │ retry
    [Idle] ──start──> [Processing] ──error──> [Error]
                           |
                          done
                           |
                           v
                        [Done] ──reset──> [Idle]
```

---

## Gantt Chart (gantt)

```
Task                 | Week1 | Week2 | Week3 | Week4
─────────────────────|───────|───────|───────|──────
Requirements         | █████ |       |       |
System design        |       | █████ |       |
Detailed design      |       | ██    |       |
Implementation       |       |    ██ | █████ | ██
Testing              |       |       |    ██ | ███
Release              |       |       |       |   █
```

---

## Mind Map (mindmap)

```
                  [Project Plan]
                 /       |       \
         [Scope]    [Schedule]   [Resources]
         /    \          |          /     \
     [Feat A][Feat B] [Milestones] [Team] [Budget]
                          |
                    [M1: Design done]
                    [M2: Dev done]
                    [M3: Release]
```

---

## Pie Chart (pie)

```
Revenue breakdown:

Product A : 40% ████████████████
Product B : 35% ██████████████
Product C : 15% ██████
Other     : 10% ████
```

---

## Git Graph (gitGraph)

```
main:    ●─────●─────────────────●─────●
          \                     /
develop:   ●─────●─────────────●
                  \           /
feature/login:     ●─────●───●
```

---

## Timeline (timeline)

```
2020 ─── Project kickoff
          └── Team formed
2021 ─── Beta release
          └── User testing begins
2022 ─── v1.0 official release
          └── Paid plans launched
2023 ─── v2.0 major update
          └── Mobile app support
```

---

## Tips

- For complex diagrams, show only the key paths/elements — no need to represent everything
- Include arrow labels (conditions, action names) to make the structure clearer
- Placeholder names are fine — the goal is to align on structure, not perfect naming
