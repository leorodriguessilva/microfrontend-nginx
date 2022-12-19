# Micro frontends witgh Nginx

This repository uses the SSI (server side includes) module from Nginx to inject html header fragment into a main page as a header, this is being done in order to clarify the process behind Nginx SSI module.

## Composition of the project

The project is composed by a main Nginx with a default page, which has instruction to inject a html page as a header fragment in the main page, thus rendering a whole page composed of a header fragment and a body text (nothing fancy). 

 - main_page
  - Dockerfile
  - index.html
  - nginx.config

  So the main page Nginx is configured with the SSI module on (see [nginx.conf](./main_page/conf/nginx.conf)), and a reverse proxy (`/header`) pointing to the host of the header html page named (surprisingly) as "header" (see [docker compose](docker-compose.yaml) file).
  And a [Dockerfile](./main_page/Dockerfile) to build a new Nginx image with new Nginx SSI configuration setup, plus the reverse proxy setup and the main html page with the SSI command to include the reverse proxy content.

 - header_fragment
  - Dockerfile
  - index.html

  The header fragment is much more simpler than the main page setup, as we just need a default Nginx (or any other web server to serve html pages). So i contains a [html page](./header_fragment/pages/index.html) which resembles the header of our application, and a [Dockerfile](./header_fragment/Dockerfile) which builds a new Nginx image with the html page and that is it.

 - docker-compose.yaml

  The Docker compose starts the build process of both images and delegates the right ports (8080, 8081) to both webservers (Nginx main and header).

## How things happen?

Before starting the explanation, to make the explanation easier the SSI enabled Nginx will be called as "main" and the header fragment server will be called as "header".

So the main Nginx which have the SSI module set to `on` is now capable of interpreting [SSI commands](http://nginx.org/en/docs/http/ngx_http_ssi_module.html#commands), so every html page in the root folder of the Nginx server (`/usr/share/nginx/html` see [nginx.conf](./main_page/conf/nginx.conf) at line 14) will now be subject of a interpreting step which will lookup for the SSI commands, and execute then and informing error in case of errors or rendering the desired output from the command which can be html from other hosts or environment variables. Imagine now that your html pages are now pre compiled and then served by Nginx.
As the main server is now configured we still need to setup the html main page to execute the desired command to include the html from the header server, we can achieve that by configuring a reverse proxy in the main Nginx by giving it a name and a route (see [nginx.conf](./main_page/conf/nginx.conf) from line 18 to 27), so the chosen route is `/header` and whenever a request is made to the main at the route `/header` it will lookup for the server header (see [nginx.conf](./main_page/conf/nginx.conf) line 22), and whichever response given by the header server it will be rendered by the main server in the browser.
So now that we have the SSI enabled and the reverse proxy is setup we can now put the pieces together with the [`index.html`](./main_page/pages/index.html) by adding a SSI command to be interpreted by the main server and thus replacing the command the injected html comming from the reverser proxy `/header`. So looking at the [`index.html`](./main_page/pages/index.html) at line 9 we can see the command `<!--#include virtual="/header" -->` being used which basically tell the main Nginx to fetch the html resulting from the reverse proxy `/header` request and inject it at that line.
 

## How to run

The project uses Docker and nothing more, so make sure you have `docker` and `docker compose` installed.

Then one can execute:

```
docker compose -f "docker-compose.yaml" up -d --build 
```

And both servers should be up and running after fetching the Nginx image (in case you don't have it already) and building the new images for the main and header.
After both servers are up and running the main should be reachable at the address `http://localhost:8080`, the reverse proxy to the header server should be reachable at the address `http://localhost:8080/header` and the standalone header should be reachable at the address `http://localhost:8081`.

## Known issues

In the header don't appear when requesting for the main page, make sure to invalidate all local cache, or use anonymised browsing from chrome or any other browser you like.