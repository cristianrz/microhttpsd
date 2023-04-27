# microhttpsd

microhttpsd is a simple script that runs a secure web server on a Linux system only
using `openssl` and `busybox httpd`. The web server can serve static files from a
specified directory and supports HTTPS connections.

`openssl` provides a way to server static files, however it is very rudimentary
(`/` won't serve `index.html`, etc) and sometimes I would like to serve an
internal HTTPS server without installing any additional software.

## Usage

```
Usage: microhttpsd [-p <port>] [-k <key_file>] [-c <cert_file>] [-d <home>] [-v] [-h]

Options:
    -p <port>      Port to listen on (default: 8443)
    -k <key_file>  Path to private key file (default: key.pem)
    -c <cert_file> Path to certificate file (default: cert.pem)
    -d <home>      Home directory (default: .)
    -v             Enable verbose output
    -h             Show this help message
```

To run the web server, simply execute the microhttpsd script with the desired
options.

For example, to serve files from the `/var/www/html` directory on port 443
(HTTPS), use:

```
./microhttpsd -p 443 -d /var/www/html
```

## Implementation Details

microhttpsd creates two named pipes (`input_fifo` and `output_fifo`) to communicate
between `openssl` and `busybox httpd`. `openssl` listens for incoming HTTPS
connections and sends incoming requests to `busybox httpd` through the
`input_fifo` pipe. `busybox httpd` serves the requested files and sends the
response back through the `output_fifo` pipe, which `openssl` then sends to the
client.

In addition, `microhttpsd` continuously restarts `busybox httpd` to handle
subsequent requests.

`microhttpsd` sets up a trap to clean up the named pipes and terminate `openssl`
when it receives a `SIGINT`, `SIGTERM`, or `EXIT` signal.

