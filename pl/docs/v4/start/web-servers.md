---
title: Web Servers
l10n-link: start-v4-web_servers
l10n-language: pl
---
T: Powrzechne jest używanie wzorca front-controller do przesyłania odpowiedniego żądania HTTP
 otrzymane przez serwer WWW do pojedynczego pliku PHP. 

Instrukcje zamieszczone poniżej wyjaśnijajæ, jak poinstruowac serwer WWW, aby wysyłał żądania HTTP do Twojego kontrolera PHP.

It is typical to use the front-controller pattern to funnel appropriate HTTP
requests received by your web server to a single PHP file. The instructions
below explain how to tell your web server to send HTTP requests to your PHP
front-controller file.

## PHP built-in server
T: Uruchom następujące polecenie w terminalu, aby uruchomić serwer WWW localhost,
zakładając, że `./public/` jest publicznie dostępnym katalogiem z plikiem `index.php`:

Run the following command in terminal to start localhost web server,
assuming `./public/` is public-accessible directory with `index.php` file:

```bash
cd public/
php -S localhost:8888
```
T: Jeśli nie używasz `index.php` jako punktu wejścia, zmień odpowiednio.

If you are not using `index.php` as your entry point then change appropriately.

> **Ostrzeżenie:** Wbudowany serwer sieciowy został zaprojektowany w celu wspomagania rozwoju aplikacji.
Może być również przydatny do celów testowych lub demonstracji aplikacji uruchamianych w kontrolowanych środowiskach. Nie jest to w pełni funkcjonalny serwer sieciowy. Nie należy go używać w sieci publicznej.
> 
> **Warning:** The built-in web server was designed to aid application development. 
It may also be useful for testing purposes or for application demonstrations that are run in controlled environments. It is not intended to be a full-featured web server. It should not be used on a public network.

## Kofiguracja Apache

T: Upewnij się, że moduł Apache `mod_rewrite` jest zainstalowany i włączony.
Aby włączyć `mod_rewrite`, wpisz następujące polecenie w terminalu:

```
sudo a2enmod rewrite
sudo a2enmod actions
```

T: Upewnij się, że pliki `.htaccess` i `index.php` są w tym samym
katalog dostępny publicznie. Plik `.htaccess` powinien zawierać następujący kod:

Ensure your `.htaccess` and `index.php` files are in the same
public-accessible directory. The `.htaccess` file should contain this code:

```bash
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```
T: Ten plik `.htaccess` wymaga przepisania adresu URL.

T: Upewnij się, że włączyłeś moduł `mod_rewrite` Apache i twój wirtualny host jest skonfigurowany
z opcją `AllowOverride`, aby można było użyć reguł przepisywania `.htaccess`:
Aby to zrobić, plik `/etc/apache2/apache2.conf` musi zostać otwarty w edytorze z uprawnieniami roota.

T: Zmień dyrektywę `<Directory ...>` z `AllowOveride None` na `AllowOveride All`.

This `.htaccess` file requires URL rewriting.

Make sure to enable Apache's `mod_rewrite` module and your virtual host is configured
with the `AllowOverride` option so that the `.htaccess` rewrite rules can be used:
To do this, the file `/etc/apache2/apache2.conf` must be opened in an editor with root privileges.

Change the `<Directory ...>` directive from `AllowOveride None` to `AllowOveride All`.

**Example**

```bash
<Directory /var/www/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```
T: Na koniec konfiguracja Apache musi zostać ponownie załadowana.
Aby ponownie uruchomić serwer WWW Apache, wpisz:

Finally, the configuration of Apache must be reloaded.
To restart Apache web server, enter:

```
sudo service apache2 restart
```
T: To polecenie działa na większości wariantów Debiana/Ubuntu.
W przypadku wszystkich innych dystrybucji Linuksa skonsultuj się
dokumentacja Twojej konkretnej dystrybucji Linuksa
aby dowiedzieć się, jak ponownie uruchomić Apache.

This command works on most Debian/Ubuntu variants.
For all other Linux distributions, please consult 
the documentation of your specific Linux distribution 
to find out how to restart Apache.

**Running in a sub-directory**

T: W poni«szym przykładzie zakłozylismy, że kontroler frontowy znajduje się w `public/index.php`.

This example assumes that the front controller is located in `public/index.php`.

T: Aby „przekierować” podkatalog do przedniego kontrolera, utwórz drugi
Plik `.htaccess` nad katalogiem `public/`.
 
To "redirect" the sub-directory to the front-controller create a second
`.htaccess` file above the `public/` directory. 

T: Drugi plik `.htaccess` powinien zawierać ten kod:

