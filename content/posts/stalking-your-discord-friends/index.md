---
date: "2021-07-20"
title: "Stalking your Discord friends"
tags: ["old"]
ShowReadingTime: true
---

Because I love statistics I made a thing that makes graphs of when people are online on Discord and what they are doing. It sounds kinda creepy and in fact it is (totally inspired by [this](https://mango.pdf.zone/graphing-when-your-facebook-friends-are-awake)).

## Discord status

Everyone knows these little colorful dots, they exist in almost every chat application. But how exactly does it work on Discord?

To find out, there is only one solution, to go deep into Discord's code. And to do that, just press <kbd>F12</kbd> or <kbd><kbd>CTRL</kbd>+<kbd>SHIFT</kbd>+<kbd>I</kbd></kbd> in a browser or in the Discord client (since it uses [Electron](https://www.electronjs.org/)).

## I'm in

![empty devtools, nothing to see here](img/empty_devtools.png#center)

Ok now do <kbd><kbd>CTRL</kbd>+<kbd>R</kbd></kbd> to reload the thing and congrats the network tab is already full. You might be wondering, what are we looking for in this mess. I spare you the details, Discord uses something called [Websocket](https://en.wikipedia.org/wiki/WebSocket) for that sort of things. To see what the client and the server are talking about, we need to find the request for the Websocket. Rest assured, it is easy to find.

You can find it with:

- The [HTTP status code 101](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/101), typically used when a websocket connection is established.
- The URL `gateway.discord.gg`
- The initiator `websocket`

## We have been bamboozled

![websocket gibberish](img/bamboozled.png#center)

We can read the content of what we send but not what we receive. These two guys speak some weird computer language! I can't use that for my beautiful graphs...

In fact, I decided later to decode the Discord desktop client version of these messages to copy the payload to act just like the regular client (except that I don't encode my payload in <abbr title="External Term Format">ETF</abbr> format). But this is an another story, let's focus again.

## Building a client from scratch

To see what on earth the server is sending to the client, I built a JavaScript websocket client with the [WS library](https://www.npmjs.com/package/ws), with the help of my friend [JunkMeal](https://github.com/JunkMeal) and with the help of the [Discord documentation](https://discord.com/developers/docs/topics/gateway). Once again, I spare you the details of coding the whole thing. Let's focus on the main topic: Spying our friends.

How can we see whenever or not a friend is online and what does he do? We learn [here](https://discord.com/developers/docs/events/gateway-events#update-presence) that the `PRESENCE_UPDATE` event is sent to the client when a user modifies his status/presence.

The thing is that I want to spy on only my friends and not the 35 624 187 users in the same servers as me.

To only spy on my friends, I first need my friends list. If you take a look [here](https://discord.com/developers/docs/events/gateway-events#ready), you can see that Discord sends it to the client in the `READY` event. I just need to make a map with everyone from my friends list:

```js
...
// switch case on the gateway events

case "READY": // when the client receives the "READY" event
    this.sessionID = data.d.session_id
    this.relationships = new Map() // create a map of relationships
    for (var u in data.d.relationships) { // for every user in the friends list
        // set a new pair with user id as the key and user object as the value
        this.relationships.set(data.d.relationships[u].id, data.d.relationships[u].user)
    }
    break
...
```

Then, I can check if the user who modified his status/presence is in my friends list:

```js
...
case "PRESENCE_UPDATE": // when the client receives the "PRESENCE_UPDATE" event
    if (this.relationships.has(data.d.user.id)) { // if the user is in my friends list
        ...
        logsArray.push({ // push to the list of logs
            time: data.d.last_modified,
            status: data.d.status,
            platforms: Object.keys(data.d.client_status),
            activities: activities
        })
        ...
    }
    break
...
```

It seems to be working, nice. This is an example of a log of an user, it contains the timestamp (time in seconds since the 1st January 1970) of the new presence, his status (online, idle, dnd, offline), the platforms on which he is connected (desktop, mobile, web) and his activities (playing, listening, ...).

```js
[
    {
        time: 1627328053420,
        status: "online",
        platforms: ["desktop"],
        activities: ["Playing", "Listening"],
    },
];
```

Just add to that a piece of code to save them to a local database and we're good to go. Now it's time to aggregate data, and for that I'll let the client run for a while.

## Graphs time

Now that we have these ~~creepy~~ beautiful data, it is time to make some good looking graphs. For that I'm using a library called [chart.js](https://www.chartjs.org/).

It took me about 5 times longer to make these graphics than it took me to write the websocket client code. I had strange problems like when it's 23:00 and the graph shows only 75% of the data... In fact, I messed up the variables names but it took me 3 hours to realize it :sweat_smile:.

## Stop saying nonsense, show me good looking graphs

Remember that you asked for it, here they are:

![line graph](img/graph_line_status.png#center)

The x-axis is time, and the y-axis is the user status. Status for an user are "online", "dnd", "idle" and "offline". Each coloured line is a different platform. It can be Desktop, Mobile or Web.

## Can I prevent you from doing that to me?

Just unfriend me (please don't, I will be alone without friends).

## Conclusion

With these data and graphics, I can surely guess when you wake up and when you go to bed. Perhaps even when you are at home (desktop) and when you are outside (mobile). Yes it is scary, but keep in mind that Discord and all other companies from who you get an online service for free probably do the same thing.
