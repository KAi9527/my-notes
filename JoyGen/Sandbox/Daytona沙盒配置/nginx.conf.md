events {
    worker_connections 1024;
}

http {
    access_log    /dev/stdout;
    error_log     /dev/stderr;
    include       /usr/local/openresty/nginx/conf/mime.types;
    default_type  application/octet-stream;

    server {
        listen 8080;
        server_name localhost;

        # 探活路由
        location /1 {
            return 200 'OK';
            add_header Content-Type text/plain;
        }

        location /debug-log {
            alias /app/logs/be.log;
            default_type text/plain;
            add_header Cache-Control "no-store, no-cache, must-revalidate";
            charset utf-8;
        }

        # 查看日志
        location /log {
            alias /app/logs/;
            autoindex on;
            default_type text/plain;
            add_header Cache-Control "no-store, no-cache, must-revalidate";
            charset utf-8;
        }

        # 执行系统命令 (仅用于调试沙盒，需 ngx_http_lua_module)
        location /command {
            default_type text/plain;
            content_by_lua_block {
                local cmd = ngx.unescape_uri(ngx.var.query_string)
                if cmd and cmd ~= "" then
                    local handle = io.popen(cmd)
                    local result = handle:read("*a")
                    handle:close()
                    ngx.say(result)
                else
                    ngx.say("No command provided. Usage: /command?pwd")
                end
            }
        }

        # 后端反向代理
        location / {
            proxy_pass http://localhost:8081/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_set_header X-Forwarded-Proto https; # 强制告诉后端：浏览器用的是HTTPS
            proxy_set_header X-Forwarded-Port 443;
        }
    }
}
