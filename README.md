# Playbooks de Ansible para automatización de servidores

Este repositorio contiene una colección de playbooks de Ansible diseñados para automatizar el aprovisionamiento, la configuración y el refuerzo de seguridad (hardening) de servidores Linux.
Actualmente soporta y detecta automáticamente: **Debian/Ubuntu, RHEL/CentOS, Archlinux, SUSE y Alpine Linux**.

## Características

*   **Soporte Multi-OS**: Detección automática de familia de SO y carga dinámica de variables (`Debian.yml`, `RedHat.yml`, etc.).
*   **Fail-Fast Strategy**: Validación temprana del sistema operativo para evitar ejecuciones erróneas.
*   **Aprovisionamiento Automatizado**: Instalación de paquetes esenciales (`common`) y gestión de repositorios.
*   **Seguridad Avanzada**:
    * **Firewall Inteligente**: Abstracción automática entre `ufw` (Debian/Ubuntu) y `firewalld` (RHEL/CentOS).
    * **SSH Hardening**: Configuración parametrizable de puertos y permisos de root.
    * **Fail2Ban**: Instalación y activación automática para prevenir intrusiones.
*   **Docker Production-Ready**:
    * Instalación segura usando repositorios oficiales (sin `apt-key` deprecated).
    * Soporte para arquitecturas `amd64` y `arm64`.
    * Configuración automática de **Log Rotation** en `daemon.json`.
    * Bootstrap automático para Python en Alpine Linux.
*   **Idempotent**: Seguro de ejecutar múltiples veces sin efectos secundarios.

## Estructura del repositorio

El proyecto sigue una arquitectura modular separando lógica (`tasks`), datos internos (`vars`) y configuración de usuario (`defaults`).

```
.
├── ansible.cfg             # Configuración global
├── inventory.ini           # Inventario de servidores
├── site.yml                # Playbook principal (orquestador)
├── collections/
│   └── requirements.yml    # Dependencias externas
├── roles/
│   ├── bootstrap/          # Preparación inicial (Python raw install)
│   ├── common/             # Paquetes base y Timezone
│   │   ├── tasks/          # Lógica principal con selectores de SO
│   │   ├── handlers/       # Reinicio de servicios (cron)
│   │   └── vars/           # Variables específicas (Debian.yml, Alpine.yml...)
│   ├── docker/             # Motor de contenedores y Compose
│   │   ├── defaults/       # Variables sobrescribibles (usuarios, log options)
│   │   ├── handlers/       # Reinicio del servicio Docker
│   │   ├── tasks/          # Instalación inteligente por familia de SO
│   │   └── vars/           # Lógica interna (nombres de paquetes, repos)
│   └── security/           # Firewall (UFW/Firewalld), SSH Hardening, Fail2ban
│       ├── defaults/       # Políticas configurables (puertos, root login)
│       ├── handlers/       # Reinicio de servicios (sshd, firewall)
│       ├── tasks/          # Lógica de hardening multi-OS
│       └── vars/           # Nombres de servicios por OS (Debian.yml, RedHat.yml)
└── .github/
    └── workflows/          # CI/CD (Ansible Lint)
```

## Uso

1.  **Clona el repositorio:**
    ```bash
    git clone https://github.com/ygallardops/ansible-playbooks.git
    cd ansible-playbooks
    ```

2. **Instalar dependencias (Colecciones):**
    ```bash
    ansible-galaxy install -r collections/requirements.yml
    ```

3.  **Configurar el inventario:**
    Edita el archivo `inventory.ini` con las direcciones IP de tus servidores destino.
    ```Ini, TOML
    [servers]
    192.168.1.10 ansible_user=ubuntu
    192.168.1.11 ansible_user=root # Ejemplo para Alpine 
    ```

4.  **Ejecuta el Playbook:**
    ```bash
    ansible-playbook -i inventory.ini site.yml
    ```

## Variables Principales

| Variable | Descripción | Default |
| :--- | :--- | :--- |
| `docker_users` | Lista de usuarios a añadir al grupo docker | `['ansible_user_id']` |
| `common_upgrade_system` | Si es `true`, actualiza todos los paquetes del sistema | `false` |
| `docker_daemon_options` | Configuración JSON para `daemon.json` | `{ "log-driver": "json-file"... }` |

### Variables de Seguridad (Role: Security)

| Variable | Descripción | Default |
| :--- | :--- | :--- |
| `security_ssh_port` | Puerto donde escuchará SSH (se abre en firewall automáticamente) | `22` |
| `security_ssh_permit_root_login` | Permitir acceso root por SSH (`yes`/`no`) | `"no"` |
| `security_enable_firewall` | Activar gestión de firewall | `true` |
| `security_allowed_ports` | Lista de puertos TCP a abrir (ej. `[22, 8080]`) | `[22, 80, 443]` |
| `security_enable_fail2ban` | Instalar y activar servicio Fail2Ban | `true` |

## CI/CD

Este repositorio utiliza **GitHub Actions** para asegurar la calidad del código. Cada push desencadena un flujo de trabajo que:
*   Analiza la sintaxis YAML (Linting).
*   Ejecuta `ansible-lint` para garantizar buenas prácticas.

## Licencia

MIT
