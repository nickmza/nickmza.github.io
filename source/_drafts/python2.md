---
title: Adventures in Python- Mars Rover 2
tags:
    python
excerpt:
    Previously I got the basic Mars Rover Kata working in Python. Today I'll be adding the next iteration of requirements.
---

    Previously I got the basic Mars Rover Kata working in Python. Today I'll be adding the next iteration of requirements. Specifically:
    - Implement wrapping at edges. But be careful, planets are spheres.
    - Implement obstacle detection before each move to a new square. - If a given sequence of commands encounters an obstacle, the rover moves up to the last possible point, aborts the sequence and reports the obstacle.

    Let's get started....

# Collision Detection
I wanted to make this a little more interesting so I'm altering the collision requirement