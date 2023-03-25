# The problem I faced and I already solved

## Remember! If you don't get a problem like me you don't have to follow this

The origin hubs repo is always updating and we are running on different server so it comes with different kind of error.

Sometimes the error I face doesn't necessarily mean you also face it.

## - 502 server communication error in hubs admin like this [issue](https://github.com/mozilla/hubs/issues/4970#issue-1087523703)

<details>

Ok, let's give try to solve this.

The problem is an API call to this route

`/api/ita/admin-info`

and other with slash ita `/ita`

if we check where is the code in the reticulum which handles that
open the `router.ex` ita API call the `RetWeb.Plugs.ItaProxy` if you in vs code you can command+click / ctrl+click for find the function is called.

```elixir

scope "/api/postgrest" do
   pipe_through([:secure_headers, :auth_required, :admin_required, :proxy_api])
   forward("/", RetWeb.Plugs.PostgrestProxy)
end

```

### What is port 3000?

Thanks to the contributor that give me a clue about the port 3000

its a [PostgREST](https://postgrest.org/en/stable/index.html)

Thanks to daniel, Give me info about [Running PostgREST locally](https://github.com/mozilla/hubs-ops/wiki/Running-PostgREST-locally)

![hubs admin panel work](/docs_img/admin_panel_work.png)

</details>

## - Architecture Kit Failed to Load and Import

<details>
This problem is related to number 1 above. We don't need `CORS_PROXY_SERVER` so set it with an empty string

![env spoke](/docs_img/env_spoke.png)

and make condition like this picture bellow

![env spoke](/docs_img/env_spoke_1.png)
</details>

## - Spoke Assets Thumbnail not Showing on Production

<details>
First way:

Edit Api.js to bypass thumbnail resize service.

![env spoke](/docs_img/spoke_failed_3.png)

Second way:

In reticulum `config/dev.exs` append `https://nearspark-dev.reticulum.io` to `asset_hosts` if you won't self-host nearspark, that for csp rules setting.

```elixir

asset_hosts =
  "https://localhost:4000 https://localhost:8080 " <>
    "https://#{host}:4000 https://#{host}:8080 https://#{host}:3000 https://#{host}:8989 https://#{host}:9090 https://#{cors_proxy_host}:4000 " <>
    "https://assets-prod.reticulum.io https://asset-bundles-dev.reticulum.io https://asset-bundles-prod.reticulum.io https://hubs.local:5000 " <>
    "https://nearspark-dev.reticulum.io"
```

or you can directly edit `lib/ret_web/plugs/add_csp.ex`.

Self-hosted Nearspark(thumbnail server):

Nearspark [resizes preview images to 200x200 thumbnails](https://github.com/mozilla/Spoke/blob/9fe7af7e0b4eab5908d1ada17c06aab223c978ce/src/api/Api.js#L430) for some media source like Sketchfab.

The original NearSpark repository (https://github.com/MozillaReality/nearspark) is not compatible with Node.js version 16 (node-gyp problem) and has a bug related to base64 URL decode. To address these issues, Use a fork of the repository and added support for the `certs/` directory for HTTPS settings. You can find fork at https://github.com/XuHaoJun/nearspark

Clone, copy `certs/`, install dependencies, and start.

```shell

git clone https://github.com/XuHaoJun/nearspark.git
cp hubs/certs nearspark
cd nearspark
npm ci
node app.js
```

set environment variable `THUMBNAIL_SERVER="localhost:5000"` in Spoke.

</details>

## - Spoke Console Error prefetch-src

<details>
I got error like this

![Spoke console error](/docs_img/spoke_console_error.png)

Then add rule `prefetch-src` like this.

![Spoke console fix](/docs_img/spoke_console_fix.png)

</details>

## - Upload Assets too Large on Spoke Production

<details>
It because nginx

Open up the nginx configuration file

```
sudo nano /etc/nginx/nginx.conf
```

add this `client_max_body_size` to the http section

```
http {

    # other line...

    client_max_body_size 100M;
}
```

Then restart nginx

```
sudo systemctl restart nginx
```

for more detail see [this article](https://www.tecmint.com/limit-file-upload-size-in-nginx/)

</details>

## - Spoke Sketchfab tab is empty

<details>

To obtain a Sketchfab API key, go to (https://sketchfab.com/settings/password). Once you have obtained the API key,
you can set it in the `config/dev.exs` file of Reticulum as shown in the following:

```elixir

config :ret, Ret.MediaResolver,
  giphy_api_key: nil,
  deviantart_client_id: nil,
  deviantart_client_secret: nil,
  imgur_mashape_api_key: nil,
  imgur_client_id: nil,
  youtube_api_key: nil,
  sketchfab_api_key: "YOUR_API_KEY_HERE",
  ytdl_host: nil,
  photomnemonic_endpoint: "https://uvnsm9nzhe.execute-api.us-west-1.amazonaws.com/public"
```

If thumbnails not work, go to [Spoke Assets Thumbnail not Showing on Production](#spoke-assets-thumbnail-not-showing-on-production) section.

Sketchfab tab should work.

</details>

## - Error: listen EADDRNOTAVAIL: address not available

<details>
I install this project on server in china. alibaba elastic compute service

then i got that error when trying to run Spoke and hubs admin.

![address error](/docs_img/address_error.png)

just remove `process.env.HOST_IP ||` like picture above.

and leave `"0.0.0.0"`

thanks to [this](https://stackoverflow.com/questions/53955562/node-js-error-listen-eaddrnotavail-52-1122)

</details>

## - My Experience Installing On Alibaba Elastic Compute Service

[see](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/EXPERIENCE.md)

## - Dialog Error Keep Restarting on pm2

<details>
```
this._mediasoupRouter._transports.size
```

![dialog restart](/docs_img/dialog_restart.png)

</details>

## - MediaSoupError: port bind failed due to address not available

<details>

Change the prod command in `package.json`

it because `MEDIASOUP_LISTEN_IP`. set value with `0.0.0.0`

```
MEDIASOUP_LISTEN_IP=0.0.0.0 MEDIASOUP_ANNOUNCED_IP=123.xxx.xxx.xxx HTTPS_CERT_FULLCHAIN=/etc/letsencrypt/live/example.com/fullchain.pem HTTPS_CERT_PRIVKEY=/etc/letsencrypt/live/example.com/privkey.pem DOMAIN=example.com node index.js
```

</details>

## - 500 /api/v1/media when load image (reticulum)

#### An error occured during media resolution

<details>

I modify reticulum on `lib/ret_web/controllers/api/v1/media_controller.ex`

become this code

```elixir
# This is an error response that we have cached ourselves
  defp render_resolved_media_or_error(conn, {_status, :error}) do
    # DON'T THROW ERROR
    url = conn.params["media"]["url"]
    content_type = MIME.from_path(url)
    resolved_media = URI.parse(url) |> Ret.MediaResolver.resolved(%{expected_content_type: content_type})

    render_resolved_media(conn, resolved_media)
    # send_resp(conn, 500, "An error occured during media resolution")
  end
```

</details>

#### Forbidden

<details>

I modify reticulum on `/lib/ret/media_resolver.ex`

Become

```elixir
def resolve(%MediaResolverQuery{} = query, root_host) do
    # If we fall through all the known hosts above, we must validate the resolved ip for this host
    # to ensure that it is allowed.
    resolved_ip = HttpUtils.resolve_ip(query.url.host)
    case resolved_ip do
      nil ->
        :error

      resolved_ip ->
        if HttpUtils.internal_ip?(resolved_ip) do
          resolvedMedia = resolve_with_content_type(query)
          {:commit, resolvedMedia}

          # :forbidden
        else
          resolve_with_ytdl(query, root_host, query |> ytdl_format(root_host))
        end
    end
end
```

And add this code bellow it

```elixir
def resolve_with_content_type(%MediaResolverQuery{url: %URI{} = uri}) do
    content_type = MIME.from_path(uri.path)
    uri |> resolved(%{expected_content_type: content_type})
end
```

</details>

## - Postgrest Services not running or can't be restart

<details>
Im using apple silicone chip m1.

When running reticulum locally got this error.

```
[error] Postgrex.Protocol (#PID<0.651.0>) failed to connect: ** (DBConnection.ConnectionError) tcp connect (localhost:5432): connection refused - :econnrefused
```

i install postgres with brew, so we need to list all the brew services.

```
brew services list
```

then [see](https://stackoverflow.com/a/41804478)

For Apple Silicone Macs use `rm /opt/homebrew/var/postgres/postmaster.pid` as the brew folder is by default in a `/opt/homebrew` instead of `/usr/local`

then restart the postgres services with

```
brew services restart postgresql@14
```

</details>

<br>
<br>

<a href='https://paypal.me/AlbirrKarim' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://user-images.githubusercontent.com/29292018/186840848-65e25ff9-47e2-424b-bfa0-4ca5d027b346.png' border='0' alt='Donate via paypal' /></a>

<a href='https://ko-fi.com/Q5Q0BC92X' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

## Also read:

[How to Maintenance Server (Backup, etc)](https://github.com/albirrkarim/how-to-maintenance-server)

[Hubs Memory Efficiency & Usage Simulation (Private Repo)](https://github.com/albirrkarim/mozilla-hubs-optimization)

[Hosting Mozilla Hubs on VPS](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/VPS_FOR_HUBS.md)

[The Problem I Still Faced](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_UNSOLVED.md)

[The Problem I Faced and I Already Solved](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/PROBLEM_SOLVED.md)

[Tips for Modification](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/HOW_TO_MODIFY.md)

[Overview System With Figma](https://www.figma.com/file/h92Je1ac9AtgrR5OHVv9DZ/Overview-Mozilla-Hubs-Project?node-id=0%3A1)

[Experience Sharing About Hosting on Other Server](https://github.com/albirrkarim/mozilla-hubs-installation-detailed/blob/main/EXPERIENCE.md)
