---
title: Are Polymarket Crypto 5m/15m Prices a Random Walk?
date: 2026-03-11 13:00:00 +0800
categories: [Weekly Research, Polymarket]
tags: [polymarket, microstructure, random-walk, markov-chain]
math: true
toc: true
---

## Motivation

This week I begin with a basic statistical question:

Are short-horizon Polymarket crypto contracts close to random walks?

This matters because if price increments are not fully memoryless, short-horizon state dependence may exist.

## Questions

1. Are 5m / 15m price increments serially independent?
2. Is upward/downward movement symmetric?
3. Can coarse state transitions help predict final resolution?

## Initial Direction

The first step is to test short-horizon autocorrelation, price-zone asymmetry, and transition frequencies.
