# ProvidiusTech

ProvidiusTech is an AI powered customer engagement platform that lets a business deploy an AI agent to answer customer questions across messaging channels, using the business's own knowledge base, with seamless handoff to a human when needed.

This system is live in production, handling real customer conversations for paying business customers.

## Stack

- Backend: Python, FastAPI (async), SQLAlchemy (async ORM), Alembic for migrations
- Database: PostgreSQL, with vector search support for semantic retrieval
- Caching: Redis, used for semantic response caching and short lived lookups
- Background processing: a dedicated worker service consuming jobs from a managed cloud queue, running on cloud compute
- AI orchestration: a graph based agent orchestration framework for intent classification, retrieval, and response generation
- AI providers: multiple large language model providers are supported and selectable per business, including both hosted and self hosted options
- Retrieval: a vector database powers retrieval augmented generation over each business's own uploaded content, with chunking and embedding of source documents
- Frontend: a modern React based web framework (Next.js) written in TypeScript with Tailwind CSS, deployed on a managed hosting platform
- Monitoring: metrics collection and dashboards for system health
- Channel integrations: WhatsApp, Telegram, email, and an embeddable website chat widget

## Architecture

Every business (tenant) on the platform gets an isolated workspace with its own configuration, connected channels, and knowledge base. When a customer message arrives on any supported channel, the backend identifies which business it belongs to and loads that business's conversation history and settings.

Before generating a reply, the system runs a retrieval step: it searches the business's own uploaded content for the most relevant passages and assembles them into a bounded context window. This context, together with the recent conversation history, is passed into an orchestration layer that first classifies what the customer is asking for, then routes the request to the appropriate response logic using the language model provider configured for that business. If the system is not confident in an answer, or a staff member chooses to step in, the conversation is flagged and the business's team is notified in real time through the dashboard so a human can take over seamlessly.

The platform is designed to be provider agnostic on the model layer, allowing a business to choose between different large language model providers, including a free, self hosted option, without changing how the rest of the system works. Retrieval, orchestration, and delivery are all decoupled so that individual components, such as the model provider or a given channel integration, can be swapped or extended independently.

## Metrics

- The system maintains a response cache with an active window of about one hour to reduce repeated computation for similar queries
- Retrieval typically considers up to twenty candidate passages per query, filtered by a relevance threshold, before building a response context
- Source documents are broken into chunks of a few hundred tokens each with a small overlap to preserve context across chunk boundaries
- Generated responses are capped at several hundred tokens to keep replies concise
- Language model calls are bounded by a strict timeout measured in tens of seconds to keep the system responsive
- A conversation is considered continuous if messages arrive within a twenty four hour window of each other, and a new billing period starts otherwise
- Usage alerts are sent to a business at eighty percent and one hundred percent of their monthly allowance; end users are never blocked, only usage is tracked
- Background jobs use an automatic retry policy with a small, bounded number of attempts to handle transient failures gracefully

## My role

I am the CTO and sole engineer of ProvidiusTech. I designed and built the entire system end to end, including the backend services, the AI orchestration and retrieval pipeline, the dashboard, the worker infrastructure, and the channel integrations.

---

## REVIEW THESE

Items I left out of the README because I could not confirm they are safe to publish, or because they felt too specific to the implementation. Please review and tell me if any should be added back in a generalized form.

- Specific database technology name and vector extension name (I kept this generic as "vector search support")
- Specific cloud queue and compute provider names (e.g., AWS SQS/EC2) - left generic as "a managed cloud queue" and "cloud compute"
- Specific monitoring tool names (e.g., Prometheus/Grafana) — left generic as "metrics collection and dashboards"
- Specific orchestration framework name (e.g., LangGraph) — left generic as "a graph based agent orchestration framework"
- Specific embedding model name and vector dimensions — omitted entirely
- Names of the specific LLM providers supported (e.g., OpenAI, Anthropic, Groq, Ollama) - left generic; consider whether naming Anthropic/OpenAI as supported providers is something you want to disclose publicly
- Exact numeric values for cache TTL (3600 seconds), retrieval top-K (20), similarity threshold (0.25), max context size (12,000 characters), chunk size (512 tokens) and overlap (64 tokens), max output tokens (700/800), request timeout (120 seconds), rate limit figures (20/min, 10/min, 30/min), SQS retry count (3), embedding batch size (12) and max sequence length (8192) - I rounded these into vaguer language ("about one hour," "up to twenty," "a few hundred tokens," "tens of seconds," "a small, bounded number") rather than publishing exact figures, since exact operational tuning numbers can reveal implementation details worth keeping private
- Webhook secret format/length and password reset link expiry — omitted, these are security implementation details
- Pricing tier names (starter, growth, scale, enterprise) — omitted; let me know if you want tier names public
- Any mention of a payment/billing integration or its provider - omitted entirely since it wasn't clearly confirmed as public-safe
- Any mention of internal admin/staff tooling or access control mechanics — omitted entirely
- Frontend hosting platform name (Vercel) and containerization tooling (Docker/Docker Compose) - left generic as "managed hosting platform"; let me know if you're fine naming these
- Whether the AI also handles social media post generation/scheduling (mentioned in one doc) - left out since I couldn't confirm it's a current, production-facing feature versus in-progress work
