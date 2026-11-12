# Pick seals

This page holds one code for each pick we post, and public timestamps on the list. Nothing else is here: no picks, no prices, no results.

## What is here

- `seals.json` is the list. Each entry has a number (`n`), a code (`hash`), the time it was sealed (`sealed_at`) and a `link`.
- `proofs/` holds public timestamps on the last link of the list. New ones are added each time the list grows.

## What a seal is

When a pick is posted, its terms are written on one line: the day, the kind of pick, the game and bet, the price, the book, the bet
size, the chance we stated, the title, and a random secret. That line is run through SHA-256. The result is the code in this list.

The code tells you nothing about the pick. When the pick opens to everyone, the app shows the full line. Hash that line yourself and
you get the code that was already here. If one letter of the pick had been changed, the code would not match.

The first 56 seals are an older kind. Their line stops at the bet size, so they do not cover the chance or the title.

## What the link is

Each link ties a seal to every seal before it:

    link_n = sha256( link_(n-1) + hash_n + sealed_at_n )

All three parts are joined as text. The first link uses an empty previous link. Change, remove or reorder one seal and every link
after it changes.

## What the timestamps prove

Each file in `proofs/` is an [OpenTimestamps](https://opentimestamps.org) proof. Its name starts with a seal number and the first 16
characters of that seal's link. What it commits to is that link. To check one with the standard tool:

    ots verify -d <the full link from seals.json> proofs/<file>.ots

It shows that the link, and so every seal up to it, existed by the time the proof gives. That clock is not ours. A new proof is
"pending" for a few hours, until its Bitcoin block is in. The tool completes it by itself.

One proof is kept per calendar server, so no proof depends on any one of them.

## What this does not prove

- That a pick won. Results are checked against final scores, not here.
- That a seal was made before its game, for the seals made before the first proof. The first proof was made on 2026-11-11.
  From then on, each new seal is covered by a proof made within minutes of it.
