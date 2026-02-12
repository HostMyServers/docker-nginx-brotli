# What is this?
This project is based on Alpine Linux, the official nginx image and an nginx module that provides static and dynamic brotli compression. [Brotli](https://github.com/google/brotli) and the [nginx brotli module ](https://github.com/google/ngx_brotli) are built by Google.

**Production-ready multi-architecture build**: This image is built for production use and supports both `amd64` and `arm64` platforms.

**HTTP/3 Support**: This image includes experimental HTTP/3 (QUIC) support via the `ngx_http_v3_module`. See the HTTP/3 configuration section below for usage details.

# How to use this image
As this project is based on the official [nginx image](https://hub.docker.com/_/nginx/) look for instructions there. In addition to the standard configuration directives, you'll be able to use the brotli module specific ones, see [here for official documentation](https://github.com/google/ngx_brotli#configuration-directives)

## Example Nginx Configuration with Brotli

Here's a complete example showing how to configure the Brotli module in your nginx configuration:

```nginx
http {
    # Enable brotli compression
    brotli on;
    
    # Brotli compression level (0-11, default: 6)
    # Higher values = better compression but more CPU usage
    brotli_comp_level 6;
    
    # Minimum response length to compress (in bytes)
    brotli_min_length 20;
    
    # MIME types to compress
    brotli_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/x-javascript
        application/xml
        application/xml+rss
        application/rss+xml
        application/atom+xml
        image/svg+xml
        font/ttf
        font/otf
        font/woff
        font/woff2;
    
    # Enable static brotli compression
    # Looks for pre-compressed .br files
    brotli_static on;
    
    server {
        listen 80;
        server_name example.com;
        
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
}
```

### Key Configuration Directives

- **`brotli on/off`**: Enable/disable dynamic brotli compression
- **`brotli_comp_level`**: Compression level (0-11). Recommended: 4-6 for production
- **`brotli_static on/off`**: Serve pre-compressed `.br` files if they exist
- **`brotli_types`**: MIME types to compress
- **`brotli_min_length`**: Minimum response size to compress (in bytes)

## HTTP/3 Configuration (Experimental)

This image includes support for HTTP/3 (QUIC protocol). HTTP/3 uses UDP instead of TCP and provides improved performance, especially on mobile networks.

### Example HTTP/3 Configuration

```nginx
http {
    # Enable brotli (see above for full brotli config)
    brotli on;
    brotli_comp_level 6;
    
    # Custom log format to track HTTP/3 connections
    log_format quic '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" "$http3"';
    
    access_log /var/log/nginx/access.log quic;
    
    server {
        listen 443 ssl;                    # HTTP/2 and HTTP/1.1
        listen 443 quic reuseport;         # HTTP/3
        
        http2 on;                          # Enable HTTP/2
        http3 on;                          # Enable HTTP/3
        
        ssl_certificate /path/to/cert.crt;
        ssl_certificate_key /path/to/cert.key;
        
        # Protocols and ciphers
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers off;
        
        location / {
            # Advertise HTTP/3 availability to clients
            add_header Alt-Svc 'h3=":443"; ma=86400';
            
            root /usr/share/nginx/html;
            index index.html;
        }
    }
}
```

### Docker Run Example for HTTP/3

When running the container, you need to expose UDP port 443 for QUIC:

```bash
docker run -d \
  -p 80:80 \
  -p 443:443 \
  -p 443:443/udp \
  -v /path/to/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /path/to/certs:/etc/ssl/certs:ro \
  your-image-name
```

### Docker Compose Example

```yaml
version: '3'
services:
  nginx:
    image: your-image-name
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"  # Required for HTTP/3
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/ssl/certs:ro
```

### Important Notes on HTTP/3

- **Experimental**: HTTP/3 support in nginx is experimental. Use with caution in production.
- **SSL Required**: HTTP/3 requires SSL/TLS certificates.
- **UDP Port**: Make sure to expose port 443/udp for QUIC traffic.
- **Browser Support**: Modern browsers support HTTP/3, but they will fallback to HTTP/2 or HTTP/1.1 if HTTP/3 is unavailable.
- **Alt-Svc Header**: The `Alt-Svc` header advertises HTTP/3 availability to clients.

### HTTP/3 Configuration Directives

- **`http3 on/off`**: Enable/disable HTTP/3 protocol negotiation (default: on)
- **`http3_max_concurrent_streams`**: Maximum number of concurrent HTTP/3 streams (default: 128)
- **`http3_stream_buffer_size`**: Buffer size for QUIC streams (default: 64k)
- **`quic_retry on/off`**: Enable QUIC address validation (default: off)
- **`quic_gso on/off`**: Enable optimized batch sending on Linux (default: off)

For more details, see the [official nginx HTTP/3 documentation](https://nginx.org/en/docs/http/ngx_http_v3_module.html).
