# Nightmare #1 


## Serving static files in django Production

Handling static files has not been a nightmare until I had to deploy it.
Since django doesn't handle static files in production, it had to be served via some other servers like Nginx or Apache.


## Well, the solution was as simple as that

1. You have to do the deployment checklist provided by django `python manage.py check --deploy`
2. Do the collectstatic `python manage.py collectstatic`. This brings all of your static files under one folder, which is easy to handle
3. If your app contains any file upload, then you have to handle that too with your static file server. 
4. I had used Nginx for my deployment. In that the folders containing static files and media files had to be mirrored to a folder which Nginx can access will all the necessary permissions. `sudo bindfs -u www-data -g www-data /home/username/project-name/static /usr/share/nginx/project-name`
5. Do this for both static files folder and media files folder
6. In you Nginx conf,  do it like this:
```
server{
        server_name yourdomain.name;
        location /media/ {
            root /usr/share/nginx/project-name/;
        }
        location /static/ {
            root /usr/share/nginx/project-name/;
        }
	location / {
            include  proxy_params;
            proxy_pass http://0.0.0.0:8001;
       }
```

