# What is this?
This project is based on Alpine Linux, the official nginx image and an nginx module that provides static and dynamic brotli compression. [Brotli](https://github.com/google/brotli) and the [nginx brotli module ](https://github.com/google/ngx_brotli) are built by Google.

**Production-ready multi-architecture build**: This image is built for production use and supports both `amd64` and `arm64` platforms.

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
