# gx-cookie-proxy

gx-cookie-proxy is a WebSocket-aware SSL-capable HTTP reverse proxy based on
[drunken-hipster](https://github.com/joinmytalk/drunken-hipster)

## Deployment

### Pre-requisites

1. You are already running some sort of proxy, such as Apache2 or NGINX
2. You have deployed a Galaxy server with Postgres as the database

## Deployment

Download the binary from our [releases page](https://github.com/erasche/gx-cookie-proxy/releases) and run it:

```console
./gx-cookie-proxy \
	--galaxyDb 'postgresql://postgres:postgres@localhost:32769/postgres?client_encoding=utf8&sslmode=disable' \
	--galaxySecret 'I_LOVE_ICE_CREAM' \ # As it appears in galaxy.ini
	--listenAddr localhost:5000 \ # Address to listen on
	--connect localhost:8080 # The backend you're connecting to
```

This will cause the proxy to:

- create a tunnel between frontend and backend
- connect to the database in order to decrypt cookies into usernames

On the first request, the proxy will check the cookie and attempt to decrypt it
based on the secret and the session in the database (i.e. an active galaxy
session MUST be present).

On subsequent requests, the proxy will check its cache for that cookie value,
improving performance. Cookies are cached for a maximum of one hour. This can
be made configurable if someone requests it.

## Configuration

Example apache2 configuration:

```apache2
ProxyPass  /galaxy/gxc_proxy http://localhost:5000/galaxy/gxc_proxy
<Location "/galaxy/gxc_proxy">
	ProxyPassReverse http://localhost:5000/galaxy/gxc_proxy
</Location>
```

This will connect to your backend service (running on `localhost:8080`), and
proxy requests to the backend. The backend service should either listen on
`/galaxy/gxc_proxy/.*`, or should use completely relative paths rather than
absolute.

Note that my proxy shares a leading path component with my galaxy
server. This is required in order to access the galaxy session cookie
due to cookie restrictions.

The gx-cookie-proxy is also configurable via environment variables:

Parameter        | Env Var           | Usage
---------------- | ----------------- | -----------
`--galaxyDb`     | `GALAXY_DB_URL`   | Galaxy Database Address
`--galaxySecret` | `GALAXY_SECRET`   | Galaxy cookie secret
`--listenAddr`   | `GXC_LISTEN_ADDR` | Proxy listening address
`--connect`      | `GXC_BACKEND_URL` | Backend host + port to connect to

# License

MIT

# Authors

Original Drunken Hipster Proxy - Andreas Krennmair <ak@synflood.at>
Galaxy Portions - Eric Rasche <esr@tamu.edu>
