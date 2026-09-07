---
title: "Event 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---


# Summary Report: “GenAI-powered App-DB Modernization workshop”

### Event Objectives

1. Share best practices in modern application design
2. Introduce Domain-Driven Design (DDD) and event-driven architecture
3. Provide guidance on selecting the right compute services
4. Present AI tools to support the development lifecycle

### Speakers

1. **Jignesh Shah** – Director, Open Source Databases
2. **Erica Liu** – Sr. GTM Specialist, AppMod
3. **Fabrianne Effendi** – Assc. Specialist SA, Serverless Amazon Web Services

### Key Highlights

#### Identifying the drawbacks of legacy application architecture

1. Long product release cycles → Lost revenue/missed opportunities  
2. Inefficient operations → Reduced productivity, higher costs  
3. Non-compliance with security regulations → Security breaches, loss of reputation  

#### Transitioning to modern application architecture – Microservices

Migrating to a modular system — each function is an **independent service** communicating via **events**, built on three core pillars:

1. **Queue Management**: Handle asynchronous tasks  
2. **Caching Strategy**: Optimize performance  
3. **Message Handling**: Flexible inter-service communication  

#### Domain-Driven Design (DDD)

1. **Four-step method**: Identify domain events → arrange timeline → identify actors → define bounded contexts  
2. **Bookstore case study**: Demonstrates real-world DDD application  
3. **Context mapping**: 7 patterns for integrating bounded contexts  

#### Event-Driven Architecture

1. **3 integration patterns**: Publish/Subscribe, Point-to-point, Streaming  
2. **Benefits**: Loose coupling, scalability, resilience  
3. **Sync vs async comparison**: Understanding the trade-offs  

#### Compute Evolution

1. **Shared Responsibility Model**: EC2 → ECS → Fargate → Lambda  
2. **Serverless benefits**: No server management, auto-scaling, pay-for-value  
3. **Functions vs Containers**: Criteria for appropriate choice  

#### Amazon Q Developer

1. **SDLC automation**: From planning to maintenance  
2. **Code transformation**: Java upgrade, .NET modernization  
3. **AWS Transform agents**: VMware, Mainframe, .NET migration  

### Key Takeaways

#### Design Mindset

1. **Business-first approach**: Always start from the business domain, not the technology  
2. **Ubiquitous language**: Importance of a shared vocabulary between business and tech teams  
3. **Bounded contexts**: Identifying and managing complexity in large systems  

#### Technical Architecture

1. **Event storming technique**: Practical method for modeling business processes  
2. Use **event-driven communication** instead of synchronous calls  
3. **Integration patterns**: When to use sync, async, pub/sub, streaming  
4. **Compute spectrum**: Criteria for choosing between VM, containers, and serverless  

#### Modernization Strategy

1. **Phased approach**: No rushing — follow a clear roadmap  
2. **7Rs framework**: Multiple modernization paths depending on the application  
3. **ROI measurement**: Cost reduction + business agility  

### Applying to Work

1. **Apply DDD** to current projects: Event storming sessions with business teams  
2. **Refactor microservices**: Use bounded contexts to define service boundaries  
3. **Implement event-driven patterns**: Replace some sync calls with async messaging  
4. **Adopt serverless**: Pilot AWS Lambda for suitable use cases  
5. **Try Amazon Q Developer**: Integrate into the dev workflow to boost productivity  

### Event Experience

Attending the **“GenAI-powered App-DB Modernization”** workshop was extremely valuable, giving me a comprehensive view of modernizing applications and databases using advanced methods and tools. Key experiences included:

#### Learning from highly skilled speakers
1. Experts from AWS and major tech organizations shared **best practices** in modern application design.  
2. Through real-world case studies, I gained a deeper understanding of applying **DDD** and **Event-Driven Architecture** to large projects.  

#### Hands-on technical exposure
1. Participating in **event storming** sessions helped me visualize how to **model business processes** into domain events.  
2. Learned how to **split microservices** and define **bounded contexts** to manage large-system complexity.  
3. Understood trade-offs between **synchronous and asynchronous communication** and integration patterns like **pub/sub, point-to-point, streaming**.  

#### Leveraging modern tools
1. Explored **Amazon Q Developer**, an AI tool for SDLC support from planning to maintenance.  
2. Learned to **automate code transformation** and pilot serverless with **AWS Lambda** to improve productivity.  

#### Networking and discussions
1. The workshop offered opportunities to exchange ideas with experts, peers, and business teams, enhancing the **ubiquitous language** between business and tech.  
2. Real-world examples reinforced the importance of the **business-first approach** rather than focusing solely on technology.  

#### Lessons learned
1. Applying DDD and event-driven patterns reduces **coupling** while improving **scalability** and **resilience**.  
2. Modernization requires a **phased approach** with **ROI measurement**; rushing the process can be risky.  
3. AI tools like Amazon Q Developer can significantly **boost productivity** when integrated into the current workflow.  

#### Some event photos
*(Photos recorded from the workshop)*  

> Overall, the event not only provided technical knowledge but also helped me reshape my thinking about application design, system modernization, and cross-team collaboration.
