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
