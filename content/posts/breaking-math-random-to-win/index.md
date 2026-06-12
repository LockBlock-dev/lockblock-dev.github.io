---
date: "2025-03-03"
title: "Breaking Math.Random to win"
tags: ["JavaScript"]
ShowReadingTime: true
---

I was talking with [MasterIO](https://github.com/MasterIO02) about Yuna, the Discord bot of [o!rdr](https://ordr.issou.best/), when I said that I could win big money.

Big money in this story means virtual currency from a Discord bot. Sadly I did not buy a house with JavaScript.

He told me to "win against `Math.random`".

So I did what any reasonable person would do. I googled how to break `Math.random`.

## JavaScript moment

`Math.random()` is not magic. It does not call the universe and ask it politely for a number. It is pseudo-random, which means that it uses an internal state to generate numbers that look random enough for most things.

The important part is "most things".

If you use it to choose a random color, it's fine. If you use it for an economy system where people can gamble all their credits, perhaps it is time to sit down for a minute.

I remembered watching a video about predicting V8's `Math.random()` a while ago, but I forgot the details. After some googling I found the repository of said video: [PwnFunction/v8-randomness-predictor](https://github.com/PwnFunction/v8-randomness-predictor), which does the hard work with Z3 and some `xorshift128+` witchcraft.

So, credit for the actual prediction tool goes to PwnFunction. I was mostly the guy holding a Discord command and asking "what if".

## A very generous roll command

Yuna has a command called `y!roll`. Like many roll commands, you could ask it to roll a random number.

The small problem was that there was no useful limit.

This meant I could ask Yuna to roll with a very big maximum value. Since the bot probably did something like this:

```js
Math.floor(Math.random() * max);
```

the result leaked a lot of digits from the original random float.

With a small `max`, the result is too imprecise to be useful. With a huge `max`, the roll result keeps much more precision from the original `Math.random()` value. And precision is exactly what the predictor needs.

## Computer science happened here

The plan became:

- roll a very big number
- recover a part of the original random float
- do it a few times
- predict the next `Math.random()` values

In practice, I used `y!roll` enough times to recover useful outputs from `Math.random()`. Then I converted the roll results back into floats close enough for the predictor.

The predictor takes a few outputs, gives them to Z3, and Z3 solves the internal state of the generator. This is the part where I will spare you the ritual because it mostly looks like math symbols bullying JavaScript until JavaScript confesses.

After around 5 values, I could predict the next random numbers.

Nice.

## Gambling time

The next step was to connect this to `y!bet`.

I did not have the bot source code in front of me, but the behavior was easy to guess. The command probably used `Math.random()` and compared it to a threshold, something like:

```js
if (Math.random() > 0.5) {
    win();
} else {
    lose();
}
```

So if I could predict the next `Math.random()` value, I could know if the next bet would win or lose.

If the next number was bad, I just called `y!roll` again to advance the random generator. If the next number was good, I could bet. If the next number was very close to 1, then it was time to be financially irresponsible with fake money.

I first tested the prediction with a roll. It matched.

Then I tested with a small gamble and lost something like 20 credits on purpose. It also matched.

Then I went almost all in.

And won.

## The real enemy

There was one funny problem. The predictor only works if I know exactly how many times `Math.random()` was called.

So if someone, in a random Discord server, typed `y!roll` between my prediction and my bet, the whole thing could become desynchronized. Same thing if someone used another command that called `Math.random()`.

The attack was not only against pseudo-randomness. It was also against people being bored on Discord.

Imagine losing everything because someone somewhere rolled a dice at the wrong time. Beautiful.

## The fix

The fix was very simple:

Cap `y!roll` to `1,000,000`.

With a small enough maximum value, the command does not leak enough precision from the raw `Math.random()` output to recover the internal state.

Of course, the better lesson is to not use `Math.random()` for anything that behaves like gambling or money, even if the money is fake. Use a better randomness source, don't expose raw-ish random outputs, and don't let every command share one predictable stream of future sadness.

## tl;dr

- `Math.random()` is not secure randomness
- Unlimited `y!roll` leaked too much information
- V8 randomness can be predicted if you collect enough outputs
- [PwnFunction/v8-randomness-predictor](https://github.com/PwnFunction/v8-randomness-predictor) did the solver magic
- I used rolls to know when `y!bet` would win
- I won fake money
- MasterIO fixed it by capping `y!roll`
