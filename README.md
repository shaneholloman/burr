# <img src="https://github.com/user-attachments/assets/2ab9b499-7ca2-4ae9-af72-ccc775f30b4e" width=25 height=25/> Burr

<div>

[![Discord](https://img.shields.io/badge/Join-Burr_Discord-7289DA?logo=discord)](https://discord.gg/6Zy2DwP4f3)
[![Downloads](https://static.pepy.tech/badge/burr/month)](https://pepy.tech/project/burr)
![PyPI Downloads](https://static.pepy.tech/badge/burr)
[![GitHub Last Commit](https://img.shields.io/github/last-commit/dagworks-inc/burr)](https://github.com/dagworks-inc/burr/pulse)
[![X](https://img.shields.io/badge/follow-%40burr_framework-1DA1F2?logo=x&style=social)](https://twitter.com/burr_framework)
<a href="https://app.commanddash.io/agent/github_DAGWorks-Inc_burr"><img src="https://img.shields.io/badge/AI-Code%20Agent-EB9FDA"></a>
<a target="_blank" href="https://linkedin.com/showcase/dagster" style="background:none">
  <img src="https://img.shields.io/badge/DAGWorks-Follow-purple.svg?logo=linkedin" />
</a>
<a href="https://twitter.com/burr_framework" target="_blank">
  <img src="https://img.shields.io/badge/burr_framework-Follow-purple.svg?logo=X"/>
</a>
<a href="https://twitter.com/dagworks" target="_blank">
  <img src="https://img.shields.io/badge/DAGWorks-Follow-purple.svg?logo=X"/>
</a>

</div>

Burr makes it easy to develop applications that make decisions (chatbots, agents, simulations, etc...) from simple python building blocks.

Burr works well for any application that uses LLMs, and can integrate with any of your favorite frameworks. Burr includes a UI that can track/monitor/trace your system in real time, along with
pluggable persisters (e.g. for memory) to save & load application state.

Link to [documentation](https://burr.dagworks.io/). Quick (<3min) video intro [here](https://www.loom.com/share/a10f163428b942fea55db1a84b1140d8?sid=1512863b-f533-4a42-a2f3-95b13deb07c9).
Longer [video intro & walkthrough](https://www.youtube.com/watch?v=rEZ4oDN0GdU). Blog post [here](https://blog.dagworks.io/p/burr-develop-stateful-ai-applications). Join discord for help/questions [here](https://discord.gg/6Zy2DwP4f3).

## 🏃Quick start

Install from `pypi`:

```bash
pip install "burr[start]"
```

(see [the docs](https://burr.dagworks.io/getting_started/install/) if you're using poetry)

Then run the UI server:

```bash
burr
```

This will open up Burr's telemetry UI. It comes loaded with some default data so you can click around.
It also has a demo chat application to help demonstrate what the UI captures enabling you too see things changing in
real-time. Hit the "Demos" side bar on the left and select `chatbot`. To chat it requires the `OPENAI_API_KEY`
environment variable to be set, but you can still see how it works if you don't have an API key set.

Next, start coding / running examples:

```bash
git clone https://github.com/dagworks-inc/burr && cd burr/examples/hello-world-counter
python application.py
```

You'll see the counter example running in the terminal, along with the trace being tracked in the UI.
See if you can find it.

For more details see the [getting started guide](https://burr.dagworks.io/getting_started/simple-example/).

## 🔩 How does Burr work?

With Burr you express your application as a state machine (i.e. a graph/flowchart).
You can (and should!) use it for anything in which you have to manage state, track complex decisions, add human feedback, or dictate an idempotent, self-persisting workflow.

The core API is simple -- the Burr hello-world looks like this (plug in your own LLM, or copy from [the docs](https://burr.dagworks.io/getting_started/simple-example/#build-a-simple-chatbot>) for _gpt-X_)

```python
from burr.core import action, State, ApplicationBuilder

@action(reads=[], writes=["prompt", "chat_history"])
def human_input(state: State, prompt: str) -> State:
    # your code -- write what you want here!
    return state.update(prompt=prompt).append(chat_history=chat_item)

@action(reads=["chat_history"], writes=["response", "chat_history"])
def ai_response(state: State) -> State:
    response = _query_llm(state["chat_history"]) # Burr doesn't care how you use LLMs!
    return state.update(response=content).append(chat_history=chat_item)

app = (
    ApplicationBuilder()
    .with_actions(human_input, ai_response)
    .with_transitions(
        ("human_input", "ai_response"),
        ("ai_response", "human_input")
    ).with_state(chat_history=[])
    .with_entrypoint("human_input")
    .build()
)
*_, state = app.run(halt_after=["ai_response"], inputs={"prompt": "Who was Aaron Burr, sir?"})
print("answer:", app.state["response"])
```

Burr includes:

1. A (dependency-free) low-abstraction python library that enables you to build and manage state machines with simple python functions
2. A UI you can use view execution telemetry for introspection and debugging
3. A set of integrations to make it easier to persist state, connect to telemetry, and integrate with other systems

![Burr at work](https://github.com/DAGWorks-Inc/burr/blob/main/chatbot.gif)

## 💻️ What can you do with Burr?

Burr can be used to power a variety of applications, including:

1. [A simple gpt-like chatbot](https://github.com/dagworks-inc/burr/tree/main/examples/multi-modal-chatbot)
2. [A stateful RAG-based chatbot](https://github.com/dagworks-inc/burr/tree/main/examples/conversational-rag/simple_example)
3. [An LLM-based adventure game](https://github.com/DAGWorks-Inc/burr/tree/main/examples/llm-adventure-game)
4. [An interactive assistant for writing emails](https://github.com/DAGWorks-Inc/burr/tree/main/examples/email-assistant)

As well as a variety of (non-LLM) use-cases, including a time-series forecasting [simulation](https://github.com/DAGWorks-Inc/burr/tree/main/examples/simulation),
and [hyperparameter tuning](https://github.com/DAGWorks-Inc/burr/tree/main/examples/ml-training).

And a lot more!

Using hooks and other integrations you can (a) integrate with any of your favorite vendors (LLM observability, storage, etc...), and
(b) build custom actions that delegate to your favorite libraries (like [Hamilton](https://github.com/DAGWorks-Inc/hamilton)).

Burr will _not_ tell you how to build your models, how to query APIs, or how to manage your data. It will help you tie all these together
in a way that scales with your needs and makes following the logic of your system easy. Burr comes out of the box with a host of integrations
including tooling to build a UI in streamlit and watch your state machine execute.

## 🏗 Start building

See the documentation for [getting started](https://burr.dagworks.io/getting_started/simple-example), and follow the example.
Then read through some of the concepts and write your own application!

## 📃 Comparison against common frameworks

While Burr is attempting something (somewhat) unique, there are a variety of tools that occupy similar spaces:

| Criteria                                          | Burr | Langgraph | temporal | Langchain | Superagent | Hamilton |
| ------------------------------------------------- | :--: | :-------: | :------: | :-------: | :--------: | :------: |
| Explicitly models a state machine                 |  ✅  |    ✅     |    ❌    |    ❌     |     ❌     |    ❌    |
| Framework-agnostic                                |  ✅  |    ✅     |    ✅    |    ✅     |     ❌     |    ✅    |
| Asynchronous event-based orchestration            |  ❌  |    ❌     |    ✅    |    ❌     |     ❌     |    ❌    |
| Built for core web-service logic                  |  ✅  |    ✅     |    ❌    |    ✅     |     ✅     |    ✅    |
| Open-source user-interface for monitoring/tracing |  ✅  |    ❌     |    ❌    |    ❌     |     ❌     |    ✅    |
| Works with non-LLM use-cases                      |  ✅  |    ❌     |    ❌    |    ❌     |     ❌     |    ✅    |

## 🌯 Why the name Burr?

Burr is named after [Aaron Burr](https://en.wikipedia.org/wiki/Aaron_Burr), founding father, third VP of the United States, and murderer/arch-nemesis of [Alexander Hamilton](https://en.wikipedia.org/wiki/Alexander_Hamilton).
What's the connection with Hamilton? This is [DAGWorks](www.dagworks.io)' second open-source library release after the [Hamilton library](https://github.com/dagworks-inc/hamilton)
We imagine a world in which Burr and Hamilton lived in harmony and saw through their differences to better the union. We originally
built Burr as a _harness_ to handle state between executions of Hamilton DAGs (because DAGs don't have cycles),
but realized that it has a wide array of applications and decided to release it more broadly.

## Testimonials
<link rel="stylesheet" type="text/css" href="https://raw.githubusercontent.com/DAGWorks-Inc/burr/refs/heads/main/docs/_static/testimonials.css">
<div class="testimonial-container">
<div class="testimonial-card">
    <div class="testimonial-content">
        <p>"After evaluating several other obfuscating LLM frame-works, their elegant yet comprehensive state management solution proved to be the powerful answer to rolling out robots driven by AI decision making."</p>
        <h4>Ashish Ghosh</h4>
        <span>CTO, Peanut Robotics</span>
    </div>
</div>
<br/>
---
<div class="testimonial-card">
    <div class="testimonial-content">
        <p>"Of course, you can use it [LangChain], but whether it's really production-ready and improves the time from 'code-to-prod' [...], we've been doing LLM apps for two years, and the answer is no [...] All these 'all-in-one' libs suffer from this [...].  Honestly, take a look at Burr. Thank me later."</p>
        <h4>Reddit User</h4>
        <span>LocalLlama, Subreddit</span>
    </div>
</div>
<br/>
---
<div class="testimonial-card">
    <div class="testimonial-content">
        <p>"Using Burr is a no-brainer if you want to build a modular AI application. It is so easy to build with and I especially love their UI which makes debugging, a piece of cake. And the always ready to help team, is the cherry on top."</p>
        <h4>Ishita</h4>
        <span>Founder, Watto.ai</span>
    </div>
</div>
<br/>
---
<div class="testimonial-card">
    <div class="testimonial-content">
        <p>"I just came across Burr and I'm like WOW, this seems like you guys predicted this exact need when building this. No weird esoteric concepts just because it's AI."</p>
        <h4>Matthew Rideout</h4>
        <span>Staff Software Engineer, Paxton AI</span>
    </div>
</div>
<br/>
<div class="testimonial-card">
    <div class="testimonial-content">
        <p>"Burr's state management part is really helpful for creating state snapshots and build debugging, replaying and even building evaluation cases around that"</p>
        <h4>Rinat Gareev</h4>
        <span>Senior Solutions Architect, Provectus</span>
    </div>
</div>
<div class="testimonial-card">
    <div class="testimonial-content">
        <p>"I have been using Burr over the past few months, and compared to many agentic LLM platforms out there (e.g. LangChain, CrewAi, AutoGen, Agency Swarm, etc), Burr provides a more robust framework for designing complex behaviors."</p>
        <h4>Hadi Nayebi</h4>
        <span>Co-founder, CognitiveGraphs</span>
    </div>
</div>
<br/>
---
<div class="testimonial-card">
    <div class="testimonial-content">
        <p>"Moving from LangChain to Burr was a game-changer! <br/>Time-Saving: It took me just a few hours to get started with Burr, compared to the days and weeks I spent trying to navigate LangChain. <br/>Cleaner Implementation: With Burr, I could finally have a cleaner, more sophisticated, and stable implementation. No more wrestling with complex codebases. <br/>Team Adoption: I pitched Burr to my teammates, and we pivoted our entire codebase to it. It's been a smooth ride ever since."</p>
        <h4>Aditya K.</h4>
        <span>DS Architect, TaskHuman</span>
    </div>
</div>
</div>

## 🛣 Roadmap

While Burr is stable and well-tested, we have quite a few tools/features on our roadmap!
1. FastAPI integration + hosted deployment -- make it really easy to get Burr in an app in production without thinking about REST APIs
2. Various efficiency/usability improvements for the core library (see [planned capabilities](https://burr.dagworks.io/concepts/planned-capabilities/) for more details). This includes:
   1. First-class support for retries + exception management
   2. More integration with popular frameworks (LCEL, LLamaIndex, Hamilton, etc...)
   3. Capturing & surfacing extra metadata, e.g. annotations for particular point in time, that you can then pull out for fine-tuning, etc.
   4. Improvements to the pydantic-based typing system
3. Tooling for hosted execution of state machines, integrating with your infrastructure (Ray, modal, FastAPI + EC2, etc...)
4. Additional storage integrations. More integrations with technologies like MySQL, S3, etc. so you can run Burr on top of what you have available.

If you want to avoid self-hosting the above solutions we're building Burr Cloud. To let us know you're interested
sign up [here](https://forms.gle/w9u2QKcPrztApRedA) for the waitlist to get access.

## 🤲 Contributing

We welcome contributors! To get started on developing, see the [developer-facing docs](https://burr.dagworks.io/contributing).

## 👪 Contributors

### Code contributions

Users who have contributed core functionality, integrations, or examples.

- [Elijah ben Izzy](https://github.com/elijahbenizzy)
- [Stefan Krawczyk](https://github.com/skrawcz)
- [Joseph Booth](https://github.com/jombooth)
- [Nandani Thakur](https://github.com/NandaniThakur)
- [Thierry Jean](https://github.com/zilto)
- [Hamza Farhan](https://github.com/HamzaFarhan)
- [Abdul Rafay](https://github.com/proftorch)

### Bug hunters/special mentions

Users who have contributed small docs fixes, design suggestions, and found bugs

- [Luke Chadwick](https://github.com/vertis)
- [Evans](https://github.com/sudoevans)
- [Sasmitha Manathunga](https://github.com/mmz-001)
