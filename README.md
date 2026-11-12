# Pick seals

This page holds one code for each pick we post, one code for each reading we take of closing prices, and public timestamps on the
list. Nothing else is here: no picks, no prices, no results.

## What is here

- `seals.json` is the list. Each entry has a number (`n`), a code (`hash`), the time it was sealed (`sealed_at`) and a `link`.
- `proofs/` holds public timestamps on the last link of the list. New ones are added each time the list grows.

## What a seal is

When a pick is posted, its terms are written on one line: the day, the kind of pick, the game and bet, the price, the book, the bet
size, the chance we stated, the title, the other side's price, and a random secret. That line is run through SHA-256. The result is
the code in this list.

The code tells you nothing about the pick. When the pick opens to everyone, the app shows the full line. Hash that line yourself and
you get the code that was already here. If one letter of the pick had been changed, the code would not match.

Older seals cover less. The first 56 stop at the bet size, so they do not cover the chance or the title. Seals made before
2026-11-12 do not cover the other side's price.

Some seals are not picks. Shortly before a game we read the book's last prices for our picks, and we seal that reading the same way,
within minutes. Such a seal belongs to no pick. It shows the reading was not changed later. It does not show the reading was right:
those prices are our own readings.

## What the link is

Each link ties a seal to every seal before it:

    link_n = sha256( link_(n-1) + hash_n + sealed_at_n )

All three parts are joined as text. The first link uses an empty previous link. Change, remove or reorder one seal and every link
after it changes.

## What the timestamps prove

Each file in `proofs/` is an [OpenTimestamps](https://opentimestamps.org) proof. Its name starts with a seal number and the first 16
characters of that seal's link. What it commits to is that link. To check one with the standard tool:

    ots --no-bitcoin verify -d <the full link from seals.json> proofs/<file>.ots

The tool reads the proof and prints a Bitcoin block number and that block's merkle root. Look the block up on any block explorer.
The root must match, and the block's time is the proof's time. (Without `--no-bitcoin` the tool wants a Bitcoin node on your own
computer.)

It shows that the link, and so every seal up to it, existed by that block's time. That clock is not ours. A block's time is set by
its miner; the network's rules hold it to about two hours, and it is usually right to minutes.

A new proof is "pending" for a few hours, until its Bitcoin block is in. We then store the finished proof here, so the check does
not lean on the calendar server. One proof is kept per calendar server, so no proof depends on any one of them.

## What this does not prove

- That a pick won. Results are checked against final scores, not here.
- That a seal was made before its game, for the seals made before the first proof. The first proof was made on 2026-11-11.
  From then on, each new seal is covered by a proof made within minutes of it.
- That a closing price we read was the true one. A seal fixes what we wrote down, not what the book showed.
