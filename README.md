Kubernetes for Tradeflow Application

frontend http_prod
        mode http
        bind *:80
        default_backend tradeflow_prod

backend tradeflow_prod
        mode http
        option forwardfor
        balance roundrobin
        server node-worker-1 172.31.41.254:31723 check
        server node-worker-2 172.31.33.245:31723 check

frontend http_dev
        mode http
        bind *:8080
        default_backend tradeflow_dev

backend tradeflow_dev
        mode http
        option forwardfor
        balance roundrobin
        server node-worker-1 172.31.41.254:31417 check
        server node-worker-2 172.31.33.245:31417 check# TradeFlow-Helm
