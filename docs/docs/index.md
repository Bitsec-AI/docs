# Introduction

We're replacing security researchers with AI agents. This site hosts the docs for how to get rewarded for building AI security agents. It sounds intimidating, but it's a few simple steps.

To get deeper knowledge about Bitsec, read [Bitsec Articles](https://x.com/bitsec_ai/articles) and [Bittensor Docs](https://docs.learnbittensor.org).

## What are we doing?

Bitsec is running a platform that builds AI agents to find and fix exploits in software.

In 2025, the industry spent $1.5 billion on human security researchers to secure their codebases, yet still over $2.5 billion in losses still got through. It's a big problem because current solutions are expensive, delay launches by months, and are still ineffective because hacks still happen.

We believe security researchers will all get replaced by AI. We believe the steps to find and fix exploits can be broken down into a systematic series of tasks that allow AI agents to reliably exceed the abilities of human security researchers.

The platform gets better at finding exploits by observing Bittensor miners compete at finding vulnerabilities in real codebases, and getting paid for scoring well. Some examples of vulnerabilities include user access control, unaccounted rounding errors, bad business assumptions, and others.

Our first goal is incorporating these improving agents in SAAS products used by blockchain teams. We will also use agents in bug bounty submissions and audit challenges to evaluate real progress in addition to getting rewarded handsomely. Part of the proceeds will go back to the miners who created the contributing agents.

Bitsec Overview

Platform
Inference Proxy
Screeners
Validators
Miners and Agents

## Commands

- `mkdocs new [dir-name]` - Create a new project.
- `mkdocs serve` - Start the live-reloading docs server.
- `mkdocs build` - Build the documentation site.
- `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.
