---
title: Troubleshooting Professional Media over IP Networks
author: Michael S. Liebman, CEV, CBNT
date: April 12, 2024
bibliography: troubleshooting-moip.bib
csl: ieee-with-url
link-citations: true
link-bibliography: true
---

### Topics

* Troubleshooting Process
* Tools
  * Hardware
  * Software
* Techniques & Tips

## Troubleshooting Process

### It's not that different

from traditional media systems!

### Troubleshooting Methods

:::::::::::::: {.columns}
::: {.column width="50%"}

#### CompTIA [@comptiaGuideNetworkTroubleshooting]

1. Identify the problem.
2. Develop a theory.
3. Test the theory.
4. Create an action plan.
5. Implement the solution.
6. Document the issue.

:::
::: {.column width="50%"}

#### DECSAR Method [@rossTeachingStructuredTroubleshooting2009]

1. Define the problem.
2. Examine the situation.
3. Consider the causes.
4. Consider the solution.
5. Act and test.
6. Review troubleshooting.

:::
::::::::::::::

## Hardware Tools

### Cable Testers

:::::::::::::: {.columns}
::: {.column width="50%"}

![Klein Tools Cable Tester](images/tools/hardware/continuity.jpg)

:heavy_check_mark: Continuity

:heavy_check_mark: Pinout

:x: Connectivity

:x: Crosstalk, attenuation, interference...

:::
::: {.column width="50%"}

![Fluke Networks LinkIQ+](images/tools/hardware/better-continuity.webp)

:heavy_check_mark: Continuity

:heavy_check_mark: Pinout

:heavy_check_mark: Connectivity

:x: Crosstalk, attenuation, interference...

:::
::::::::::::::

::: notes

This style of tester is nothing more than a glorified continuity tester.
It will tell you if your pinout is correct.
But it can't tell you if the cable will *work*.

:::

## References

::: {#refs}
:::
