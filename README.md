# 🛡️ Ansible Playbooks: Linux Server Setup

Este repositorio contiene una configuración básica para un servidor Ubuntu/Debian utilizando Ansible.

## ✔️ Roles incluidos

- `common`: actualización e instalación de paquetes básicos
- `docker`: instalación completa de Docker
- `security`: configuración de Fail2Ban y UFW

## 🔧 Uso

1. Edita `inventory.ini` con la IP y usuario de tu servidor
2. Ejecuta:

```bash
ansible-playbook -i inventory.ini site.yml
```

> Requiere acceso por SSH y permisos sudo en el host.

