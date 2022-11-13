+++
title = "VLOG[0]: Why are we building a new database?"
date = 2022-11-13
draft = true
[extra]
author = "Frank Hampus Weslien"
+++

**Why are we building a new database?**
The simple answer is that there doesn't exist a database that supports building
the type of applications that we want to build. We want to build local-first web
apps that can sync. Ideally, we manage to do it p2p and then we won't need to pay cloud bills.

The web has become more and more powerfull as application platform, though it
is still lagging behind native applications. You have support for:

- [Notifications](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)
- [Structured data storage via indexeddb](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Bluetooth](https://developer.mozilla.org/en-US/docs/Web/API/Bluetooth)
- [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API)
- [p2p connections via WebRTC](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API)

... and a _lot more_! You can even install web apps and have their icon on your home
screen across both mobile, and desktop by creating a progressive web app (PWA).

For a lot of applications the web has a lot to offer. Especially considering the
web's two fundmental advantages: reach (you can link to a runnable application), and avoiding the apple/playstore tax.
We believe that today you could write 10x better versions of a lot of applications
by going web first.

Imagine a todo-app which you can reach via a url. No login required.
It can be installed and added to your homescreen.
If you want to sync tasks between your phone and laptop you scan a QR code.
There are a lot of apps that are hardly more complex then a ToDo application
where this design would work brilliantly.

How does this vision translate into requirements for a database?
Well, we need it to work in the browser.
As soon as a user adds a new entry we want to be able to update all queries
so that the UI reflects the latest state.
It also needs to be able to sync without merge conflicts.

We will write the database in Typescript, it gives us easy access to all browser APIs.
If the need arises one could also compile Rust to webassembly to make the database faster.

Merging is hard so we will use [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) to make it easy.
The basic idea is that instead of having an algorithm decide how to merge you instead use a datastructure that
naturally supports syncing. Sets are a basic example of such a datastructure; you can always join two sets without there being conflicts.
In our case we will use a [tripplestore](https://en.wikipedia.org/wiki/Triplestore) combined with timestamps and resolve using last-write-wins.

The query engine will be using datalog. It is powerfull, simple, and supports
reactive queries.

---

The [MirrorDB repository](https://github.com/Mirror-Labs/MirrorDB) have had its
first few commits! If you are reading this in the future look for the git tag "vlog-0".
