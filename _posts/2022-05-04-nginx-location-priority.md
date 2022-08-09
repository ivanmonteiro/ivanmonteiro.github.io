---
layout: post
title:  "Understanding nginx location priority"
date:   2022-05-04
categories: nginx reverse-proxy
---

To help clarify nginx location priority I will expliain the order in which nginx will try to match a location to a request.

1. Exact Match 

Nginx will look for exact match locations first. If it finds one, it will stop looking for other locations.

e.g.:
```
location = /about {
    #...
}
```

2. Preferential Prefix

Next, nginx will look for locations with a preferential prefix. If it finds one, it will stop looking for other locations.

e.g.:
```
location ^~ /about {
    #...
}
```

3. Regex Match

If no exact match or preferential prefix is found, nginx will look for regex match locations.

e.g.:
```
location ~ ^/about {
    #...
}
```

4. No Modifier 

If none of the above is found, nginx will look for locations with no modifier. So if you have a location with no modifier, it will be the last one to be and will be the one that will be used.

e.g.:
```
location /about {
    #...
}
```