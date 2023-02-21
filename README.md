## How to install
I am using docker for demonstrating benefits of the opcache preloading.

If you have docker installed then you can get up and running by following.

1 - Clone repository:

    git clone https://github.com/hasnatinter/laravel-preload-example
2 - Run docker composer:

    docker-compose up -d --build 
3 - Check that the two containers are running:
    
    docker-compose ps

4 - Use apache bench (or similar HTTP benchmarking tool) to test the performance:
**Note:** that I am using 78 port for my application. You can set the port in env file by setting `APP_PORT_LOCAL` variable.

    ab -n 1000 -c 25 -l http://localhost:78

5 - Check performance without preloading:
Open`docker/php/php.ini`file and comment out preload and preload_user so your files looks like:

    opcache.enable=1
    ;opcache.preload=/var/www/html/preload.php
    ;opcache.preload_user=www-data

6 - Rebuild docker image by repeating step 2 and test using step 4:

    docker compose up -d --build
    ab -n 1000 -c 25 -l http://localhost:78

7 - Observe the difference in performance of two tests. You may want to run benchmark command multiple times and maybe by increasing the number of request to 5000 `-n 5000`

8- Run without opcache enabled: To see full benefits of opcache, do try to run once with opcache disabled by setting in `docker/php/php.ini`file

    opcache.enable=0

Also note that factors like cache warming will be eliminated by running more than once.

Now you will be fully able to appreciate the benefits of opcache in PHP.

### When to use opcache and preloading:
As a rule, use opcache while developing and use with preloading in production.

## Further reading
- [Opcache preloading in PHP and how to properly us it in Laravel](https://medium.com/@hasnatinter10/opcache-preloading-in-php-and-how-to-properly-us-it-in-laravel-8fe273356b4e)

- [PHP RFC: Preloading](https://wiki.php.net/rfc/preload)
