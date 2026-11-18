# Visit counter

A web page that shows how many times it has been visited.

Live version: https://student21.wt.thomasjuldo.com

It is made of three services, started together by Docker Compose:

| Service | What it does | Built from |
|---|---|---|
| `site` | Serves the web page (`site/index.html`) | `nginx:alpine` |
| `api` | Flask API: `GET /visits` adds 1 to the counter and returns `{"visits": N}` | `api/Dockerfile` |
| `db` | Redis database that stores the counter, kept in the `counter-data` volume | `redis:alpine` |

The page calls the API with `fetch`, the API asks Redis to add 1 (`INCR`) and sends the new total back.

## Run it locally

You need [Docker Desktop](https://www.docker.com/products/docker-desktop/) (running) and Git.

1. Download the project:

   ```bash
   git clone https://github.com/MaxVial/visit-counter.git
   cd visit-counter
   ```

2. Build and start the three services:

   ```bash
   docker compose up -d --build
   ```

   The first run downloads the images and installs Flask, so it takes a minute.

3. Open http://localhost:8081 in a browser. The page shows `Visits to this page: 1`, and each reload adds 1.

   The API can also be called directly: http://localhost:8082/visits

4. Stop everything:

   ```bash
   docker compose down
   ```

   The counter is kept in the `counter-data` volume, so it starts again from the same number next time.
   `docker compose down -v` also deletes the volume and resets the counter to 0.

No configuration or API key is needed.

### If something does not work

- **`port is already allocated`**: another program uses port 8081 or 8082. Stop it, or change the left number in `ports` in `compose.yaml` (for example `"9081:80"`). The page detects the API with port 8081, so keep 8081 for `site` if possible.
- **The page says "The API cannot be reached"**: check that the three services are running with `docker compose ps`, and read the API logs with `docker compose logs api`.

## Files

```
compose.yaml          the three services for local use
api/app.py            the Flask API
api/Dockerfile        how the API image is built
api/requirements.txt  Python dependencies (flask, redis)
site/index.html       the page
compose.server.yaml   extra settings used only on the server (see below)
Caddyfile             reverse proxy configuration used only on the server
```

## On the server

On the server, `compose.server.yaml` is added on top of `compose.yaml`:

```bash
docker compose -f compose.yaml -f compose.server.yaml up -d --build
```

It removes the public ports of `site` and `api`, restarts the services automatically, and adds **Caddy**, which is the only service open to the internet (ports 80 and 443). Caddy gets the HTTPS certificate automatically and routes the requests:

- `/api/...` goes to the API (with `/api` removed, so `/api/visits` becomes `/visits`),
- `/portfolio/...` goes to my portfolio, built from the [portfolio](https://github.com/MaxVial/portfolio) repository cloned next to this one,
- everything else goes to the page.

Because the page and the API share the same address on the server, the page calls `/api/visits` there, and `http://localhost:8082/visits` locally.
