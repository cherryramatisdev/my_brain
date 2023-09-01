Have you ever encountered a problem and immediately had the solution pop into your mind without the need for debugging? If not, you're not alone. In this article, I'll introduce a method to provide real-time feedback on the functions you create as you work through a problem. After all, none of us are infallible geniuses who can always nail it first try like ThePrimeagen.

## But what is a REPL?

[![C# Interactive - REPL | Блог Черкашина Александра](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.cherkashin.dev%2Fassets%2Frepl%2Frepl-meme.png&f=1&nofb=1&ipt=a0b043d8684763fff7dd672daf45d42fc02a9c40a3600c7f75401649871675ac&ipo=images)](https://www.cherkashin.dev/assets/repl/repl-meme.png)

A REPL is simply an infinite loop that accept a command and eval it outputting the result of the operation. As a practical example you can run `irb` on your shell right now and try some ruby functions:

```sh
$ irb
irb(main):001> 1 + 1
=> 2
irb(main):002> raise 'Some error'
(irb):2:in `<main>': Some error (RuntimeError)
	from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/lib/ruby/gems/3.2.0/gems/irb-1.8.0/exe/irb:9:in `<top (required)>'
	from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/bin/irb:25:in `load'
	from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/bin/irb:25:in `<main>'
irb(main):003>
```

As you can see we see the output of simple operation stuff like `1 + 1` or even exception raising, quite cool right? This is an amazing feature because we can quickly evaluate complex operation to help us understand the algorithm as it's going.