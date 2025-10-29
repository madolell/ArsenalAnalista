# ⚡ Masscan CheatSheet: Comandos Esenciales y Avanzados

Esta hoja de comandos está diseñada para la **eficiencia y velocidad** en escenarios de reconocimiento a gran escala.

## I. Sintaxis Básica y Escaneo Rápido

| Comando | Descripción |
| :--- | :--- |
| `masscan <rango_IP> -p<puertos> --rate <pps>` | Sintaxis base para escaneo de alta velocidad. |
| `masscan 192.168.1.0/24 -p80,443 --rate 10000` | Escaneo rápido del segmento 192.168.1.x para HTTP/HTTPS a 10k pps. |
| `masscan -iL objetivos.txt -pU:53 -pT:21-23` | Escaneo de objetivos desde archivo; Puertos UDP (DNS) y TCP (FTP, SSH, Telnet). |
| `masscan --top-ports 100 10.0.0.0/8` | Escanea los 100 puertos más comunes en la red Clase A. |

---

## II. Control de Velocidad (`--rate`) y Adaptación

La bandera `--rate` es el corazón de Masscan. Indica la cantidad de paquetes por segundo (pps).

| Comando | Escenario | Notas de Rendimiento |
| :--- | :--- | :--- |
| `masscan ... --rate 1000` | Entornos de red sensible o VMs. | Tasa segura para la mayoría de entornos. |
| `masscan ... --rate 100000` | Alto rendimiento en un segmento de red local (LAN). | Requiere buena CPU y hardware de red. |
| `masscan ... --rate 10000000` | Escaneo a escala de Internet (solo si el hardware y la red lo soportan). | ¡Cuidado! Puede saturar tu enlace. |
| `masscan ... --offline` | Modo de prueba: Solo simula la velocidad sin enviar paquetes. | Ideal para estimar el tiempo sin generar tráfico. |

---

## III. Salida de Resultados Profesional (`-oX`, `-oJ`, `-oL`)

Para un análisis posterior y la integración con otras herramientas. **Recomendación: usar `-oG` o `-oL` para procesado o `-oX` para integración con Nmap.**

| Flag | Formato | Uso Recomendado |
| :--- | :--- | :--- |
| `-oL <file>` | **List Format** (IP:Puerto). | Ideal para *piping* simple y scripts (**más fácil de parsear**). |
| `-oG <file>` | **Grepable Format** (Nmap Style). | Compatible con herramientas que procesan salida Nmap (no XML). |
| `-oX <file>` | **XML Format** (Detallado). | **Recomendado para la integración con Nmap** (`-iX`). |
| `-oJ <file>` | **JSON Format**. | Ideal para proyectos que integran bases de datos (ElasticSearch, etc.). |

### 💡 Ejemplo Avanzado: Parseo Rápido

```bash
# Escanea puertos comunes, solo reporta los abiertos y guarda la salida limpia.
sudo masscan 172.16.0.0/16 -p21,22,80,443,3389 --rate 50000 --open-only -oL masscan_abiertos.list

# Extrae solo las IPs de la lista para pasar a otra herramienta (ej. Nmap)
cat masscan_abiertos.list | awk '{print $NF}' | sort -u > targets_para_nmap.txt

```
