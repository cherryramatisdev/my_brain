---
title: A single query language to rule them all! How to explore multiple data types
description: Has anyone being in that situation as a developer where you need to provide some piece of information about your system or even a more involved analysis around a particular data set to a marketing or business person? Most devs working for start-ups and agencies have been on that shoe before and when that happens it's really important to know some tools that quickly help doing that. In this article we'll dive in more on these possibilities.
tags: databases,esproc,opensource
cover_image: 
canonical_url: 
published: false
---

Has anyone being in that situation as a developer where you need to provide some piece of information about your system or even a more involved analysis around a particular data set to a marketing or business person? Most devs working for start-ups and agencies have been on that shoe before and when that happens it's really important to know some tools that quickly help doing that. In this article we'll dive in more on these possibilities.

## Table of contents

- [Why even consider having SQL everywhere?](#why-even-consider-having-sql-everywhere)
- [SQL like queries for every filetype](#sql-like-queries-for-every-filetype)
	- [Aggregating the songs by album](#sql-like-queries-for-every-filetype#aggregating-the-songs-by-album)
	- [Filtering by specific album](#sql-like-queries-for-every-filetype#filtering-by-specific-album)
	- [Merging two different data sources](#sql-like-queries-for-every-filetype#merging-two-different-data-sources)
	- [Looking into more file types](#sql-like-queries-for-every-filetype#looking-into-more-file-types)
- [The built-in SPL language as an alternative to SQL query language](#the-built-in-spl-language-as-an-alternative-to-sql-query-language)
- [Conclusion](#conclusion)

## Why even consider having SQL everywhere?

We developers really need to start valuing query languages in general more than just when we're communicating with a database in a API, but also in a more broad view to look and analyse **any data source**. SQL is such an awesome and powerful language not only to get a first view into a set of data, but also performing complex aggregations, filtering and transformations on that same data.
Specially when considering [basics concepts of piping data in linux](https://dev.to/cherryramatis/linux-filters-how-to-streamline-text-like-a-boss-2dp4#what-is-a-pipeline) it's possible to view a whole new world of possibilities where it's possible to for example:

- 1. Make an HTTP request with `curl`
- 2. Pipe it to `jq` to get just the part of the JSON that you want to analyze
- 3. Pipe again to `esprocx` and perform special aggregations using *Simple SQL*

## SQL like queries for every filetype

We'll start slowly by exploring a text file with a couple of song-related information, and gradually delve into more interesting aspects of the language, shall we?

For example, assume the following `data.txt` file:

```txt
SONG                  ALBUM
Shooting Star         Midnight Chronicles
Another shooting star Midnight Chronicles
Lost in Paradise      Serenade of Memories
Rhythm of Love        Melodies of Summer
Whispering Wind       Twilight Serenade
Eternal Sunshine      Echoes of Dawn
Dancing in the Rain   Symphony of Dreams
```

Considering this data source, it's possible to collect some kind of information using some well known SQL queries:

### Aggregating the songs by album

```sql
$select SONG AS Song, count(*) AS Albums from data.txt group by ALBUM
```

### Filtering by specific album

```sql
$select SONG AS Song from data.txt where ALBUM = 'Midnight Chronicles'
```

### Merging two different data sources

For this specific example we'll consider another text file for the join to happen properly, the file will look like the following `artist.txt` file:

```txt
ALBUM                  ARTIST
Midnight Chronicles    Stellar Dreams
Serenade of Memories   Moonlight Serenade
Melodies of Summer     Summer Breeze
Twilight Serenade      Whispering Shadows
Echoes of Dawn         Dawn's Embrace
Symphony of Dreams     Harmonic Bliss
```

With both data sources, it's possible to join and show them as follows:

```sql
$select D.SONG as "Song Name", A.ARTIST as "Artist Name", D.ALBUM as "Album Name" 
from data.txt D 
join artist.txt A  on D.Album = A.ALBUM
```

### Looking into more file types

The reason why we're looking into `text` file is just from the *WOW* factor, but it's possible to really deep dive into more file type like CSV, JSON, XML, etc..

You can look more about it on the [Simple SQL](https://doc.scudata.com/esproc/tutorial/jiandansql.html) section of Esproc documentation.

## The built-in SPL language as an alternative to SQL query language

Although I find the Simple SQL feature really nice it's not the only way you can look into data using the Esproc platform. With [SPL](https://c.scudata.com/article/1634722432114) you get some neat and specific features such as: *Built-in Parallelism*, *Manual Multithreading*, *Integration with databases via ODJBC* while also being able to integrate Simple SQL within a instruction using a `query()` function. That way you don't need to lose anything by opt-in to use the SPL language!

Another cool feature of using SPL within the [Esproc IDE](https://doc.scudata.com/esproc/tutorial/jsqdaz.html) is viewing the interactive grid layout response while you're writing the queries and actions. For example:

Considering this simple query:

```spl
=file("Order_Books.txt").import@t()
=A1.id(SalesID)
=A1.id(PName)
```

We get a very visual result from it:

![[Pasted image 20240206185950.png]]

## Conclusion

When it comes to exploring data, I always aim to be efficient and accurate. However, there are times when the query becomes too complex, and that's when I turn to a Python script. It's remarkable to see the abundance of open-source concepts that aim to bring together data exploration. If you're interested in optimizing your data exploration process, these resources are definitely worth exploring! May the force be with you 🍒