The second `.htaccess` file should contain this code:

```
RewriteEngine on
RewriteRule ^$ public/ [L]
RewriteRule (.*) public/$1 [L]
```

T: Możesz także ustawić ścieżkę bazową, aby router mógł:
dopasuj adres URL z przeglądarki do ścieżki ustawionej w rejestracji trasy.
Odbywa się to za pomocą metody `setBasePath()`.

You may also set the base path so that the router can 
match the URL from the browser with the path set in the route registration.
This is done with the `setBasePath()` method. 

```php
$app->setBasePath('/myapp');
```

**Read more**

* [Running Slim 4 in a subdirectory](https://akrabat.com/running-slim-4-in-a-subdirectory/)
* [Slim 4 base path middleware](https://github.com/selective-php/basepath)

## Nginx configuration

T: To jest przykładowa konfiguracja hosta wirtualnego Nginx dla domeny `example.com`.
Nasłuchuje przychodzących połączeń HTTP na porcie 80. Zakłada serwer PHP-FPM
działa na porcie 9123. Należy zaktualizować `nazwa_serwera`, `dziennik_błędów`,
Dyrektywy `access_log` i `root` z własnymi wartościami. Dyrektywa `root`
jest ścieżką do publicznego katalogu głównego dokumentu aplikacji; Twoja aplikacja Slim
W tym katalogu powinien znajdować się plik kontrolera frontowego `index.php`.


This is an example Nginx virtual host configuration for the domain `example.com`.
It listens for inbound HTTP connections on port 80. It assumes a PHP-FPM server
is running on port 9123. You should update the `server_name`, `error_log`,
`access_log`, and `root` directives with your own values. The `root` directive
is the path to your application's public document root directory; your Slim app's
`index.php` front-controller file should be in this directory.

```bash
server {
    listen 80;
    server_name example.com;
    index index.php;
    error_log /path/to/example.error.log;
    access_log /path/to/example.access.log;
    root /path/to/public;

    location / {
        try_files $uri /index.php$is_args$args;
    }

    location ~ \.php {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
        fastcgi_index index.php;
        fastcgi_pass 127.0.0.1:9123;
    }
}
```

## HipHop Virtual Machine

T: Twój plik konfiguracyjny maszyny wirtualnej HipHop powinien zawierać ten kod (wraz z innymi ustawieniami, których możesz potrzebować). Pamiętaj, aby zmienić ustawienie „SourceRoot”, aby wskazywało katalog główny dokumentu aplikacji Slim.

Your HipHop Virtual Machine configuration file should contain this code (along with other settings you may need). Be sure you change the `SourceRoot` setting to point to your Slim app's document root directory.

```bash
Server {
    SourceRoot = /path/to/public/directory
}

ServerVariables {
    SCRIPT_NAME = /index.php
}

VirtualHost {
    * {
        Pattern = .*
        RewriteRules {
            * {
                pattern = ^(.*)$
                to = index.php/$1
                qsa = true
            }
        }
    }
}
```

## IIS

T: Upewnij się, że pliki `Web.config` i `index.php` znajdują się w tym samym katalogu dostępnym publicznie. Plik `Web.config` powinien zawierać następujący kod:

Ensure the `Web.config` and `index.php` files are in the same public-accessible directory. The `Web.config` file should contain this code:

```bash
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="slim" patternSyntax="Wildcard">
                    <match url="*" />
                    <conditions>
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="index.php" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```

## lighttpd

T: Twój plik konfiguracyjny lighttpd powinien zawierać ten kod (wraz z innymi ustawieniami, których możesz potrzebować). Ten kod wymaga lighttpd >= 1.4.24.

Your lighttpd configuration file should contain this code (along with other settings you may need). This code requires lighttpd >= 1.4.24.

```bash
url.rewrite-if-not-file = ("(.*)" => "/index.php/$0")
```

T: Przy zało«eniu, że `index.php` Slima znajduje się w głównym folderze twojego projektu (www root).

## Run From a Sub-Directory

T:
                                                                                                        If you want to run your Slim Application from a sub-directory in your Server's Root instead of creating a Virtual Host, you can configure ``$app->setBasePath('path-to-your-app')`` right after the ``AppFactory::create()``.
Assuming that your Server's Root is ``/var/www/html/`` and path to your Slim Application is ``/var/www/html/my-slim-app`` you can set the base path to ``$app->setBasePath('/my-slim-app')``.

```php
<?php
use Slim\Factory\AppFactory;
use Slim\Middleware\OutputBufferingMiddleware;
// ...
$app = AppFactory::create();
$app->setBasePath('/my-slim-app');
// ...
$app->run();
```
