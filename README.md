# nix-odoo
Nix templates for Odoo development servers

## USAGE

(!) **After INSTALL**

**Start server (postgres, odoo)**

`$ ./dev-server.sh start`

When you want to import a database which takes a long time, use:

`$ ./dev-server.sh start --limit-time-real=-1`

This ensures the importer won't be killed for taking too long.

**Other commands, just...**

`$ ./dev-server.sh`

**Example: psql**

`$ ./dev-server.sh psql`


## INSTALL (bootstrap)

1. git clone https://github.com/novacode-nl/nix-odoo

2. `$ cp -R nix-odoo/odoo-15 TARGET`

   Example:
   `$ cp -R nix-odoo/odoo-15 novacode-15`

3. `$ cd TARGET`

4. Edit odoo.conf
   - addons_path
   - db_host (absolute path to postgres socket-dir)
   - http_port
   - longpolling_port
   - Other dirs/files (geoip, screenshots)

5. Add odoo (+ enterprise) dir

   Example:
   `$ git clone https://github.com/odoo/odoo.git --branch 15.0 --depth 1`

6. Add project AKA addons dir

    **The `dev-server.sh` script expects this as directory name `addons`.**

    Example:
   `$ git clone https://github.com/novacode-nl/novacode.git --branch 15.0 addons`

7. `$ lorri shell` (STILL NEEDED with direnv ?)
8. `$ nix-build wkhtmltopdf.nix`
9. `$ ./dev-server.sh install`
10. `$ ./dev-server.sh start`


## FEATURES

- PostgreSQL listens (serves) from socket directory
- File `odoo-VERSION/pyproject.toml` (created from odoo `requirements.txt`)

## UPGRADE - upgrade.odoo.com

Having PostgreSQL running on a Unix socket causes issues with Odoo's upgrade script.\
The upgrade script excepts PostgreSQL connection at `/run/postgresql/.s.PGSQL.5432`.

### Current workaround

**Terminal (tty) 1**

```
cd nix-odoo/TARGET
./dev-server postgres_tcp
```

**Terminal (tty) 2**
```
sudo su
mkdir /run/postgresql
ln -s nix-odoo/TARGET/postgres/.s.PGSQL.5432 /run/postgresql/
python <(curl -s https://upgrade.odoo.com/upgrade) test -d <your db name> -t <target version>
```

## TODO

1. Unclutter the root-dir

```
   nix-odoo/
       .git # this git repo
       odoo-15/
           # will become a git repo, eg. to share
           .gitignore # odoo, postgres, data (liquid dirs, files)
           dev-server.sh
           pyproject.toml
           README.md
           shell.nix
           odoo/
               odoo.conf
               addons/
                  .gitempty
               (odoo/)
               (enterprise/)
	   postgres/
               .gitempty
	   data/
               .gitempty
	   scripts/
               prepare.sql
```

2. wkhtmltopdf (0.12.5)
   See file `shell.nix`

3. Generate `odoo-VERSION/pyproject.toml` from odoo `requirements.txt`
