# Portal

Portal provides the ability to access specific CRM data and functions for your customers and partners. Administrators can create multiple portals. Each portal can have its own settings, dashboard, user list, access control settings.

To create portal follow Administration > Portals, click Create Portal button.

* *Is Active*. If not checked, the portal won't be available for anybody.
* *Is Default*. Means that the portal will be available by shorter url: `https://YOUR_ESPO_URL/portal`.
* *Roles*. Specify one or multiple portal roles that will be applied to users logged into the portal. More information about the portal roles is below.
* *Tab List*. Tabs which will be shown in the navigation bar.
* *Dashboard Layout*. Specify dashlets that will be displayed on the home page of the portal. Note that portal users can't configure their dashboard.
* *URL*. Read-only field that displays the link you can access the portal with.
* *Layout Set*. Provides the ability to use different layouts from the portal. See more [info](layout-manager.md#different-layouts-for-teams-portals).

## Portal Users

Administrators can create portal users.

1. Administration > Portal Users.
2. Click *Create Portal User* button .
3. Select Contact the portal user will be linked with or *Proceed w/o Contact*
4. Fill needed fields on the form and click *Save*.

Portal user should be linked to Portal record to be able to access that portal.

Portal users can have one or multiple additional *Portal Roles*. These roles will be merged with roles specified for a portal.

Portal users can have a specific *Dashboard Layout*. It allows certain users to have a specific layout that differs from the default portal layout.

## Portal Roles

Portal roles are similar to regular roles in EspoCRM but with a few distinctions.

* not-set ‒ Denies access.
* own ‒ Records created by the user. E.g. a portal user created some case and this case is owned by this user.
* account ‒ Records related to the account the portal user is related to. Relation (link) should be named `account` or `accounts`.
* contact ‒ Records related to the contact the portal user is related to. Relation (link) should be named `contact` or `contacts`.

*Assigned User* and *Teams* fields are read-only for portal users.

Portal roles can be applied to:

* Portal ‒ all users of the portal will receive this role (multiple roles are merged);
* Portal User ‒ to grant certain users specific permissions.

### Example

*Portal users should be able to create cases, view cases related to their account; they should be able to view knowledge base.*

1. Open Create Portal Role form (Administration > Portal Roles > Create Role).
2. Enable access to Cases, set: *create - yes, read - account, edit - no, delete - no, stream - account*.
3. Enable access to Knowledge Base, set: *create - no, read - account, edit - no, delete - no*.
4. Edit your portal record (Administration > Portals). Select your portal role in Roles field and then save.

## Access to Portal

You can find the URL for your portal in the *URL* field of the portal record. It's also possible to use server configuration tools (such as mod_rewrite) to be able to access by different url. For this case, you need to fill in 'Custom URL' field.

### Access by Custom URL for Apache server

Custom URL: `portal-host-name.com`.
Portal ID: `16b9hm41c069e6j24`.

#### crm.portal.conf

```
<VirtualHost *:80>
    DocumentRoot /path/to/espocrm/instance/
    ServerName portal-host-name.com

    <Directory /path/to/espocrm/instance/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### Mod rewrite rules

Specify portal record ID instead of `{PORTAL_ID}`. Portal record ID can be obtained from the address bar of your web browser when you open the detail view of the portal record. Like: `https://my-espocrm-url.com/#Portal/16b9hm41c069e6j24`. *16b9hm41c069e6j24* is the portal record ID.

```
  RewriteCond %{HTTP_HOST} ^portal-host-name\.com$
  RewriteRule ^client - [L]

  RewriteCond %{HTTP_HOST} ^portal-host-name\.com$
  RewriteCond %{REQUEST_URI} !^/portal/{PORTAL_ID}/.*$
  RewriteRule ^(.*)$ /portal/{PORTAL_ID}/$1 [L]
```

### Access portal by Custom URL for Nginx server

Custom URL: `portal-host-name.com`.
Portal ID: `5a8a9b9328e6a955b`.

#### crm.portal.conf
```
server {
    listen 80;
    listen [::]:80;

    server_name portal-host-name.com; # Replace portal-host-name to your domain name
    root /var/www/html/espocrm; # Specify your EspoCRM document root

    index index.php index.html index.htm;

    # SSL configuration
    #
    # listen 443 ssl;
    # listen [::]:443 ssl;
    # include snippets/snakeoil.conf;

    # Specify your PHP (php-cgi or php-fpm) based on your configuration
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        # With php7.3-cgi alone:
        # fastcgi_pass 127.0.0.1:9000;

        # With php7.3-fpm:
        fastcgi_pass unix:/run/php/php7.3-fpm.sock;
        fastcgi_param ESPO_PORTAL_IS_CUSTOM_URL "true";
        include snippets/fastcgi-php.conf;
    }

    # Add rewrite rules
    location /client {
        rewrite ^/client/(.*) /client/$1 break;
    }

    location / {
        proxy_pass http://portal-host-name.com/portal/5a8a9b9328e6a955b/;
    }

    location /api/v1/ {
        if (!-e $request_filename){
            rewrite ^/api/v1/(.*)$ /api/v1/index.php last; break;
        }
    }

    location /api/v1/portal-access {
        if (!-e $request_filename){
            rewrite ^/api/v1/(.*)$ /api/v1/portal-access/index.php last; break;
        }
    }

    location /portal/ {
        try_files $uri $uri/ /portal/index.php?$query_string;
    }

    location ^~ (data|api)/ {
        if (-e $request_filename){
            return 403;
        }
    }
    
    location ^~ /data/ {
        deny all;
    }
    location ^~ /application/ {
        deny all;
    }
    location ^~ /custom/ {
        deny all;
    }
    location ^~ /vendor/ {
        deny all;
    }
    location ~ /\.ht {
        deny all;
    }
}
```

## See also

[Portal ACL customization](../development/acl.md#portal-acl)
