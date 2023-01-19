# 040 LavaSuite Initial Setup

## switch back from other dev project (e.g from lava to lavasuite)

```bash
update-alternatives --config php
# choose v8.0
update-alternatives --config node
# choose v16
ng --version
# check Angular is v13, nodejs is 16 and npm is >8.5
# if tools/checkOwnVersions.sh is complaining there are several option:
#   - downgrade npm with e.g. `npm install -g npm@8.5.5` or
#   - `chown root:root /var/www/lavasuite/lava-ui-*` or
#   - use npm alias or function (of ~/.bash_configs/bash_lava), which is:
#     function npm() { >&2 echo "sudo -u www-data  npm $*" ; sudo -u www-data npm ${@} ; }
# if still problems with it, maybe its access or owner of your $HOME/.npm/ folder
composer self-update
dev switch lavasuite
```

## WSL-Linux / Windows Host interoperability

you can access your windows host file system from inside the WSL via e.g.:

```bash
$ ls -l /mnt/c/Users/hoffmannd/Nextcloud/TeamFolders/Lavasuite/
# convert a linux style path to windows style path and vice versa:
$ wslpath -a -w /mnt/c/Users/hoffmannd/Nextcloud/TeamFolders/Lavasuite/
C:\Users\hoffmannd\Nextcloud\TeamFolders\Lavasuite
$ wslpath -a -u 'C:\Users\hoffmannd\Nextcloud\TeamFolders\Lavasuite'
/mnt/c/Users/hoffmannd/Nextcloud/TeamFolders/Lavasuite
# you can use this to mv or cp files vice versa
```

## using ~/.bash_configs

```bash
git clone https://scm.liongate.cloud:443/scm/repo/Intern/bash_configs ~/.bash_configs
~/.bash_configs/0initialInstall.sh
source ~/.bashrc
```

## prerequisites

### jq

apt-get update && apt-get install -y jq
### git

git version > 2.30.x

```bash
# if still on debian buster (10)
# sudo apt-get -t buster-backports install git
```

be sure you `$HOME/.gitconfig` has the correct autocrlf settings:

```bash
# on &nbsp;&nbsp;&nbsp; WINDOWS HOST!!! &nbsp;&nbsp;&nbsp; (NOT inside the wsl!):
git config --global core.autocrlf true

# inside &nbsp;&nbsp;&nbsp; WSL/LINUX &nbsp;&nbsp;&nbsp; machine
git config --global core.autocrlf input
```

be sure to have configured your git with some minimal global config (`~/.gitconfig`)<br/>
and be sure to just have EITHER of:

```bash
# on windows host (NOT inside the wsl!):
git config --global core.autocrlf true

# on wsl/linux machine
git config --global core.autocrlf input
```

```bash
git config --global user.name "Your Fullname"
git config --global user.email "Your@Email.de"
git config --global credential.helper "cache --timeout=3600"
git config --global submodule.recurse true
git config --global status.submodulesummary true
git config --global diff.submodule log
git config --global pull.rebase true
git config --global core.autocrlf input # use true instead of input if on the windows host and not inside the WSL/LINUX
```

make sure you have also other basic mandatory git config configs set, e.g. by editing your `~/.gitconfig` file.<br/>
an extensive example can be found under: https://scm.liongate.cloud/scm/repo/Intern/bash_configs/code/sources/master/gitconfigExample/

check your `~/.gitconfig` file, you need at a minimum:

```ini
[user]
        name = Your Dr. Fullname
        email = your.email@liongate.de
[credential]
        helper = cache --timeout=3600
[submodule]
        recurse = true
[status]
        submodulesummary = true
[pull]
        rebase = true
[core]
        autocrlf = input # use true instead of input if on the windows host and not inside the WSL/LINUX
```

### php

```bash
php --version
# if not version correct  version
phpversion=8.0
# wget -q https://packages.sury.org/php/apt.gpg -O- | sudo apt-key add -
# echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list
# apt-get update
apt-get install -y php${phpversion} phpunit php${phpversion}-apcu php${phpversion}-gmp php${phpversion}-igbinary php${phpversion}-imagick php${phpversion}-imap php${phpversion}-memcached php${phpversion}-msgpack php${phpversion}-redis php${phpversion}-xdebug php${phpversion}-bcmath php${phpversion}-fpm php${phpversion}-xml php${phpversion}-mysql php${phpversion}-zip php${phpversion}-intl php${phpversion}-ldap php${phpversion}-gd php${phpversion}-cli php${phpversion}-bz2 php${phpversion}-curl php${phpversion}-mbstring php${phpversion}-opcache php${phpversion}-soap php${phpversion}-cgi
```

```bash
mkdir -p /usr/local/bin && echo "installing composer to /usr/local/bin" \
 && EXPECTED_CHECKSUM="$(php -r 'copy("https://composer.github.io/installer.sig", "php://stdout");')" \
 && php -r "copy('https://getcomposer.org/installer', '/root/composer-setup.php');" \
 && ACTUAL_CHECKSUM="$(php -r "echo hash_file('sha384', '/root/composer-setup.php');")" \
 && if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]; then >&2 echo 'ERROR: Invalid installer checksum' ; rm /root/composer-setup.php ; return ; fi \
 && php /root/composer-setup.php --install-dir=/usr/local/bin --filename=composer \
 && rm /root/composer-setup.php
 ```

### nodejs angular npm

```bash
nodeversion=16
nodeversionalias=lts/gallium
angularversion=13
curl -fsSL https://deb.nodesource.com/setup_${nodeversion}.x | bash - && apt-get install -y nodejs

npm install -g @angular/cli@${angularversion}
```

### dev new|switch lavasuite

```shell
  dev upgrade
  if ls /etc/liongate/dev/lavasuite >/dev/null 2>&1 ; then
    echo "ok you already have a liongate dev setup for lavasuite"
  else
    echo "no liongate dev setup found for lavasuite"
    dev new lavasuite # remenber lavasuite does not(!) use the default created /etc/apache2/sites-available/lavasuite.conf (see below!)
    dev switch lavasuite
  fi
```

## dev env root directory

The root directory of your lavasuite dev env should exist now and is `/var/www/lavasuite/`


### prepare bash environment and git clone lavasuite

```bash
if [[ ! -d /var/www/lavasuite ]]; then mkdir /var/www/lavasuite ; fi
cd /var/www/lavasuite
git clone https://scm.liongate.cloud:443/scm/repo/otelo/lava-api-portal LavaApiPortal
```

once you have cloned `LavaApiPortal` you can use the `tools/doInAllLavaGitDirs.sh` helper script inside it<br/>
a) to get all other necessary git repos cloned
b) set up [LavaApiPortal environment variables](#lavaapiportal-environment-variables)
c) to operate on all or certain repos using just one command (use `/var/www/lavasuite/LavaApiPortal/tools/doInAllLavaGitDirs.sh` to see all available commands)

