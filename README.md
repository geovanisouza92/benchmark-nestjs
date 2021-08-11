# benchmark-nestjs

The objective here was to create a simple comparison between Express and Fastify platform on a [Nestjs](https://nestjs.com/) application.

On the documentation, it's said:

> [...] fastify is much faster than Express, achieving almost two times better benchmarks results.

So I'd like to confirm that findings.

This repository is a standard project with a simple route `GET /` that returns the message `Hello World!`.

## Running the server

To start the Express server we use:

```sh
nest start
```

To start the Fastify server we use:

```sh
nest start --config=nest-cli.fastify.json
```

That uses [another entrypoint](./src/main.fastify.ts).

[Apache benchmark](https://httpd.apache.org/docs/2.4/programs/ab.html) was used to simulate parallel requests and measure de perceived performance of the server with this command:

```sh
ab -k -c 200 -n 20000 http://localhost:3000/
```

## Profiling with Node

To understand the performance of the Node process, I used an additional option on the command line:

```sh
nest start -e 'node --prof --no-logfile-per-isolate'
```

for Express and:

```sh
nest start --config=nest-cli.fastify.json -e 'node --prof --no-logfile-per-isolate'
```

for Fastify. Each execution created a `v8.log` file that I processed with:

```sh
node --prof-process v8.log > express/fastify.txt
```

For more information about this, read [this article](https://nodejs.org/en/docs/guides/simple-profiling/).

## Results

These are the results on a _Intel(R) Core(TM) i7-7500U CPU @ 2.70GHz_:

| Measure (unit)                 | Express |  Fastify |     Δ |
|--------------------------------|--------:|---------:|------:|
| Time taken for tests (seconds) |   4.544 |    1.676 |  -63% |
| Total transferred (bytes)      | 4780000 |  3520000 |  -26% |
| Requests per second (#/sec)    | 4401.73 | 11935.64 | +171% |
| Time per request (ms)          |  45.437 |   16.757 |  -63% |
| Time per request (all) (ms)    |   0.227 |    0.084 |  -63% |
| Transfer rate (Kbytes/second)  | 1027.36 |  2051.44 |  +99% |

The profiling revealed that Express has some hotspots specially with dynamic functions compilation and RegExp usage (or abuse).
