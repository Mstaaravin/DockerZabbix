# DockerZabbix — Instrucciones del proyecto

## Descripción
Stack Docker para administrar un servidor Zabbix en el host remoto `zabbix`.
Incluye: Traefik v3 (reverse proxy + TLS), Zabbix Server 7.4, Zabbix Web y MariaDB 12.

---

## HOST LOCAL — source of truth

**Path:** `~/Projects/git.DockerZabbix` (= `/home/cmiranda/Nextcloud/Projects/git.DockerZabbix`)
**Repo git:** `git@gitea.marvin.ar:lhome/DockerZabbix.git` (push también a `git@gitlab.lan:lhome/dockerzabbix.git`)

- Es el origen de todos los cambios de configuración.
- Los archivos sensibles (`.env`) usan **placeholders** (`changeme_*`) para poder versionarlos sin exponer credenciales.
- Cada cambio se commitea aquí primero y luego se sincroniza al remoto.

---

## HOST REMOTO — producción

**Host:** `zabbix` (alias SSH) = `root@172.16.250.15`
**Path:** `/docker`

- Contiene los archivos reales con valores verdaderos (contraseñas, IPs, etc.).
- Tiene su **propio repositorio git local independiente** (sin remote configurado).
- Se debe hacer commit en `/docker` en el remoto con cada cambio sincronizado.
- El git del remoto no se pushea a ningún servidor; es solo para auditoría local.
- Zabbix 7.4 usa Bearer token en el header, no el campo auth.

---

## Workflow de sincronización

1. Hacer cambios y commit en LOCAL (`~/Projects/git.DockerZabbix`)
2. Sincronizar al remoto con rsync, excluyendo `.git`, `CLAUDE.md` y `.claude/`:
   ```
   rsync -av --exclude='.git' --exclude='CLAUDE.md' --exclude='.claude/' \
     ~/Projects/git.DockerZabbix/ zabbix:/docker/
   ```
3. En el remoto, actualizar el `.env` con valores reales si corresponde
4. Hacer commit en el remoto:
   ```
   ssh zabbix "cd /docker && git add -A && git commit -m 'descripción'"
   ```
5. Aplicar cambios en el stack si es necesario:
   ```
   ssh zabbix "cd /docker && docker compose up -d"
   ```

---

## Archivos y estructura

| Archivo / Dir | Versionado local | En remoto | Notas |
|---|---|---|---|
| `compose.yml` | sí | sí | Stack completo |
| `.env` | sí (placeholders) | sí (valores reales) | Sensible |
| `.gitignore` | sí | sí (copiado vía rsync) | |
| `traefik/certs/` | sí | sí | Certificado TLS zabbix.lan |
| `zabbix_db/backup.conf` | sí | sí | Config de backup |
| `zabbix_db/data/custom.cnf` | sí | sí | Config MariaDB |
| `zabbix_db/data/*` | no (gitignore) | sí | Datos de MariaDB |
| `zabbix/*` | no (gitignore) | sí | Scripts, módulos, etc. |
| `CLAUDE.md` | no (gitignore) | no (excluido rsync) | Solo instrucciones |
| `.claude/` | no (gitignore) | no (excluido rsync) | |

---

## Referencias

- Documentación variables MariaDB Docker:
  https://mariadb.com/docs/server/server-management/automated-mariadb-deployment-and-administration/docker-and-mariadb/mariadb-server-docker-official-image-environment-variables

- Documentación oficial de Zabbix v7.4:
  https://www.zabbix.com/documentation/7.4/en/manual/

- Docker Variables for Zabbix server
  https://hub.docker.com/r/zabbix/zabbix-server-mysql#other-variables
