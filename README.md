# Analítica de logs de switches

Analiza los logs de switches de distintos fabricantes y muestra qué errores se repiten, qué equipos están en
riesgo, qué eventos de varios equipos tienen una **causa común** y qué interfaces pueden **fallar en las próximas
24–72 h**. Corre en tu propio servidor Ubuntu: los logs no salen de tu red.

**Gratis (freeware).** Se distribuye compilado: se puede usar y compartir, pero no modificar (ver [LICENSE](LICENSE)).

## Qué hace
- **Lee** syslog suelto o comprimido y paquetes de soporte: `cl-support` de NVIDIA Cumulus, sysdump de NVIDIA Onyx
  (Mellanox) y logs de Cisco IOS/IOS-XE/NX-OS, Arista EOS, Juniper Junos, Huawei VRP y HPE Aruba CX.
- **Normaliza** cada mensaje a una categoría y un nivel de impacto comunes, sin importar la marca.
- **Prevención**: riesgo 0–100 por equipo, interfaces con problemas físicos (caídas, protodown, cambio de
  transceptor y si siguió cayendo), alertas tempranas, tendencias y flapping.
- **Correlación y pronóstico**: junta los problemas simultáneos de varios equipos en episodios con su causa probable
  y la evidencia (degradación óptica previa, mismo enlace, mismo servidor en el par MLAG, reinicio, peerlink,
  cambio de configuración), detecta patrones que se repiten y calcula la probabilidad de que cada interfaz vuelva a
  caer. Con los `cl-support` lee además cada transceptor: potencia Rx/Tx contra su umbral, temperatura y errores.
- **Histórico**: guarda cada carga con un nombre, muestra qué se repite entre cargas y borra lo antiguo solo.
- **Reporte** en PDF (ejecutivo + técnico) y Excel, con anexo opcional de la configuración de los equipos y las
  claves ocultas.

## Requisitos
- Ubuntu **26.04** x86_64 (servidor o máquina virtual), 2 CPU y 4 GB de RAM o más.
- Internet para instalar las librerías, o el paquete **sin internet** (`-offline`).

## Instalar
1. Descarga de [Releases](../../releases) el paquete más reciente y su `SHA256SUMS`:
   - `switch-log-analytics-<versión>-protegido-ubuntu26.04-x86_64.tar.gz`: instala las librerías por internet.
   - `...-offline.tar.gz`: trae todo, para servidores sin internet.
2. Verifica que el archivo es el original (no fue alterado):
   ```bash
   sha256sum -c SHA256SUMS --ignore-missing
   ```
3. Instala:
   ```bash
   tar xzf switch-log-analytics-*-ubuntu26.04-x86_64.tar.gz
   cd switch-log-analytics
   sudo ./instalar.sh
   ```
4. Entra a `http://<ip-del-servidor>:8580` con la clave que muestra el instalador al terminar.

Queda como servicio `switchlogs`. Para actualizar, se repite lo mismo con el paquete nuevo: se conservan los datos y
la configuración.

| Qué | Dónde |
|---|---|
| Programa (de root, solo lectura) | `/opt/switch-log-analytics` |
| Datos: base de datos y logs subidos | `/var/lib/switchlogs` |
| Configuración: clave, puerto, zona horaria, días que se guarda cada carga | `/etc/switchlogs.env` |

```bash
sudo systemctl status switchlogs        # estado
sudo journalctl -u switchlogs -n 100    # registro
sudo systemctl restart switchlogs       # después de cambiar /etc/switchlogs.env
```

## Cómo sacar los logs de cada equipo
| Equipo | Qué subir |
|---|---|
| NVIDIA Cumulus | `sudo cl-support` → `/var/support/cl_support_<equipo>_<fecha>.txz` |
| NVIDIA Onyx (Mellanox) | `debug generate dump` → el sysdump |
| Cisco, Arista, Juniper, Huawei, Aruba | los archivos del servidor syslog central |

## Privacidad
Todo se procesa en tu servidor. La app no envía nada a internet. Los logs tienen IPs, usuarios y nombres de
equipos: mantén la clave de acceso y no publiques el puerto a internet.

## Errores y sugerencias
Abre un *Issue* en este repositorio. **No adjuntes logs reales ni `cl-support`**: traen datos de tu red.

## Licencia
© 2026 Sebastian Antunez Noguera. Freeware: uso gratis, incluso en empresas, y redistribución del paquete original
sin cambios. Prohibido modificarlo, descompilarlo o venderlo. Detalle en [LICENSE](LICENSE).
