---
title: "How `httr` caches tokens"
output: html_document
---
This is the flow to generate a token, by setting `cache = TRUE`, `httr` leaves behind a `.httr-oauth` file in the current working directory.


```r
scope_list <- c("https://spreadsheets.google.com/feeds",
                "https://docs.google.com/feeds")

googlesheets_app <-
  httr::oauth_app("google",
                  key = getOption("googlesheets.client_id"),
                  secret = getOption("googlesheets.client_secret"))

google_token <-
  httr::oauth2.0_token(httr::oauth_endpoints("google"), googlesheets_app,
                       scope = scope_list, cache = TRUE)
```

This is what the token looks like: 

```r
> google_token
<Token>
<oauth_endpoint>
 authorize: https://accounts.google.com/o/oauth2/auth
 access:    https://accounts.google.com/o/oauth2/token
 validate:  https://www.googleapis.com/oauth2/v1/tokeninfo
 revoke:    https://accounts.google.com/o/oauth2/revoke
<oauth_app> google
  key:    178989665258-f4scmimctv2o96isfppehg1qesrpvjro.apps.googleusercontent.com
  secret: <hidden>
<credentials> access_token, token_type, expires_in, refresh_token
---
```

When `httr` caches it into `.httr-oauth`, this is what it looks like when you read it directly with `readRDS()`.


```r
> token <- readRDS(".httr-oauth")
> token
$faa68f85c8290ff6d9f1ac0811d605e3
<Token>
<oauth_endpoint>
 authorize: https://accounts.google.com/o/oauth2/auth
 access:    https://accounts.google.com/o/oauth2/token
 validate:  https://www.googleapis.com/oauth2/v1/tokeninfo
 revoke:    https://accounts.google.com/o/oauth2/revoke
<oauth_app> google
  key:    178989665258-f4scmimctv2o96isfppehg1qesrpvjro.apps.googleusercontent.com
  secret: <hidden>
<credentials> access_token, token_type, expires_in, refresh_token
---
```

During the caching process, httr creates a [hash function](https://github.com/hadley/httr/blob/e8a85c0a137c543090b60c694ea4d332e4833d68/R/oauth-token.r#L104) based on the arguments passed in `google_token <- httr::oauth2.0_token(httr::oauth_endpoints("google"), googlesheets_app, scope = scope_list, cache = TRUE)`, specifically, the endpoints, app info (client identification), and scopes. The hash function **serves as a id** for the token, because tokens for other applications with different endpoints, app info and scopes are also cached in `.httr-oauth`. To save the token, `httr` first reads in `.httr-oauth` and finds the slot (using the hash) for the token and saves it there using `saveRDS()`. [See here](https://github.com/hadley/httr/blob/e746e973e18504996f3d4916c9f2bba334a85ac8/R/oauth-cache.R#L54) for the caching functions. 


```r
tokens <- load_cache(cache_path) # which is calling readRDS(cache_path) where cache_path is .httr-oauth
tokens[[token$hash()]] <- token
saveRDS(tokens, cache_path)
```

When `httr` loads a cached token, it calls on `readRDS(cache_path)`, generates the hash function from the arguments passed into `httr::oauth2.0_token()` and uses it to access the token.


```r
readRDS(cache_path)[[hash]]
```

