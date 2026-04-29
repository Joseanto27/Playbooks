# 🖥️ PROYECTO: Automatización mediante Playbooks en Bash

| Campo | Detalle |
| :--- | :--- |
| **Alumnos** | Eric Guillen Artacho <br> Ayoub Eddahbi Bourbah <br> José Antonio Pineda Fernández |
| **Curso** | 2º ASIX – Administración de Sistemas Informáticos en Red |
| **Módulo** | Implantación de Sistemas Operativos / Seguridad |

---

## 1. Descripción del Proyecto

Este proyecto consiste en la creación de un despachador de eventos desarrollado íntegramente en Bash. El objetivo es monitorizar el sistema de forma continua y aplicar **Playbooks** (recetas de acciones) cuando se detecten anomalías. He estructurado el código para separar la detección de la reacción, cumpliendo con los requisitos de diseño modular.

---

## 2. Arquitectura de Archivos (Estructura Modular)

Para no tener un script gigante e inmanejable, he dividido la lógica en tres archivos en `/root/`:

| Archivo | Rol |
|--------|-----|
| `eventos.sh` | Biblioteca de auditoría. Define las funciones que chequean el sistema (Apache, Disco y Oracle). |
| `suscriptores.sh` | Biblioteca de soluciones. Contiene las funciones que arreglan los problemas detectados. |
| `playbook_main.sh` | Motor principal que coordina todo usando Arrays Asociativos. |

---

### 📄 `eventos.sh`

<img width="802" height="329" alt="image" src="https://github.com/user-attachments/assets/0bcc6de6-073f-43d7-bf2a-059c1bb94afd" />


> Lo más importante aquí no es el comando que usamos (como pgrep), sino cómo gestionamos el retorno. En Bash, el Playbook principal interpreta un return 0 como "Evento Detectado (Éxito)" y un return 1 como "Estado OK (Fallo)". Esta es la base de la lógica binaria del despachador.

---

### 📄 `suscriptores.sh`

<img width="841" height="398" alt="image" src="https://github.com/user-attachments/assets/faf3d326-743b-4d60-9840-885187118a43" />


> El punto fuerte de este archivo es la integración con Ansible (ansible-playbook). Al delegar tareas complejas de seguridad (como el Hardening de Oracle) a una herramienta de orquestación, demostramos cómo Bash puede coordinar diferentes niveles de automatización. Además, el uso de la variable $? para chequear si Ansible terminó bien o mal añade robustez al log.

---

### 📄 `playbook_main.sh`

<img width="724" height="513" alt="image" src="https://github.com/user-attachments/assets/f11f9e89-e46d-40fb-8085-f9aefc63ca44" />


> El uso de declare -A es la clave técnica del proyecto. Estos arrays actúan como diccionarios, permitiendo "suscribir" funciones de respuesta a IDs de eventos de forma dinámica. Esto cumple con el requisito de abstracción: podemos añadir 50 eventos nuevos sin tocar ni una sola línea del bucle for principal.

---

## 3. Implementación de los 3 Playbooks

### 📦 Playbook 1: Gestión de Almacenamiento (`Evento DISCO`)

He configurado un evento que detecta cuando el espacio en disco es crítico. El suscriptor asociado ejecuta una limpieza de archivos temporales para evitar que el sistema se bloquee.

### 🌐 Playbook 2: Disponibilidad de Servicios (`Evento APACHE`)

Mediante el comando `pgrep`, el script monitoriza el servicio web. Si el servicio cae, el suscriptor lanza automáticamente un `systemctl start apache2`, garantizando la alta disponibilidad (RA3).

### 🔒 Playbook 3: Hardening de Seguridad (`Evento ORACLE + Ansible`)

Este es el punto más avanzado. He integrado **Ansible** dentro del flujo de Bash. Si el evento de Oracle detecta que falta configuración de seguridad, el script lanza un Playbook de Ansible (`cis_oracle19c.yml`) que aplica normativas CIS de contraseñas y bloqueo de usuarios.

---

## 4. El Despachador (Uso de Arrays Asociativos)

Para que el sistema sea escalable, he usado una de las herramientas más potentes de Bash: los **arrays asociativos**. Esto permite "suscribir" funciones a eventos de forma muy sencilla:

```bash
declare -A EVENTOS
declare -A SUSCRIPTORES

# Registro de la suscripción
EVENTOS[APACHE]="evaluar_estado_servicio_apache"
SUSCRIPTORES[APACHE]="apache_restorer_responder"
```

---

## 5. Automatización con Cron

Para que el sistema funcione de forma autónoma (sin que yo tenga que estar delante), he configurado un archivo en `/etc/cron.d/proyecto`. El sistema se autoevalúa **cada 2 minutos**:

```bash
*/2 * * * * root /bin/bash /root/playbook_main.sh >> /root/log_playbook.txt 2>&1
```
