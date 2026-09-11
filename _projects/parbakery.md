---
layout: page
title: parbakery
description: performant, repeatable, deterministic autodocumentation for many-datasets ecosystem
img: assets/repos/parbake_final.png
importance: 2
category: work
github: https://github.com/mseryn/parbakery
giscus_comments: false
---

Companion project to [bakery](/projects/bakery/) — two parts of the same
system.

Tools for documenting directories of HPC data files. Point them at a
directory of CSVs and get back a plain-text description of what is in
each one.

For each CSV file, parbakery generates:

- a plain-text description
- a par-baked [Croissant](https://mlcommons.org/working-groups/data/croissant/) file
- Markdown documentation

This serves two purposes: assessing data quality by examining column
contents, and screening files before sharing to catch potentially
sensitive information — usernames or project names, say — that should
have been anonymized.
