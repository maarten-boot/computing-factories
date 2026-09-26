# computing-factories
Computing factories, a old idea i have 30 years ago

Q:

Ideas about a computing environment mimicking factories and workplaces in the physical world.

The idea is that computing could be outsourced in relevant steps.
Just like large and complex constructions like bridges and airplanes or flat buildings are planned, outsourced by components and later assembled in steps and sent back to the requestor, computing of complex data could be broken down in parts and send to computing `factories` for heavy work and when the result is known it can be sent back to the requestor.

The idea considered a collection of information directories or brokers to act as who-is-who provider to find specific compute services needed by the tasks and a routing highway for sending and receiving requests and responses based on sender id and receiver id, the receiver being provided by the lookup service of the brokers or index directories like the yellow pages or the telephone book.

Just like trucks trains and boats transport half finished components between requestor of a service and providers of a service a message highway net could transport compute requests to specialized endpoints based on the type of computing required.

It is currently a general idea that dates back 30 to 40 years back, so some of the infrastructure has naturally manifested as the internet , the transport network and cloud computing as places where compute factories could be hosted.

look at the idea , reflect and provide feedback from your standpoint.

A:

This is a genuinely interesting idea to unpack, partly because so much of it *has* actually happened — just not all in one clean, coherent system the way you're describing it. Let me go through what resonates, what's already been tried (and why it succeeded or stalled), and where I think the idea still has teeth.

**What's already been built along these lines**

Your "brokers as yellow pages" concept is almost exactly what UDDI (Universal Description, Discovery, and Integration) tried to do in the early 2000s alongside SOAP/WSDL — a registry where services published their capabilities and consumers looked them up. It mostly died. The reason is instructive: technical discovery (finding *an* endpoint that claims to do X) turned out to be the easy part. The hard part was semantic trust — knowing whether "compute matrix inversion" from provider A means the same thing, with the same guarantees, as from provider B. Yellow pages work for phone numbers because "give me a plumber in this zip code" has almost no ambiguity. "Give me a compute service that transforms my data correctly" has enormous ambiguity, and that's where the metaphor strains.

The "factory" half of your idea is closer to what actually stuck: grid computing (Globus, SETI@home-style volunteer computing), and now serverless/FaaS (AWS Lambda, Step Functions) plus workflow orchestrators (Airflow, Temporal). These do genuinely decompose big jobs into steps, ship them to specialized compute, and reassemble results — your bridge/airplane analogy is basically the mental model behind MapReduce and modern data pipelines. Message brokers like Kafka or RabbitMQ are your "routing highway" made literal, with sender/receiver addressing.

**Where I think your framing adds something these systems still lack**

Most of what exists optimizes for *one organization's* internal factory floor — one company's cloud, one company's task queue. Your version is more like an actual open marketplace of independent factories, which is closer to something like a supply chain than an internal assembly line. That's rarer and harder, and it maps onto things like blockchain "oracle" networks or decentralized compute marketplaces (Golem, Akash) — which exist but remain niche, mostly because of the same trust/verification problem: how do you know the "factory" did the computation honestly and didn't just return garbage or leak your data? Physical manufacturing solves this with reputation, contracts, and physical inspection of the delivered part. Verifying a *computation* was done correctly without redoing it yourself is a much harder problem (this is literally an active cryptography research area — verifiable computation, zero-knowledge proofs of computation).

**The economic asymmetry that physical logistics doesn't have**

Trucks and trains are worth using because moving a half-built wing is cheaper than building a second factory. But for a lot of computing, moving the *data* to the factory costs more than just running the computation locally — bandwidth and serialization overhead can dwarf compute cost, especially for anything data-heavy rather than compute-heavy. This is why cloud computing centralized rather than fragmenting into a marketplace of specialist shops: it's cheaper to keep data close to elastic general-purpose compute than to ship it around to specialists. Your idea works best precisely for the cases where computation is genuinely specialized and heavy relative to the data volume (e.g., a small input producing a huge search space, or requiring hardware you don't own — this is basically why GPU-as-a-service and specialized AI inference exist).

**Where I'd push you if you wanted to develop this further**

- The interesting unsolved problem isn't the routing/highway — that's solved (internet, message queues). It's trust and verification of remote computation, and semantic discovery (finding the *right* factory for a genuinely novel task, not a commodity one).
- A modern version of your idea might look less like "generic yellow pages for any compute" and more like narrow, high-trust verticals — which is exactly what's emerging with things like decentralized GPU marketplaces or specialized model-inference marketplaces, each with their own reputation and verification layer, rather than one universal directory.

It's a solid, well-formed intuition that anticipated real infrastructure trends decades early — the parts that materialized (message-passing, task decomposition, service discovery within a company) did so because they solved a real cost problem; the parts that stayed conceptual (open cross-organizational marketplaces with a universal directory) stalled on trust and semantics rather than on the transport/routing idea itself.
