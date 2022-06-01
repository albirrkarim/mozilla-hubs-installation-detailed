# Advice

In here i give you advices based on my experience.

# Avoid Dependency Crash

like this [issue](https://github.com/mozilla/hubs/issues/5472) and this [issue](https://github.com/mozilla/hubs/issues/5471)

Hubs build with lot of dependencies. And it keep update.

I using github action runner that run `npm install` when i commit to `master` branch. Problem like this is dangerous on production phase it will make the entire app is crash.

## First Lets Find Posibility Changes That Makes Crash:

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

5. Environment library change (rare)


## Solution: 
### - Lock the version of package by commit hash

```
 "aframe": "github:mozillareality/aframe.git#033ec6571ff6ec2c9162e26ff4e23ee2e65afc12",
```

### - Lock the version. Remove the `^` symbol
```
"core-js": "3.6.5",
"dashjs": "3.1.0",
"deepmerge": "2.1.1",
"detect-browser": "3.0.1",
"downshift": "6.0.5",
"draft-js": "0.10.5",
```

### - Using yarn install instead of npm install

Sometime it work.

## How if We Need Update?

If you see an update in hubs like adding feature or something. you can clone it first and check it first. make sure it work. then compare the code manually. please correct me if im doing wrong way.

## Always Backup

Dependencies crash is really dangerous, so always provide backup.Backup this file and folder to 

`/home/admin/hubs-action-runners/<hubs or spoke or dialog>/backup`

```
node_modules
package-lock.json
package.json
```

<br>
<br>

[Paypal](https://paypal.me/AlbirrKarim)

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>


