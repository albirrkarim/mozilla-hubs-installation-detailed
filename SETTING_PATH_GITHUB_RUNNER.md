Thanks to [this](https://github.com/actions/setup-node/issues/182#issuecomment-718233039)

GitHub Runners creates a file called **.path** in the same directory where you have **config.sh**. It contains the content of `$PATH` variable, which is further used during the execution of your workflows.
 
You have to add a directory with the yarn binary.

It will solve:

- yarn not found in github action runner
- mix command not found in github action runner
- elixir command not found in github action runner

Let me give and example for `yarn` for `mix` or `elixir` you can give a try by yourself.
 
### Find out where is the yarn
First on your server run:

```
which yarn
```

then you got the yarn location, like this
```
/home/admin/.nvm/versions/node/v16.14.0/bin/yarn
```

copy the path (not including the yarn)
```
/home/admin/.nvm/versions/node/v16.14.0/bin
```

### Edit path
run 
```
nano .path
```

you got something like

`aaaaa:bbbb:cccc`

separated with `:`

then paste the `/home/admin/.nvm/versions/node/v16.14.0/bin`

so the `.path` become

`/home/admin/.nvm/versions/node/v16.14.0/bin:aaaaa:bbbb:cccc`

**Don't add the enter or something. Just give paste and give the `:` next to it**

### Restart runner 
You need restart it. so it will load the new path

Mostly you are running with `svc.sh` then stop it
```
sudo ./svc.sh stop
```
Run it again
```
sudo ./svc.sh start
```

<br>
<br>



<a href='https://paypal.me/AlbirrKarim' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://user-images.githubusercontent.com/29292018/186840848-65e25ff9-47e2-424b-bfa0-4ca5d027b346.png' border='0' alt='Donate via paypal' /></a>

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

## Also read:

[List Hubs Custom Features](https://github.com/albirrkarim/mozilla-hubs-custom-features)

[Hubs Memory Efficiency & Usage Simulation (Private Repo)](https://github.com/albirrkarim/mozilla-hubs-optimization)

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The Problem I Still Faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Tips for Modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

[Experience Sharing About Hosting on Other Server](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/EXPERIENCE.md)