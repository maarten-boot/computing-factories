# computing-factories
Computing factories, a old idea i have 30 years ago

Me:

Ideas about a computing environment mimicking factories and workplaces in the physical world.

The idea is that computing could be outsourced in relevant steps.
Just like large and complex constructions like bridges and airplanes or flat buildings are planned, outsourced by components and later assembled in steps and sent back to the requestor, computing of complex data could be broken down in parts and send to computing `factories` for heavy work and when the result is known it can be sent back to the requestor.

The idea considered a collection of information directories or brokers to act as who-is-who provider to find specific compute services needed by the tasks and a routing highway for sending and receiving requests and responses based on sender id and receiver id, the receiver being provided by the lookup service of the brokers or index directories like the yellow pages or the telephone book.

Just like trucks trains and boats transport half finished components between requestor of a service and providers of a service a message highway net could transport compute requests to specialized endpoints based on the type of computing required.

It is currently a general idea that dates back 30 to 40 years back, so some of the infrastructure has naturally manifested as the internet , the transport network and cloud computing as places where compute factories could be hosted.

look at the idea , reflect and provide feedback from your standpoint.

You:

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

Me:

Ok that is super feedback i can think about.

You:

Glad it landed. Sounds like a good chunk to sit with — the trust/verification angle in particular seems like the part that would decide whether a modern version of this is even worth building versus staying a nice thought experiment.

If you want to poke at it further sometime, a few threads that might be fun to pull:

- How narrow verticals with reputation systems (rather than a universal directory) could actually get you most of the benefit without the hardest problem
- Whether there's a cheap way to spot-check remote computation without fully verifying it (sampling, redundancy, occasional re-runs) — a poor man's version of verifiable computation
- Where the data-movement-cost math actually favors your model today, since that seems to be the real filter for whether "shipping compute" beats "just do it locally"

No rush — just flagging in case any of them are useful hooks next time you come back to it.

Me:

Yes — and this flips the economic asymmetry I raised a moment ago almost entirely in your favor, which is worth sitting with, because it resolves the exact objection I made.

**This is already a named, foundational principle — worth knowing the lineage**

"Move computation to the data, not data to the computation" is literally one of the founding design principles of Hadoop/MapReduce (Google's original paper on this, ~2004, explicitly argues it's cheaper to ship a small program to where petabytes sit than to move the petabytes). It's also the whole rationale behind:

- **In-database computation**: stored procedures, and more recently things like Snowflake's or BigQuery's ability to run user-defined functions right next to the data rather than exporting it
- **Spark's data locality scheduling**: the scheduler explicitly tries to place a task on the same node (or rack) that already holds the data block it needs
- **Edge computing**: pushing inference or filtering logic out to IoT devices or edge nodes so raw sensor data never has to traverse the network at all — only the distilled result travels back

So your "compute factories" idea, recombined with this insight, becomes less like a marketplace of remote specialist workshops and more like sending a blueprint/robot to an existing warehouse rather than trucking the warehouse's inventory somewhere else to be assembled. That's a genuinely different logistics model, and it's the one that actually won in practice for big-data workloads.

**Why this also weakens my earlier "trust and verification" objection — partially**

If the compute travels to sit inside infrastructure the data owner already controls (their database, their messaging system, their cloud region), the data owner retains custody the whole time. They don't have to trust a stranger's black-box factory with their data — they're just trusting a piece of mobile code to run inside their own walled garden, which is a much more tractable security problem (sandboxing, permissions, resource quotas — solved territory: think AWS Lambda@Edge, or how Kafka Streams/ksqlDB let you attach processing logic directly to a topic without extracting the data).

Where the trust problem *doesn't* go away: verifying the code itself did what it claimed and isn't exfiltrating data on the sly once it's inside your perimeter. That becomes a code-supply-chain trust problem instead of a computation-result trust problem — arguably more tractable (static analysis, code signing, sandboxed permissions) than "did the remote server actually run my algorithm correctly."

**Where this reframing sharpens your original idea**

Your brokers/yellow-pages concept becomes less "who can do this computation" and more "where does the relevant data currently live, and what execution environments does that location support" — which is closer to how service meshes and data catalogs (e.g., data lineage/catalog tools like Apache Atlas) already work: they're indexes of *where data sits* and *what's allowed to touch it*, not indexes of generic compute providers.

One tension this doesn't fully resolve: it only works when the data owner is willing to host arbitrary third-party logic inside their infrastructure at all — which is a much bigger ask than "let me send you a request and get an answer back." That's the real gate: today this pattern thrives *within* one company's cloud account, but cross-organizationally it's rare, precisely because letting someone else's code run inside your perimeter is a much scarier trust boundary than letting them query an API.
