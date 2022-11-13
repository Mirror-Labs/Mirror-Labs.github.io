+++
title = "VLOG[0]: Why are we building MirrorDB?"
date = 2022-11-13
draft = false
[extra]
author = "Frank Hampus Weslien"
+++

**Why are we building MirrorDB?**
We want to build local-first web apps with p2p syncing and there doesn't exist a database that allow us to build
them.

The web has become more and more powerful as an application platform. You have support for:

- [Notifications](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)
- [Structured data storage via indexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Bluetooth](https://developer.mozilla.org/en-US/docs/Web/API/Bluetooth)
- [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API)
- [p2p connections via WebRTC](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API)
- [... and a lot more](https://developer.mozilla.org/en-US/docs/Web/API)

You can even install web apps and add them to your home
screen across both mobile, and desktop by creating a progressive web app (PWA).

The web has a lot to offer. Especially considering the
web's two fundamental advantages: reach (you can link to a runnable application), and avoid the apple/play store tax.
We believe that today you could write 10x better versions of a lot of applications
by going web native.

Imagine apps which you can reach via a url. No login required.
It can be installed and added to your homescreen.
If you want to pair your phone and laptop you scan a QR code and it will automatically start syncing.
There are a lot of apps where this design would work brilliantly and with applications in mind we can start sketching the requirements for our database.

As soon as a user adds a new entry we want to be able to update all queries
so that the UI always reflects the latest state.
We need a query language with a focus on joins; it is very typical for the UI to want data
about multiple related entities.
A user also expects to be able to make changes on their phone, turn it off, keep using the app on their laptop and still in the end be able to merge the state on the phone and laptop without conflicts.
Ideally, the merging-layer is protocol agnostic and works across: files, bluetooth, usb, webrtc, websocket, and etc.

There are a couple of technologies/ideas that we believe will work great in tandem to realise this vision.

**IndexedDB** is a database that exists in your browser and allows you to store structured data and create indexes on that data.
Obviously you can not create a local-first web app without it.

**Typescript/Rust** will be used to build the database. Typescript adds extra safety
compared to Javascript while still giving us access to all browser API:s. Rust
is interesting because of its great webassembly support. Databases are performance critical
and webassembly might greatly speed it up.

**CRDTs**, conflict-free replicated data types, are a type of data structure with
merging built in. The idea is that instead of writing complex algorithms for merging data we push it
into the very semantics of the data structure itself and get merging for free.
Sets are a basic example of such a data structure; you can always join two sets without there being conflicts.

**Datalog** is a simple but powerful language to write logical programs over a set of facts.
Usually the facts are separated into different tables like this:

_Person Table_

| ID  | Name                | Born         |
| :-: | :------------------ | :----------- |
| 100 | "James Cameron"     | "1954-08-16" |
| 200 | "Quentin Tarantino" | "1963-03-27" |
| ... | ...                 | ...          |

_Movie Table_

| ID  | Title            | Release Date | Director |
| :-: | :--------------- | :----------- | :------- |
| 300 | "The Terminator" | "1984-10-26" | 100      |
| 400 | "Pulp Fiction"   | "1994-05-21" | 200      |
| 500 | "Aliens"         | "1986-07-18" | 100      |
| ... | ...              | ...          | ...      |

In Datalog you take existing tables and create new ones, here I look for all the movies each director has made by combining the information from two tables.

```datalog

  DirectorToMovie(dir_name, movie_title) :-
    Person(dir_id, dir_name, _),
    Movie(_, movie_title, _, dir_id)

```

[Look here for a introduction to datalog.](https://x775.net/2019/03/18/Introduction-to-Datalog.html)

**Triple Stores** are nice because they are so simple (and easy to make into a CRDT). You can think of it as your
database only consisting of one table with three columns "id", "attribute", and "value".
Even though it is so simple you can model any kind of data. Infact, let's try to merge the Person and Movie tables above into one.

Example:

| ID  | Attribute            | Value               |
| :-: | :------------------- | :------------------ |
| 100 | "person/name"        | "James Cameron"     |
| 100 | "person/born"        | "1954-08-16"        |
| 200 | "person/name"        | "Quentin Tarantino" |
| 200 | "person/born"        | "1963-03-27"        |
| 300 | "movie/title"        | "The Terminator"    |
| 300 | "movie/director"     | 100                 |
| 300 | "movie/release-date" | "1984-10-26"        |
| 400 | "movie/title"        | "Pulp Fiction"      |
| 400 | "movie/director"     | 200                 |
| 400 | "movie/release-date" | "1994-05-21"        |
| 500 | "movie/title"        | "Aliens"            |
| 500 | "movie/director"     | 100                 |
| 500 | "movie/release-date" | "1986-07-18"        |
| ... | ...                  | ...                 |

Notice how there is a lot of replicated information. This is actually useful because it makes it very simple
to update individual attributes for an entity, especially in a distributed system such as ours.
If we add timestamps to all triples we can resolve merges by always picking the latest one.[^1]

We feel very hopeful that we will be able to solve our problem by combining all these technologies/ideas.
Now we just need to do the difficult thing of building it!

---

[^1] Note that we shouldn't be using normal timestamps but instead use something like a [Hybrid Logical Clock](https://medium.com/geekculture/all-things-clock-time-and-order-in-distributed-systems-hybrid-logical-clock-in-depth-7c645eb03682).
