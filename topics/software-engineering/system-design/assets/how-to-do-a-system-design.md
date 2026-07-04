# How to do a System Design of a Software Application

Generally, all System Designs follow the same strategy.

![system-design-plan](system-design-plan.png)

---
## Requirements

1. Start your interview by defining the functional and non-functional requirements. For user facing applications like this one, functional requirements are the "Users should be able to..." statements whereas non-functional defines the system qualities via "The system should..." statements.
2. Prioritize the top 3 functional requirements. Everything else shows your product thinking, but clearly note it as "below the line" so the interviewer knows you won't be including them in your design. Check in to see if your interviewer wants to move anything above the line or move anything down. Choosing just the top 3 is important to ensuring you stay focused and can execute in the limited time window.
3. When evaluating non-functional requirements, it's crucial to uncover the distinct characteristics that make this system challenging or unique. To help you identify these, consider the following questions:
    - **CAP theorem:** Does this system prioritize availability or consistency? Note, that in some cases, the answer is different depending on the part of the system -- as you'll see is the case here.
	- **Read vs write ratio:** is this a read heavy system or write heavy? Are either notably heavy for any reason?
    - **Query access pattern:** Is the access pattern of the system regular or are there patterns or bursts that require particular attention. For example, the holidays for shopping sites or popular events for ticket booking.

---
## Core Entities

A broad overview of the primary entities. At this stage, it is not necessary to know every specific column or detail. We will focus on the intricacies, such as columns and fields, later when we have a clearer grasp. Initially, establishing these key entities will guide our thought process and lay a solid foundation as we progress towards defining the API.

Example:

```
// Core Entities

- Venue
- Performer
- Ticket

```

---
## API

At this point we provide simple (most likely RESTful) APIs that can satisfy our functional requirements.

Example:

```
GET /events/:eventId -> Event & Venue & Performer & Ticket[]

POST /bookings/:eventId -> bookingId 
headers: Authorization - JWT auth bearer token
body - {"ticketIds": string[], "paymentDetails": ... }

```

---
## High-level Design

Here we provide a High-Level Design of the system.

I really like using https://excalidraw.com/ for it.

Almost exclusively start with the client, an API gateway that handles Authorization, routing & rate limiting and then move on to the rest of the system.

## Deep Dives

In this section we can dive deeply into some of the implementations

E.x: how are we going to make sure no ticket is booked by different people? (Distributed Lock)

How are we going to handle increased load during popular events? (Load Balancer, Horizontal Scaling, Caching)

---