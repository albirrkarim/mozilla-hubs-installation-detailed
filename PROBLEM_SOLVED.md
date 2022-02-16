# The problem i faced and i already solved

## 1. Local scene asset failed to load 403 Forbidden

Api call POST to `/api/v1/media` 403 Forbidden

![Local assets](/docs_img//local_assets_failed.png)

## 2. Architecture Kit Failed to load and import

This problem is related to number 1 above. We don't need `CORS_PROXY_SERVER` so set it with empty string

![env spoke](/docs_img/env_spoke.png)

and make condition like this picture bellow

![env spoke](/docs_img/env_spoke_1.png)

On spoke if we want to import some object in architecture kit we got error 500 from api call POST to `/api/v1/media` like number 1 above.

## 3. Link element fail to load thumbnail

On the spoke we can put link to external web. like `github.com/albirrkarim` normally hubs will show up the screenshot of that web link.

Call `api/v1/media` got 500

<br>
<br>

# The solution for number 1,2,3

Goto `media_controller.ex` give fallback like this picture
![Local assets](/docs_img/spoke_failed_2.png)

Goto `media_resolver.ex` file and change the return `:forbidden` with like this picture

![Local assets](/docs_img/spoke_failed.png)

Add this function

```elixir
def resolve_with_content_type(%MediaResolverQuery{url: %URI{} = uri}) do
   content_type = MIME.from_path(uri.path)
   uri |> resolved(%{expected_content_type: content_type})
end
```

Below is the solution for number 2. Add another resolve function that `root_host` equal to `reticulum.io`

```elixir
def resolve(%MediaResolverQuery{} = query, "reticulum.io" = _root_host) do
    resolve_with_context_type(query)
end
```

FYI: the `root_host` is a variabel, but why i add some underscore `_` at the begining? because it's unused variable in the body of function. we just need that var for conditioning on head of function.