So to get alle necessary git repos and update them to the latest version, use<br/>
`/var/www/lavasuite/LavaApiPortal/tools/doInAllLavaGitDirs.sh pull`

To fetch all the repos and see if they are up-to-date you can use<br/>
`/var/www/lavasuite/LavaApiPortal/tools/doInAllLavaGitDirs.sh status`<br/>
(it is a good idea to run this command several times a day, to see if somebody else pushed something meanwhile)

### Generate openApi code from openApi yml definitions

make sure you at least once did a `composer install` in `/var/www/lavasuite/LavaApiPortal`.

get the openApi Generator API for Liongate from [https://github.com/LionGate-AG/openapi-generator/releases](https://github.com/LionGate-AG/openapi-generator/releases)<br/>
and put the jar into `/var/www/lavasuite/`, e.g. by copying (see beginning of this documentation) or<br/>

```bash
gitUser="LionGate-AG"
gitRepo="openapi-generator"
version="$(curl -sL https://api.github.com/repos/${gitUser}/${gitRepo}/releases/latest | jq -r '.tag_name')"
url=$(curl -sL https://api.github.com/repos/${gitUser}/${gitRepo}/releases/latest | jq -r ".assets[].browser_download_url")
set -x
wget $url -O /var/www/lavasuite/openapi-generator-cli.jar
set +x
```

install java if not already there:

```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

sdk list java | cat
# use the latest java temurin 11 version
sdk install java 11.x.x-tem
java --version
```

using the `openapi-generator-cli.jar` (and having java installed) generate the openApi classes with:

```bash
doInAllLavaGitDirs generate
```

## lavasuite-cicd/setupLavasuite/setupLavasuite.sh

This is the script that originally was intended to do all of the below... give it a try!<br />
You only need stuff from below if something didn't work out.

but you will need some files (as described below) from nextcloud fileshare <https://liongate.cloud/apps/files/?dir=/TeamFolders/Lavasuite&fileid=1132952>

```bash
cd /var/www/lavasuite/lavasuite-cicd/setupLavasuite
./setupLavasuite.sh
```

## setup database and DB schema migration and DB data seeding

legacy DB Dumpfiles for seeding should be in Nextcloud under `ll /mnt/c/Users/<winuser>/Nextcloud/TeamFolders/Lavasuite/legacyXLogic/xlogic_*.sql`<br/>
copy the latest version of any db dump needed to `/var/www/lavasuite/` by e.g.:
`cp /mnt/c/Users/<winuser>/Nextcloud/TeamFolders/Lavasuite/legacyXLogic/xlogic_*.sql /var/www/lavasuite/`

make sure you only have ONE file for each legacy DB (winback, kulanz, etc.)

### prepare Mysql Database

mysql command line cli:

```bash
mysql -u root
# or
mysql -u lavasuite -p
```

might be you have to alter the root user if you run into problems:

```sql
-- you only have to do these if you run into problems with mysql
ALTER USER 'root'@'localhost' IDENTIFIED VIA mysql_native_password USING '' OR unix_socket;
UPDATE USER SET authentication_string = PASSWORD('new_password') WHERE User = 'root' AND Host = 'localhost';
-- this one fixed it for me
update mysql.user set plugin = 'mysql_native_password' where User='root';
FLUSH PRIVILEGES;
```

Open mysql and create a new user

```sql
-- use the team password as password and not literal 'password' !!!
CREATE USER 'lavasuite'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON * . * TO 'lavasuite'@'localhost';
FLUSH PRIVILEGES;
```

if you have your database dump files in `/var/www/lavasuite/..sql` you also can try to use</br>
`/var/www/lavasuite/lavasuite-cicd/setupLavasuite/resetDBs.sh`

and for being safe run the migrations again:

`php artisan migrate`

and

`php artisan module:seed Tasklist --class=AddFakeTaskSeeder`

#### exporting mysql database to file (if needed)

```shell
mysqldump -u root -p <DBNAME> > ./dbdumpfile.sql
```

#### importing mysql database from file (if needed)

```shell
mysql -u root -p <DBNAME> < ./dbdumpfile.sql
```

## build

### Frontend UI

```bash
/var/www/lavasuite/LavaApiPortal/tools/doInAllLavaGitDirs.sh generate
/var/www/lavasuite/LavaApiPortal/tools/doInAllLavaGitDirs.sh npminstall
/var/www/lavasuite/LavaApiPortal/tools/doInAllLavaGitDirs.sh npmbuild
```

### Backend

```bash
/var/www/lavasuite/LavaApiPortal/tools/doInAllLavaGitDirs.sh generate
/var/www/lavasuite/LavaApiPortal/tools/doInAllLavaGitDirs.sh composerinstall
```

## apache2

***apache2 has to run to serve the php api backend!!!***

please be aware that lavasuite does NOT use the default apa
che2 conf that LG dev script will provide, but the one from:

```bash
# also note the things in LavaApiPortal/LavasuiteConfig/apache2/README.md
cp /var/www/lavasuite/LavaApiPortal/LavasuiteConfig/apache2/lavasuite.conf /etc/apache2/sites-available/lavasuite.conf
```

on restart of you computer or the wsl, the wsl might leave apache2 stopped on restart/relogin!<br/>
in that case you either might `dev switch lavasuite` or `sourcebash` `source $HOME/.bashrc` or `source $HOME/.bash_configs/bash_lava`

or manually

```shell
sudo service apache2    status
sudo service php7.4=fpm status
sudo service mysql      status

sudo service apache2    start # restart / reload
sudo service php7.4-fpm start # restart / reload
sudo service mysql      start # restart / reload
```

### XDEBUG Settings

With the LG DEV, we should be able to have already all XDBUG related installations.

In order to use for the development & debug with vscode we need to apply the following recommended settings.

#### PHP CLI Settings

```ini
[xdebug]
xdebug.mode = debug
xdebug.start_with_request = trigger
xdebug.client_port = 9001

and run with
XDEBUG_TRIGGER=1 php <script> .
```

#### PHP FPM Settings

```ini
[xdebug]
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.discover_client_host = yes
xdebug.client_port = 9000
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
```

## VSCODE Settings

```json
{
   "name": "PHP Listen for Xdebug on e.g. /etc/php/8.0/fpm/* xdebug.client_port",
   "type": "php",
   "request": "launch",
   "port": 9000,
   "runtimeExecutable": "/usr/bin/php"
},
{
  "name": "PHP Listen for Xdebug on e.g. /etc/php/8.0/cli/* xdebug.client_port",
  "type": "php",
  "request": "launch",
  "port": 9001
},
{
   "name": "PHP Launch script currently open on e.g. /etc/php/8.0/cli/* xdebug.client_port",
   "type": "php",
   "request": "launch",
   "program": "${file}",
   "cwd": "${fileDirname}",
   "port": 9003,
   "runtimeArgs": [ "XDEBUG_TRIGGER=1"],
    "env": {
      "XDEBUG_MODE": "debug,develop",
      "XDEBUG_CONFIG": "client_port=${port}"
    }
}
```

## SAML Authentication and Authorisation

Login via <https://iam.liongate.cloud/auth/module.php/core/authenticate.php?as=LionGate-Managed-Services>

*TL;DR*

relevant files and links:

- `/etc/liongate/dev-packages/simplesaml/etc/`
- `/etc/liongate/dev-packages/simplesaml/etc/config/authsources.d/010-authsource-DEV-IDP-MAIN.php`
- `/etc/liongate/dev-packages/simplesaml/etc/metadata/saml20-idp-hosted.d/010-saml20-idp-hosted-LG-DEV-IDP-MAIN.php`
- `/etc/liongate/dev-packages/simplesaml/etc/metadata/saml20-sp-remote.d/020-saml20-sp-remote-LAVASUITE-DEV-SP.php`
- <http://localhost/api/saml2/lavasuite/metadata>
- <http://localhost/simplesaml/admin/metadata-converter.php>
- <https://iam.liongate.cloud/auth/module.php/core/authenticate.php?as=LionGate-Managed-Services>

### Setup

Using SimpleSANMLphp, which Liongate also uses for its own iam.liongate.cloud.

Installation will come automatically with the "dev" script version >1.2.0 (dev version)

as root user run:

```bash
dev upgrade
dev install simplesaml
```

After having done this a complete SAML2.0 IdP based on SimpleSAMLphp (Version >1.19.5) will be available in the WSL for local dev usage.

reachable at <http://localhost/simplesaml/>

"Admin-Passwort" found at the same location as the other dev passwords.

Configuration files are located under `/etc/liongate/dev-packages/simplesaml/etc/`

Further information you can get on the homepage of SimpleSAMLphp.

Be sure to ***NOT COMMIT ANY Security relevant things into git repos!!!***</br>
especially no Certificates (even if self-signed)

**.env**

- Define following .env properties. Here the keys are based on the idpName which is defined in the lavasuite_idp_settings.php file.

```shell
# SAML Paramaters
SAML2_PROXY_VARS=false

# SAML SP Paramaters
SAML2_LAVASUITE_SP_x509="file:///etc/liongate/dev-packages/simplesaml/etc/cert/dev.liongate.de.crt"
SAML2_LAVASUITE_SP_PRIVATEKEY="file:///etc/liongate/dev-packages/simplesaml/etc/cert/dev.liongate.de.pem"
SAML2_LAVASUITE_SP_ENTITYID="LAVASUITE-DEV-SP"

# SAML IDP Paramaters
SAML2_LAVASUITE_IDP_HOST="http://localhost/simplesaml"
SAML2_LAVASUITE_IDP_x509="file:///etc/liongate/dev-packages/simplesaml/etc/cert/dev.liongate.de.crt"
SAML2_LAVASUITE_IDP_ENTITYID="LG-DEV-IDP-MAIN"
SAML2_LAVASUITE_AUTH_REQUESTS_SIGNED=false
```

### local SAML configuration inside WSL

Once all above changes are completed you have to create/edit your local SAML configuration files:

1. You get the base saml configuration (which you will have to alter afterwards!!!) via `http://localhost/api/saml2/lavasuite/metadata`
2. Leave this Browser-Tab open and open a new one for converting the xml for the SP registration: `http://localhost/simplesaml/admin/metadata-converter.php`
   - Login is the WSL SimpleSAML password (shared via Nextcloud password manager)
3. copy the returned XML data of step 1. into the XML metadata textfield of Step 2. and hit the "Parse" Button
4. copy the translated php code and create/replace the content of the file with it:<br/>
   `/etc/liongate/dev-packages/simplesaml/etc/metadata/saml20-sp-remote.d/020-saml20-sp-remote-LAVASUITE-DEV-SP.php`
5. make sure that file's first line is `<?php`
6. make sure that you delete the line with  `expire' => 1******,` in that file (which if deleted will avoid metadata expiration checks)

Create/Edit the file:<br/>
`/etc/liongate/dev-packages/simplesaml/etc/metadata/saml20-idp-hosted.d/010-saml20-idp-hosted-LG-DEV-IDP-MAIN.php`<br/>
and make sure the file has `$metadata['LG-DEV-IDP-MAIN'] = [` at the start and the `'auth' => 'AS-DEV-IDP-MAIN'` parameter.

```php
<?php
/**
 * SAML 2.0 IdP configuration for SimpleSAMLphp.
 *
 * See: https://simplesamlphp.org/docs/stable/simplesamlphp-reference-idp-hosted
 */

$metadata['LG-DEV-IDP-MAIN'] = [
    /*
     * The hostname of the server (VHOST) that will use this SAML entity.
     *
     * Can be '__DEFAULT__', to use this entry by default.
     */
    'host' => '__DEFAULT__',

    // X.509 key and certificate. Relative to the cert directory.
    'privatekey' => 'dev.liongate.de.pem',
    'certificate' => 'dev.liongate.de.crt',

    /*
     * Authentication source to use. Must be one that is configured in
     * 'config/authsources.php'.
     */
    'auth' => 'AS-DEV-IDP-MAIN',

    /* Uncomment the following to use the uri NameFormat on attributes. */
    /*
    'attributes.NameFormat' => 'urn:oasis:names:tc:SAML:2.0:attrname-format:uri',
    'authproc' => [
        // Convert LDAP names to oids.
        100 => ['class' => 'core:AttributeMap', 'name2oid'],
    ],
    */

    /*
     * Uncomment the following to specify the registration information in the
     * exported metadata. Refer to:
     * http://docs.oasis-open.org/security/saml/Post2.0/saml-metadata-rpi/v1.0/cs01/saml-metadata-rpi-v1.0-cs01.html
     * for more information.
     */
    /*
    'RegistrationInfo' => [
        'authority' => 'urn:mace:example.org',
        'instant' => '2008-01-17T11:28:03Z',
        'policies' => [
            'en' => 'http://example.org/policy',
            'es' => 'http://example.org/politica',
        ],
    ],
    */
];
```

now if you login via Liongate SAML at:

<https://iam.liongate.cloud/auth/module.php/core/authenticate.php?as=LionGate-Managed-Services>

and scroll down, you should see (as a new/last table row between `email:` and `SAML Subject` Heading) something alike (probably more):<br/>
(if you're not seeing these reach out to liongate system-administrators that they add you to the developers' FACTS groups)

```text
applicationRoles
APP-DE-LG_LV_P_POW_v
APP-DE-LG_LV_I_POW_v
APP-DE-LG_SC_P_POW_v
APP-DE-LG_SC_I_POW_v
APP-DE-LG_W_P_POW_v
APP-DE-LG_W_I_POW_v
APP-DE-LG_NBA_P_Lesen_v
APP-DE-LG_NBA_P_IT_POW_v
APP-DE-LG_NBA_I_Lesen_v
APP-DE-LG_NBA_I_IT_POW_v
```

### adding Lavasuite access rules to your user

edit `/etc/liongate/dev-packages/simplesaml/etc/config/authsources.d/010-authsource-DEV-IDP-MAIN.php`

And make sure that all needed ```lvApplication``` are set correctly

```php
<?php
$config['AS-DEV-IDP-MAIN'] = [
    'exampleauth:UserPass',
    'masterUser:password' => [
        'uid' => ['nbaread@dev.liongate.local'],
        'sumsAlias' => ['winback_tl_s', 'winback_bo_s', 'nbawrite', 'nbaread', 'lavaagent_a', 'lavaagent_c'],
        'FirstName' => ['Master'],
        'LastName' => ['User'],
        'Email' => ['master.user@dev.liongate.local'],
        'lvRoles' => ['WINBACK_BACKOFFICE', 'WINBACK_TEAMLEAD', 'SERVICE_AGENT','ADMIN','SUPERVISOR'],
        'lvPartner' => ['Vodafone', 'Sparhandy'],
        'lvApplications' => ['SALES_CLIENT', 'NBA', 'KULANZ', 'LAVA', 'WINBACK' ],
        'lvApplicationRoles' => ['WINBACK.WINBACK_TEAMLEAD', 'WINBACK.WINBACK_AGENT', 'SALE_CLIENT.ADMIN', 'NBA.ADMIN', 'LAVA.SERVICE_AGENT', 'LAVA.SUPERVISOR']
    ],

    // Give the user an option to save their username for future login attempts
    // And when enabled, what should the default be, to save the username or not
    //'remember.username.enabled' => false,
    //'remember.username.checked' => false,
    // User for NBA READ Role                                                              
    'nbaread:password' => [
        'uid' => ['nbaread@dev.liongate.local'],
        'sumsAlias' => ['nbaread'],
        'FirstName' => ['Roger'],
        'LastName' => ['Reader'],
        'Email' => ['roger.reader@dev.liongate.local'],
        'lvRoles' => ['READ'],                              
        'lvPartner' => ['Vodafone'],
        'lvApplications' => ['NBA'],
        'lvStage' => ['PROD'],
        'lvApplicationRoles' => ['NBA.READ'],
    ],
    // User for NBA Write Role
    'nbawrite:password' => [
        'uid' => ['nbawrite@dev.liongate.local'],
        'sumsAlias' => ['nbawrite'],
        'FirstName' => ['Warren'],
        'LastName' => ['Writer'],
        'Email' => ['warren.writer@dev.liongate.local'],
        'lvRoles' => ['IT_POW_v'],
        'lvPartner' => ['Vodafone'],
        'lvApplications' => ['NBA'],
        'lvStage' => ['PROD'],
        'lvApplicationRoles' => ['NBA.IT_POW_v']
    ],
    // User for LAVA SERVICE_AGENT Role of Concentrix
    'lava.agent.con:password' => [
        'uid' => ['lava.agent.con@dev.liongate.local'],
        'sumsAlias' => ['lavaagent_c'],
        'FirstName' => ['Agent'],
        'LastName' => ['Convergys'],
        'Email' => ['lava.agent.con@dev.liongate.local'],
        'lvRoles' => ['SERVICE_AGENT'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['LAVA'],
        'lvStage' => ['INT'],
        'lvApplicationRoles' => ['LAVA.SERVICE_AGENT']
    ],
    // User for LAVA SUPERVISOR Role of Concentrix
    'lava.tl.con:password' => [
        'uid' => ['lava.tl.con@dev.liongate.local'],
        'sumsAlias' => ['lavatl_c'],
        'FirstName' => ['Teamlead'],
        'LastName' => ['Convergys'],
        'Email' => ['lava.tl.con@dev.liongate.local'],
        'lvRoles' => ['SUPERVISOR'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['LAVA'],
        'lvStage' => ['INT'],
        'lvApplicationRoles' => ['LAVA.SUPERVISOR']
    ],
    // User for LAVA SERVICE_AGENT Role of Arvato
    'lava.agent.arv:password' => [
        'uid' => ['lava.agent.arv@dev.liongate.local'],
        'sumsAlias' => ['lavaagent_a'],
        'FirstName' => ['Agent'],
        'LastName' => ['Arvato'],
        'Email' => ['lava.agent.arv@dev.liongate.local'],
        'lvRoles' => ['SERVICE_AGENT'],
        'lvPartner' => ['Arvato'],
        'lvApplications' => ['LAVA'],
        'lvStage' => ['INT'],
        'lvApplicationRoles' => ['LAVA.SERVICE_AGENT']
    ],
    // User for LAVA SUPERVISOR Role of Arvato
    'lava.tl.arv:password' => [
        'uid' => ['lava.tl.arv@dev.liongate.local'],
        'sumsAlias' => ['lavatl_a'],
        'FirstName' => ['Teamlead'],
        'LastName' => ['Arvato'],
        'Email' => ['lava.tl.arv@dev.liongate.local'],
        'lvRoles' => ['SUPERVISOR'],
        'lvPartner' => ['Arvato'],
        'lvApplications' => ['LAVA'],
        'lvStage' => ['INT'],
        'lvApplicationRoles' => ['LAVA.SUPERVISOR']
    ],
    // User for LAVA SERVICE_AGENT Role of Vodafone
    'lava.agent.voda:password' => [
        'uid' => ['lava.agent.voda@dev.liongate.local'],
        'sumsAlias' => ['lavaagent_v'],
        'FirstName' => ['Agent'],
        'LastName' => ['Vodafone'],
        'Email' => ['lava.agent.voda@dev.liongate.local'],
        'lvRoles' => ['SERVICE_AGENT'],
        'lvPartner' => ['Vodafone'],
        'lvApplications' => ['LAVA'],
        'lvStage' => ['INT'],
        'lvApplicationRoles' => ['LAVA.SERVICE_AGENT']
    ],
    // User for LAVA SUPERVISOR Role of Vodafone
    'lava.tl.voda:password' => [
        'uid' => ['lava.tl.voda@dev.liongate.local'],
        'sumsAlias' => ['lavatl_v'],
        'FirstName' => ['Teamlead'],
        'LastName' => ['Vodafone'],
        'Email' => ['lava.tl.voda@dev.liongate.local'],
        'lvRoles' => ['SUPERVISOR'],
        'lvPartner' => ['Vodafone'],
        'lvApplications' => ['LAVA'],
        'lvStage' => ['INT'],
        'lvApplicationRoles' => ['LAVA.SUPERVISOR']
    ],
    
    /**
    * SALES CLIENT: ROLES of Vodafone
    */
    // User for Operation Role of Vodafone
    'sales.op:password' => [
        'uid' => ['sales.op@dev.liongate.local'],
        'sumsAlias' => ['sales_op_v'],
        'FirstName' => ['Operation'],
        'LastName' => ['Vodafone'],
        'Email' => ['sales.op@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['READ'],
        'lvPartner' => ['Vodafone'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.READ'],
        'lvStage' => ['INT']
    ],
    // User for Marketing Role of Vodafone
    'sales.market:password' => [
        'uid' => ['sales.market@dev.liongate.local'],
        'sumsAlias' => ['sales_market_v'],
        'FirstName' => ['Marketing'],
        'LastName' => ['Vodafone'],
        'Email' => ['sales.market@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['MARKETING'],
        'lvPartner' => ['Vodafone'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.MARKETING'],
        'lvStage' => ['INT']
    ],
    // User for SOX Agent Role of Vodafone
    'sales.sox:password' => [
        'uid' => ['sales.sox@dev.liongate.local'],
        'sumsAlias' => ['sales_sox_v'],
        'FirstName' => ['Sox'],
        'LastName' => ['Vodafone'],
        'Email' => ['sales.sox@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['SOX_AGENT'],
        'lvPartner' => ['Vodafone'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.SOX_AGENT'],
        'lvStage' => ['INT']
    ],
    // User for Storno Agent Role of Vodafone
    'sales.storno:password' => [
        'uid' => ['sales.storno@dev.liongate.local'],
        'sumsAlias' => ['sales_storno_v'],
        'FirstName' => ['Strono'],
        'LastName' => ['Vodafone'],
        'Email' => ['sales.storno@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['STORNO_AGENT'],
        'lvPartner' => ['Vodafone'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.STORNO_AGENT'],
        'lvStage' => ['INT']
    ],
    
    /**
    * Sales CLIENT: ROLES of Arvato
    */
    // User for SOX Agent Role of Arvato
    'sales.sox.av:password' => [
        'uid' => ['sales.sox.av@dev.liongate.local'],
        'sumsAlias' => ['sales_sox.av_v'],
        'FirstName' => ['Sox'],
        'LastName' => ['Arvato'],
        'Email' => ['sales.sox.av@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['SOX_AGENT'],
        'lvPartner' => ['Arvato'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.SOX_AGENT'],
        'lvStage' => ['INT']
    ],
    // User for Storno Agent Role of Arvato
    'sales.storno.av:password' => [
        'uid' => ['sales.storno.av@dev.liongate.local'],
        'sumsAlias' => ['sales_storno.a'],
        'FirstName' => ['Strono'],
        'LastName' => ['Arvato'],
        'Email' => ['sales.storno.av@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['STORNO_AGENT'],
        'lvPartner' => ['Arvato'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.STORNO_AGENT'],
        'lvStage' => ['INT']
    ],
    
    /**
    * WINBACK CLIENT: WINBACK ROLES of Sparhandy
    */
    
    // User for Winback Agent Role of Sparhandy
    'winback.ag.spar:password' => [
        'uid' => ['winback.ag.spar@dev.liongate.local'],
        'sumsAlias' => ['winback_ag_s'],
        'FirstName' => ['Winback Agent'],
        'LastName' => ['Sparhandy'],
        'Email' => ['winback.ag.spar@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['WINBACK_AGENT'],
        'lvPartner' => ['Sparhandy'],
        'lvApplications' => ['WINBACK'],
        'lvApplicationRoles' => ['WINBACK.WINBACK_AGENT'],
        'lvStage' => ['INT']
    ],
    
    // User for Winback Agent+Teamlead Role of Sparhandy
    'winback.tl.spar:password' => [
        'uid' => ['winback.tl.spar@dev.liongate.local'],
        'sumsAlias' => ['winback_tl_s'],
        'FirstName' => ['Winback Teamlead'],
        'LastName' => ['Sparhandy'],
        'Email' => ['winback.tl.spar@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['WINBACK_TEAMLEAD', 'WINBACK_AGENT'],
                                      
        'lvPartner' => ['Sparhandy'],
        'lvApplications' => ['WINBACK'],
        'lvApplicationRoles' => ['WINBACK.WINBACK_TEAMLEAD', 'WINBACK.WINBACK_AGENT'],
        'lvStage' => ['INT']
    ],
    
    // User for Winback Backoffice Role of Sparhandy
    'winback.bo.spar:password' => [
        'uid' => ['winback.bo.spar@dev.liongate.local'],
        'sumsAlias' => ['winback_bo_s'],
        'FirstName' => ['Chrun Backoffice'],
        'LastName' => ['Sparhandy'],
        'Email' => ['winback.bo.spar@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
                     
        'lvRoles' => ['WINBACK_BACKOFFICE', 'WINBACK_TEAMLEAD', 'WINBACK_AGENT'],
        'lvPartner' => ['Sparhandy'],
        'lvApplications' => ['WINBACK'],
        'lvApplicationRoles' => ['WINBACK.WINBACK_BACKOFFICE', 'WINBACK.WINBACK_TEAMLEAD', 'WINBACK.WINBACK_AGENT'],
        'lvStage' => ['INT']
    ],
    
    /**
    * WINBACK CLIENT: CHURN ROLES of Sparhandy
    */
    
    // User for Churn Agent Role of Sparhandy
    'churn.ag.spar:password' => [
        'uid' => ['churn.ag.spar@dev.liongate.local'],
        'sumsAlias' => ['churn_ag_s'],
        'FirstName' => ['Chrun Agent'],
        'LastName' => ['Sparhandy'],
        'Email' => ['churn.ag.spar@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
                     
        'lvRoles' => ['CHURN_AGENT'],
        'lvPartner' => ['Sparhandy'],
        'lvApplications' => ['WINBACK'],
        'lvApplicationRoles' => ['WINBACK.CHURN_AGENT'],
        'lvStage' => ['INT']
    ],
    
    // User for Churn Agent+Teamlead Role of Sparhandy
    'churn.tl.spar:password' => [
        'uid' => ['churn.tl.spar@dev.liongate.local'],
        'sumsAlias' => ['churn_tl_s'],
        'FirstName' => ['Chrun Teamlead'],
        'LastName' => ['Sparhandy'],
        'Email' => ['churn.tl.spar@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['CHURN_TEAMLEAD', 'CHURN_AGENT'],
        'lvPartner' => ['Sparhandy'],
        'lvApplications' => ['WINBACK'],
        'lvApplicationRoles' => ['WINBACK.CHURN_TEAMLEAD', 'WINBACK.CHURN_AGENT'],
        'lvStage' => ['INT']
    ],
    
    // User for Churn Backoffice Role of Sparhandy
    'churn.bo.spar:password' => [
        'uid' => ['churn.bo.spar@dev.liongate.local'],
        'sumsAlias' => ['churn_bo_s'],
        'FirstName' => ['Chrun Backoffice'],
        'LastName' => ['Sparhandy'],
        'Email' => ['churn.bo.spar@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['CHURN_BACKOFFICE', 'CHURN_TEAMLEAD', 'CHURN_AGENT'],
        'lvPartner' => ['Sparhandy'],
        'lvApplications' => ['WINBACK'],
        'lvApplicationRoles' => ['WINBACK.CHURN_BACKOFFICE', 'WINBACK.CHURN_TEAMLEAD', 'WINBACK.CHURN_AGENT'],
        'lvStage' => ['INT']
    ],
    
    
    /**
    * SALES CLIENT: Winback ROLES of Concentrix
    */    
    // User for Winback Agent Role of Concentrix
    'winback.ag.con:password' => [
        'uid' => ['winback.ag.con@dev.liongate.local'],
        'sumsAlias' => ['winback_ag_c'],
        'FirstName' => ['Winback Agent'],
        'LastName' => ['Concentrix'],
        'Email' => ['winback.ag.con@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
                     
        'lvRoles' => ['WINBACK_AGENT'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.WINBACK_AGENT'],
        'lvStage' => ['INT']
    ],
    
    // User for Winback Agent+Teamlead Role of Concentrix
    'winback.tl.con:password' => [
        'uid' => ['winback.tl.con@dev.liongate.local'],
        'sumsAlias' => ['winback_tl_c'],
        'FirstName' => ['Winback Teamlead'],
        'LastName' => ['Concentrix'],
        'Email' => ['winback.tl.con@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['WINBACK_TEAMLEAD', 'WINBACK_AGENT'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.WINBACK_TEAMLEAD', 'SALES_CLIENT.WINBACK_AGENT'],
        'lvStage' => ['INT']
    ],
    
    // User for Winback Backoffice Role of Concentrix
    'winback.bo.con:password' => [
        'uid' => ['winback.bo.con@dev.liongate.local'],
        'sumsAlias' => ['winback_bo_c'],
        'FirstName' => ['Chrun Backoffice'],
        'LastName' => ['Concentrix'],
        'Email' => ['winback.bo.con@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['WINBACK_BACKOFFICE', 'WINBACK_TEAMLEAD', 'WINBACK_AGENT'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.WINBACK_BACKOFFICE', 'SALES_CLIENT.WINBACK_TEAMLEAD', 'SALES_CLIENT.WINBACK_AGENT'],
        'lvStage' => ['INT']
    ],
    
    /**
    * SALES CLIENT: CHURN ROLES of Concentrix
    */    
    // User for Churn Agent Role of Concentrix
    'churn.ag.con:password' => [
        'uid' => ['churn.ag.con@dev.liongate.local'],
        'sumsAlias' => ['churn_ag_c'],
        'FirstName' => ['Chrun Agent'],
        'LastName' => ['Concentrix'],
        'Email' => ['churn.ag.con@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['CHURN_AGENT'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.CHURN_AGENT'],
        'lvStage' => ['INT']
    ],
    
    // User for Churn Agent+Teamlead Role of Concentrix
    'churn.tl.con:password' => [
        'uid' => ['churn.tl.con@dev.liongate.local'],
        'sumsAlias' => ['churn_tl_c'],
        'FirstName' => ['Chrun Teamlead'],
        'LastName' => ['Concentrix'],
        'Email' => ['churn.tl.con@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['CHURN_TEAMLEAD', 'CHURN_AGENT'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.CHURN_TEAMLEAD', 'SALES_CLIENT.CHURN_AGENT'],
        'lvStage' => ['INT']
    ],
    
    // User for Churn Backoffice Role of Concentrix
    'churn.bo.con:password' => [
        'uid' => ['churn.bo.con@dev.liongate.local'],
        'sumsAlias' => ['churn_bo_c'],
        'FirstName' => ['Chrun Backoffice'],
        'LastName' => ['Concentrix'],
        'Email' => ['churn.bo.con@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['CHURN_BACKOFFICE', 'CHURN_TEAMLEAD', 'CHURN_AGENT'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.CHURN_BACKOFFICE', 'SALES_CLIENT.CHURN_TEAMLEAD', 'SALES_CLIENT.CHURN_AGENT'],
        'lvStage' => ['INT']
    ],
    
    
    // User for Storno Agent Role of Concentrix
    'sales.storno.con:password' => [
        'uid' => ['sales.storno.con@dev.liongate.local'],
        'sumsAlias' => ['sales_storno_c'],
        'FirstName' => ['Strono'],
        'LastName' => ['Concentrix'],
        'Email' => ['sales.storno.con@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['STORNO_AGENT'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.STORNO_AGENT'],
        'lvStage' => ['INT']
    ],    
    // User for SOX Agent Role of Concentrix
    'sales.sox.con:password' => [
        'uid' => ['sales.sox.con@dev.liongate.local'],
        'sumsAlias' => ['sales_sox_c'],
        'FirstName' => ['Sox'],
        'LastName' => ['Concentrix'],
        'Email' => ['sales.sox.con@dev.liongate.local'],
        'applicationRoles' => ['LG-----'],
        'lvRoles' => ['SOX_AGENT'],
        'lvPartner' => ['Concentrix'],
        'lvApplications' => ['SALES_CLIENT'],
        'lvApplicationRoles' => ['SALES_CLIENT.SOX_AGENT'],
        'lvStage' => ['INT']
    ],
];
```

The following fields are defined in the IAM which considers the Mappings for the NGUM related parameters 

| NGUM Application |   lvApplications   |
|------------------|:------------------:|
| CHK              |   CHECK_MANAGER    |
| LV               |        LAVA        |
| LVCOM            | LAVA_COMMUNICATION |
| LVTELE           |   LAVA_TELESALES   |
| SC               |    SALES_CLIENT    |
| W                |      WINBACK       |
| NBA              |        NBA         |



| NGUM Partners | lvPartner  |
|---------------|:----------:|
| c             | Concentrix |
| m             |  Majorel   |
| v             | Vodafone   |
| a             |   Arvato   |
| k             |   Kairo    |
| l             |  Logitel   |
| s             | Sparhandy  |
| p             | Preisbörse |


| NGUM Roles |    lvRoles    |
|------------|:-------------:|
| AG_c       | SERVICE_AGENT |
| AG_m       | SERVICE_AGENT |
| AG_v       | SERVICE_AGENT |
| BEWT_a     | BEW_TEAM_LEAD |
| BEWT_c     | BEW_TEAM_LEAD |
| C_AG_k     | SERVICE_AGENT |
| C_BO_Rp_v  |  BACKOFFICE   |
| C_TL_k     |   TEAM_LEAD   |
| ED_c       |    EDITOR     |
| ED_m       |    EDITOR     |
| FIN_a      | FINANCE_AGENT |
| GST_a      |    EDITOR     |
| GST_c      |    EDITOR     |
| Lesen_v    |     READ      |
| MNP_a      |   MNP_DESK    |
| POW_v      |     ADMIN     |
| SCA_a      | SERVICE_AGENT |
| SCA_c      | SERVICE_AGENT |
| SCA_v      | SERVICE_AGENT |
| SV_a       |  SUPERVISOR   |
| SV_AG_c    |  SUPERVISOR   |
| SV_AG_v    |  SUPERVISOR   |
| SV_v       |  SUPERVISOR   |
| TL_c       |   TEAM_LEAD   |
| TL_m       |   TEAM_LEAD   |
| TL_v       |   TEAM_LEAD   |
| V_AG_k     | SERVICE_AGENT |
| V_BO_Rp_v  |  BACKOFFICE   |
| V_TL_k     |   TEAM_LEAD   |

Once the authsource config is defined, then by using the Username and password, we can login to the application.

In case of `nbawrite:password`, the Username is `nbawrite` and password is `password`.

The Kündigungamanager is accessible with the Lava User

## UI Client Configuration

In the root of the UI-Portal Project there is a configuration file needed, to determine which modules should be deployed/active.

It is a simple JSON file named `ui-clients.config` containing following:

```json
{
  "nba": true,
  "kulanz": false,
  "winback": true,
  "docManager": false,
  "lava": false,
  "invManager": false,
  "lavaCom": false,
  "cancellation": true,
  "easy": true
}
```

<br/><br/><hr><hr/><hr/><br/><br/>

## old stuff, might not be valid anymore...

### If you want use SAML in your own project as it was done for LavaSuite

For the Lumen/Laravel Setup we considered the `aacotroneo/laravel-saml2`. To use this package, please do the following steps

- Install package `composer require aacotroneo/laravel-saml2`
- Add config file for the saml_settings

As mentioned in the Documentation, please copy the vendor/aacotroneo/laravel-saml2/src/config/saml_settings.php to LavaApiPortal/config/saml_settings.php

Based on the requirements and configuration purpose adjust the following array keys in saml_settings.php

```php
 'idpNames' => ['lavasuite'],  // this is just a reference name for the Application SP.
 'logoutRoute' => env('SAML2_LOGOUT_ROUTE','/'),
 'proxyVars' => env('SAML2_PROXY_VARS',false),
 'saml2_controller' => 'App\Http\Controllers\Auth\LavasuiteSaml2Controller',
```

In LavaSuite we used LavasuiteSaml2Controller for the UI Call handling. Due to some missing functionality of the aacotroneo/laravel-saml2,
we developed this Controller class.

- Add config file for the idp_settings.php

As mentioned in the Documentation, please copy the vendor/aacotroneo/laravel-saml2/src/config/test_idp_settings.php  to LavaApiPortal/config/test_idp_settings.php

Based on the idpNames, the test_idp_settings.php should be renamed as lavasuite_idp_settings.php and modify the following array properties of this PHP file.

```php
$this_idp_env_id = 'LAVASUITE'; // Upper case of the idpName
$idp_host = env('SAML2_'.$this_idp_env_id.'_IDP_HOST', 'http://localhost/simplesaml');
'x509cert' => env('SAML2_'.$this_idp_env_id.'_IDP_x509', ''),
'authnRequestsSigned' => true,
'signMetadata' => true,
'wantAssertionsSigned' => true,
contactPerson.****
organization.****
```

- Add SAML2 Routes file

To enable the Routes and also to use the own Controller we need to do copy the vendor/aacotroneo/laravel-saml2/src/routes.php into
LavaApiPortal/routes/samlroutes.php and adjust based on the Lumen way of injection. Please refer samlroutes.php file in LavaApiPortal.

The aacotroneo package is programmed in the older version of lumen. That's why we adjusted based on the new version. Here we can also do some
rework to use the version as it is.

- Add SAML Event related files.

When the User is Logged into the IAM then a request /acs (Assertion Consumer Service call) will be triggered to the Service Provider where the
Service Provider needs to adopt the Given parameters where a JWT token will be generated and redirected to the UI based on the returnTo parameter.
Similarly, we need to consume the Logout event.

For this Purpose, we created the Event models Saml2Event.php, Saml2LoginEvent.php, Saml2LogoutEvent.php .

- Add Saml2LoginListener.php file in the Listener folder and implement the handle(). In this class the Saml2LoginEvent will be handled.

When the UI sends the /loginUrl request to the Backend, a new URL for the IAM login will be generated and will be sent back to the UI.
Once the User successfully logged in, a /acs call will be triggered on Service Provider.
As part of processing the /acs call a new Event Saml2LoginEvent will be generated which will be handled in this Saml2LoginListener.php.

In this handle function, the User data by UserId will be saved in the Cache. This cached data will be used for the JWT Middleware to check
if the User still exists in the Cache instead of asking the IAM if the user still exists.

- Add Saml2LogoutListener.php file in the Listener folder and implement the handle(). In this class the Saml2LogoutEvent will be handled.

When the UI sends the /logout request to the backend, a new URL for the IAM logout will be generated and will be sent back to the UI.
Once the UI forwards the logout URL and when the logout is success, the Service Provider will receive the /sls (Single Logout Service) request
from the IAM will trigger the  Saml2LogoutEvent.php. The Saml2LogoutListener.php handles this Saml2LogoutEvent and deletes the Session Cache.

- Create custom Saml2User.php for the missing attributes of \Aacotroneo\Saml2\Saml2User

- Add Saml2ServiceProvider.php Providers to register the SAML routes
- Adjust the EventServiceProvider.php with the new Saml2LoginEvent, Saml2LoginListener, Saml2LogoutEvent, Saml2LogoutListener
- Adjust the AppServiceProvider.php to overwrite the UrlGenerator from Laravel. -
- New Custom SamlAuth file LavaSuiteSaml2Auth.php is created to handle the missing / inconsistency of aacotroneo/laravel-saml2 . Here we mainly changed the loadOneLoginAuthFromIpdConfig(), sls() for our purpose
- New Custom Controller LavasuiteSaml2Controller.php is added for the Lumen request handling purpose and also adjustment which are needed the UI handling.

- Register the config files and Providers

Once all configs and Providers are prepared, they need to be registered wih the app. For this purpose we need to add the following
in the bootstrap/app.php

```php
$app->configure('saml2_settings'); // SAML2 Settings
$app->configure('lavasuite_idp_settings'); // SAML2 IDP Settings

$app->register(\App\Providers\Saml2ServiceProvider::class); // Provider class to register SAML related files.
$app->register(App\Providers\EventServiceProvider::class); // Event provider to register the SAML Events
```

## LavaApiPortal Environment Variables

Edit/Create `.env` file in `LavaApiAportal` project

Add/Edit the details of database from where the data needs to be migrated:

```properties
# target database
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=lavasuite
DB_USERNAME=lavasuite
DB_PASSWORD=<secret>

# migration source database from Winback
DB_HOST_WINBACK=localhost
DB_PORT_WINBACK=3306
DB_DATABASE_WINBACK=xlogic_winback
DB_USERNAME_WINBACK=lavasuite
DB_PASSWORD_WINBACK=<secret>
```

Similar to the above, please add also the Database connections for other Databases. See in .env.example.

for developing from inside the wsl to reach vodafone magma/sums you might tell the wsl/php to connect via a proxy, by doing:

```shell
# just for development to reach magma from inside wsl
http_proxy=192.168.254.254:8888
HTTP_PROXY=192.168.254.254:8888
https_proxy=192.168.254.254:8888
HTTPS_PROXY=192.168.254.254:8888
```

this may cause problems to reach other internet targets (like apt-get or scm-manager). Therefore [bash_configs](https://scm.liongate.cloud:443/scm/repo/Intern/bash_configs) provides an alias function to (un)set these variables accordingly, named `htproxy (set/setport/unset/info)`

then

Open mysql client, e.g. with `mysql -u root -p <secret>`

```sql
CREATE DATABASE lavasuite;
# maybe needed migration databases (see further down)
CREATE DATABASE xlogic_winback;
```

cd to LavaApiPortal

create the DB Schema (DDL) and run the seeders to populate the tables:

Before the seeders are going to run please fill all the .env Keys (starts with CC_**). Please see example in .env.template.development.
All these values are shared with the cloud passman.

```bash
php artisan migrate
php artisan --no-interaction db:seed --class=WinbackDatabaseSeeder
php artisan --no-interaction db:seed --class=KulanzDatabaseSeeder
php artisan --no-interaction db:seed --class=NbaDatabaseSeeder
php artisan --no-interaction db:seed --class=OtherDatabaseSeeder
```

(maybe you have to run db:seed again, if you get an error the first time)

## further old stuff, might not be valid anymore...

### PHP Upgrade to 8.0

The following steps are needed to upgrade to new PHP Version. This changes are done in the API Branch feature/LAXM-130-php-8.0

```bash
sudo apt install php8.0
sudo apt install php8.0-{apcu,gmp,igbinary,imagick,imap,memcached,msgpack,redis,xdebug,bcmath,fpm,xml,mysql,zip,intl,ldap,gd,cli,bz2,curl,mbstring,opcache,soap,cgi}
nano /etc/liongate/dev/lavasuite
set PHP_VERSION="8.0"
dev switch lavasuite
composer install
composer dump-autoload
```

Currently the following are depends on the 8.0 version. In the next iteration we need to make sure that these packages are compatible with the 8.1 and then change to next PHP version.
````php
https://packagist.org/packages/firebase/php-jwt  (requires php: ^7.1||^8.0)
https://packagist.org/packages/guzzlehttp/guzzle (requires php: ^7.2.5 || ^8.0)
https://packagist.org/packages/illuminate/redis (requires php: ^8.0.2)
https://packagist.org/packages/laravel/lumen-framework (requires php: ^8.0.2)
````

## Initialising the development process (UI)

### Run & watch through pm2

in lava-ui-portal run following commands to run through pm2. wait until the process completes.

```shell
npm run build-common-libs
// wait until process completes (check with command: pm2 logs)
npm run build-app-libs
// wait until process completes (check with command: pm2 logs)
npm run start
```

To see compilation errors, you can keep `pm2 logs` running.

### Building libraries

The lava-ui consists of 3 libraries that correspond to different clients (lava-ui-kulanz, lava-ui-winback, lava-ui-productcatalog) and 2 libraries which contain common components and helpers (lava-ui-commons, lava-ui-uicomponents).

The lava-ui-portal serves as a central application that communicates and bundles the functions and services provided by each library.

### Using libraries

Once the required libraries have been linked or *virtually installed* in the lava-ui-portal, they can now be accessed and used like any other Angular Package as follows:

#### For library components and modules

1. Open the `app.module.ts` in the lava-ui-portal (`lavasuite/lava-ui-portal/src/app/app.module.ts`).
2. Import the required module from the previously linked library: `import { LibraryModuleName } from 'library-name'`.
 For example, importing the lava-ui-kulanz module would look like: `import { LavaUiKulanzModule } from 'lava-ui-kulanz'`.
3. Declare the imported module inside the `imports` array under NgModule.
4. Since library modules also export the library's components, components do not have to be declared in the `app.module.ts`.
5. All components and imported modules can now be used in the lava-ui-portal application.

#### For library services

1. Navigate to the component or file that requires the library service.
2. Import the required service from the previously linked library: `import { LibraryServiceName } from 'library-name'`.
    For example, importing lava-ui-kulanz service would look like: `import { LavaUiKulanzService } from 'lava-ui-kulanz'`.
3. Inject the imported service in the class constructor of the component by defining an alias to the library service as a constructor parameter: `constructor(private aliasName: LibraryServiceName){}`.
4. All the available functions and data members from the imported service can then be accessed via the alias (aliasName in this case).

## Tools

### Terminal shell

```bash
sudo apt-get install -y mariadb-client
sudo service mysql start
sudo mysql -u root
# or
sudo mysql -u USER -p DBNAME # -p is asking for password on connection
```

#### VSCode

There are several plugins that should work out of the box:

- <https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools>
- with
- <https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql>

#### MySQL Workbench on Windows host, connecting to Database in WSL

- Powershell with admin rights: `choco install mysql.workbench`

- allow MySQL inside WSL to get connections from something else than just only localhost:

  MariaDB conf in `/etc/mysql/my.cnf`
  <br/>including: `/etc/mysql/mariadb.conf.d/50-server.cnf`
  <br/>change bind address:   `bind-address = 0.0.0.0`
  <br/>`service mysql restart`

- grant user privileges to the user connecting from outside of the WSL:
  - Inside WSL Terminal check the WSL's IP with:   `wsl hostname -I`:
  - make sure the IP matches the netmask in the following statements
  - make sure to use the team agreed password

```sql
SELECT host FROM mysql.user WHERE user = "root";
-- password not allowed to be empty here:
GRANT ALL PRIVILEGES ON *.* TO 'root'@'172.0.0.0/255.0.0.0' IDENTIFIED BY '<password>' with grant option;
FLUSH PRIVILEGES;
```

- in MySQL Workbench edit your connection:
  - go to `Advanced` Tab ('Parameters', 'SSL', --> 'Advanced')
  - in TextField `Others` write <big>`useSSL=0`</big>

<!--page layouat css styles for markup headers numbering -->
<style type="text/css"> /* automatic heading numbering */ h1 { counter-reset: h2counter; font-size: 24pt; } h2 { counter-reset: h3counter; font-size: 22pt; margin-top: 2em; } h3 { counter-reset: h4counter; font-size: 16pt; } h4 { counter-reset: h5counter; font-size: 14pt; } h5 { counter-reset: h6counter; } h6 { } h2:before { counter-increment: h2counter; content: counter(h2counter) ".  "; } h3:before { counter-increment: h3counter; content: counter(h2counter) "." counter(h3counter) ".  "; } h4:before { counter-increment: h4counter; content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) ".  "; } h5:before { counter-increment: h5counter; content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "." counter(h5counter) ".  "; } h6:before { counter-increment: h6counter; content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "." counter(h5counter) "." counter(h6counter) ".  "; } </style>
