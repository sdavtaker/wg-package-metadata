# CHAOSS WG: Package Metadata

## What is CHAOSS?

The [Community Health Analytics in Open Source Software (CHAOSS)](https://chaoss.community/) project is a Linux Foundation initiative focused on the health of open source communities.

## What is this Working Group about?

The Package Metadata Working Group explores how different package managers capture, expose, and structure metadata.

In modern open source development, package managers are the central glue that holds ecosystems together. They transform what might appear as disconnected communities and independent projects into cohesive, interdependent systems. As the primary distribution channels, package managers define how software is shared, discovered, and reused—making them one of the most essential components of today’s open source infrastructure.

At the same time, metadata from package managers underpins a wide range of critical tools and workflows. These include Software Bills of Materials (SBOMs), Software Composition Analysis (SCA) tools, vulnerability management, and compliance assessments. Since most of these outputs are generated from package manager data, their accuracy and reliability depend on the quality of metadata provided by package managers.

Today, this metadata is inconsistent, incomplete, and sometimes incorrect. This creates barriers to analyzing community health, slows down security responses, complicates compliance, and weakens the foundations on which open source sustainability rests.

By focusing on improvements to package manager metadata, this CHAOSS initiative seeks to strengthen the ecosystem as a whole. Our goals are to align communities on common metadata needs, foster collaboration across ecosystems, and ensure that metadata supports reliable, accurate, and meaningful insights into the health and trustworthiness of open source software.

This is a new working group, and we are still learning and shaping our direction. Our goals, methods, and outputs may evolve as we progress.

## Repository Structure & How We Work

This repository is the workspace for the group. The structure is designed to keep our research, analyses, and resources transparent and accessible.

### docs/

This is where we conduct most of our work.
- Each metadata attribute we study will have:
- A CSV summary: showing which package managers provide that attribute and how.
- A text-based analysis: describing what the attribute means, how different package managers approach it, what transformations are required to read/use the data, and quality considerations.
- A recommendations file: guidance for how a new package manager could implement this attribute.
- Work-in-progress analyses will be stored in the `docs/drafts/` subdirectory until they mature.
This allows ongoing discussion and visibility into how ideas evolve.

### File Naming Convention

To ensure clarity and consistency, we use explicit filenames for each attribute:
- CSV Summary
  - `<attribute>-summary.csv`
- Purpose: Compact overview of which package managers provide this attribute and in what form.
- Textual Analysis
  - `<attribute>-analysis.md`
  - Purpose: Detailed explanation of the attribute, what it means, how it is represented across package managers, what transformations may be required, and any quality considerations.
- Recommendations / Guidance
  - `<attribute>-recommendations.md`
  - Purpose: Guidance for how a new package manager (or one evolving its metadata) could best support this attribute.

Example:
- `license-summary.csv`
- `license-analysis.md`
- `license-recommendations.md`

This convention ensures that new contributors can easily understand and extend our work without needing to consult additional documentation.

### Resources

This directory contains curated references we use during our analyses, such as:
- Links to academic papers.
- Links to package manager documentation.
- Repositories with relevant examples.
- Other working group or community discussions.

All references will be collected in a single file:
- `resources.md`

## How to Contribute

We welcome contributions from anyone interested in package metadata.
- Join our discussions via the [CHAOSS community](https://chaoss.community/participate/).
- Open issues or pull requests to suggest attributes, resources, or corrections.
- Participate in reviewing draft analyses in docs/drafts/.