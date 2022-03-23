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
