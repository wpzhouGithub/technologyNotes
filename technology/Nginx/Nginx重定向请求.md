

# Nginx重定向请求

```shell
 rewrite /break.html /index.html break;
 rewrite /client/v2/wallpaper_reco/(.*) /client/v3/wallpaper_reco/$1 break;
 rewrite /client/v2/game_reco/games_today.json /client/v3/game_reco/games_today.json;
```

