# Avoid Dependency Crash

like this [issue](https://github.com/mozilla/hubs/issues/5472) and this [issue](https://github.com/mozilla/hubs/issues/5471)

Hubs build with lot of dependencies. And it keep update.

I using github action runner that run `npm install` when i commit to `master` branch

so if the dependencies is updating it version (using this symbol `^`) it sometimes resulting crash

### First Lets Find Posibility Changes That Makes Crash:

1. Package keep update because this symbol `^`

```
"core-js": "^3.6.5",
"dashjs": "^3.1.0",
"deepmerge": "^2.1.1",
"detect-browser": "^3.0.1",
"downshift": "^6.0.5",
"draft-js": "^0.10.5",
```

2. Package keep updating because point to the branch

```
 "ammo.js": "github:mozillareality/ammo.js#hubs/master",
 "animejs": "github:mozillareality/anime#hubs/master",
```

3. NPM version (rare)
use the LTS version instead.

4. Cache (rare)

5. Environtment library change (rare)


### Solution: 
- Lock the version of package by commit hash

```
 "aframe": "github:mozillareality/aframe.git#033ec6571ff6ec2c9162e26ff4e23ee2e65afc12",
```

- Lock the version. Remove the `^` symbol
```
"core-js": "3.6.5",
"dashjs": "3.1.0",
"deepmerge": "2.1.1",
"detect-browser": "3.0.1",
"downshift": "6.0.5",
"draft-js": "0.10.5",
```

- Using yarn install instead of npm install

Sometime it work.