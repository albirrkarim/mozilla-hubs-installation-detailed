## Introduction

After [installing Mozilla hubs on local](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/README.md) and [Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md) I started to modify it.

## Where is the database schema?

[see this first](https://www.youtube.com/watch?v=X9AggnaEXrM)

##  How to make hubs scene with blender

[make scene](https://www.youtube.com/watch?v=ldHwbnMMKVY)


## Reduce logs info for webpack server based

Many logs message on hubs, hubs admin, spoke terminal sometimes makes us more confused. We can reduce it by adding this on `devServer` object on `webpack.config.js`

```
devServer: {
  // This is for reducing logs output on terminal
  quiet: false,
  noInfo: false,
  stats: {
    assets: false,
    children: false,
    chunks: false,
    chunkModules: false,
    colors: true,
    entrypoints: false,
    hash: false,
    modules: false,
    timings: false,
    version: false
  }
}
```

##  How to login & create admin account?

[see](https://github.com/mozilla/reticulum#5-logging-in)

## What you want to know ?

Write it on the discussion. maybe i can help you.

<br>
<br>

## Also read:

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The Problem I Still Faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Basic Information for Modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)