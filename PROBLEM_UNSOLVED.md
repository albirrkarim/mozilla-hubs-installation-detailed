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
So what is `localhost:6000` ? 

I think mozilla not providing the source code for that. We have to provide our own backend service