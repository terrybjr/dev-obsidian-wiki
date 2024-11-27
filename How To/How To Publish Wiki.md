1. Make change
2. Push code to Github
3. Pull code in Server
4. if(new quartz installation) run npm intall
5. run  ```npx quartz build
6. rename "public" folder to 'wiki' this is how I get the PATH to work to DOMAIN_URL/wiki
7. If (new quartz installation) may have to configure nginx
   ```
   # Specific location for "/wiki" endpoint
   location /wiki {
		root C:/Users/Server/Documents/GitHub/dev-site-wiki/;
		index index.html;

		# This will attempt to serve file as is, directories with index.html
		# and will try to append .html to files if it doesn't find them.
		try_files $uri $uri/ $uri.html =404;
	}
```