

## 启动

1. uwsgi --ini scripts/uwsgi.ini  
2. daphne -b 0.0.0.0 -p 5015 acapp.asgi:application
3. ./main.py -- thrift
4. sudo /etc/init.d/nginx start
5. redis-server /etc/redis/redis.conf 