# The problem i still faced

1. 502 server communication error in hubs admin like this [issue](https://github.com/mozilla/hubs/issues/4970#issue-1087523703)

Oke let's give try solve this.

The problem is api call to this route

`/api/ita/admin-info`

and other with slash ita `/ita`

if we check where is the code in the reticulum which handle that
open the `router.ex` ita api call the `RetWeb.Plugs.ItaProxy` if you in vs code you can command+click / ctrl+click for find the function is called.

```elixir

scope "/api/postgrest" do
   pipe_through([:secure_headers, :auth_required, :admin_required, :proxy_api])
   forward("/", RetWeb.Plugs.PostgrestProxy)
end

scope "/api/ita" do
   pipe_through([:secure_headers, :auth_required, :admin_required, :proxy_api])
   forward("/", RetWeb.Plugs.ItaProxy)
end

```

So it will open the `proxies.ex`

```elixir
defmodule RetWeb.Plugs.PostgrestProxy do
  use Plug.Builder
  plug ReverseProxyPlug, upstream: "http://localhost:3000"
end

defmodule RetWeb.Plugs.ItaProxy do
  use Plug.Builder
  plug ReverseProxyPlug, upstream: "http://localhost:6000"
end
```

## What is port 3000?

Thanks to contributor that give me a clue about the port 3000

its a [PostgREST](https://postgrest.org/en/stable/index.html)

[see](https://github.com/mozilla/hubs-ops/blob/master/ansible/roles/postgrest/templates/postgrest.toml.j2)

postREST config 

```
db-uri = "postgres://postgres:postgres@localhost:5432/ret_dev"
db-schema = "ret0_admin"
db-anon-role = "postgrest_anonymous"
server-host = "localhost"
server-port = 3000
log-level = "info"

jwt-secret-enabled = true
jwt-aud = "ret_perms"
jwt-role-claim-key = ".postgrest_role"
jwt-secret = "jwk.json"
```

I will update this repo if i already solve this


## and what is `localhost:6000` ? 

I think mozilla not providing the source code for that. We have to provide our own backend service


