# ehttpd

ehttpd is a simple script that runs a secure web server on a Linux system using
openssl and busybox httpd. The web server can serve static files from a
specified directory and supports HTTPS connections.

## Usage

```
Usage: ehttpd [-p <port>] [-k <key_file>] [-c <cert_file>] [-d <home>] [-v] [-h]

Options:
    -p <port>      Port to listen on (default: 8443)
    -k <key_file>  Path to private key file (default: key.pem)
    -c <cert_file> Path to certificate file (default: cert.pem)
    -d <home>      Home directory (default: .)
    -v             Enable verbose output
    -h             Show this help message
```

To run the web server, simply execute the ehttpd script with the desired
options.

For example, to serve files from the `/var/www/html` directory on port 443
(HTTPS), use:

```
./ehttpd -p 443 -d /var/www/html
```

## Implementation Details

ehttpd creates two named pipes (`input_fifo` and `output_fifo`) to communicate
between `openssl` and `busybox httpd`. `openssl` listens for incoming HTTPS
connections and sends incoming requests to `busybox httpd` through the
`input_fifo` pipe. `busybox httpd` serves the requested files and sends the
response back through the `output_fifo` pipe, which `openssl` then sends to the
client.

In addition, `ehttpd` continuously restarts `busybox httpd` to handle
subsequent requests.

`ehttpd` sets up a trap to clean up the named pipes and terminate `openssl`
when it receives a `SIGINT`, `SIGTERM`, or `EXIT` signal.

