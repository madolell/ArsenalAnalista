# 🛡️ Masscan Avanzado: El Escáner Asíncrono para Analistas Pro

![Captura Masscan](img/masscan.png)

## 🚀 Propósito de este recurso

Este repositorio no es solo una explicación teórica de **Masscan**, sino un **Recurso Técnico Operacional (RTO)** diseñado para analistas de vulnerabilidades y *red teamers*. Su objetivo es triple:

1. **Acelerar la Fase de Reconocimiento:** Proporcionar comandos optimizados para el descubrimiento ultrarrápido de puertos.
2. **Integración en el Flujo de Análisis:** Mostrar cómo se inserta Masscan eficientemente entre el escaneo de red y el análisis detallado con herramientas como Nmap.
3. **Uso Profesional:** Cubrir funcionalidades avanzadas, optimización de velocidad (`--rate`) y manejo de resultados para flujos de trabajo en escenarios reales.

---

## 📂 Estructura del Repositorio

| Archivo | Contenido | Enfoque |
| :--- | :--- | :--- |
| `README.md` | Introducción, propósito y estructura del proyecto. | Presentación. |
| `cheatsheet.md` | Comandos clave, sintaxis y *flags* avanzados. | Referencia Rápida. |
| `flujo_trabajo.md` | Flujo de trabajo profesional, integración con Nmap y solución de problemas. | Representación. |
| `guia_masscan.md` | Guia de uso avanzado para sacar el máximo potencial.| Metodología. |

---

## 💡 ¿Por qué Masscan y no Nmap?

Masscan opera de forma **asíncrona** y está optimizado para la velocidad pura, capaz de escanear todo el espacio IPv4 en minutos.

| Característica | Masscan | Nmap |
| :--- | :--- | :--- |
| **Velocidad** | 🔥 Extrema (Millones de paquetes/segundo). | Media/Alta (Según flags). |
| **Objetivo Primario** | Descubrimiento masivo de puertos abiertos (*"What is open?"*). | Análisis detallado de servicios (*"What is running?"*). |
| **Pila TCP/IP** | Propia/Custom (Envío *raw* de paquetes). | Nativa del SO (Más confiable en escaneos lentos). |
| **Uso Profesional**| Primera pasada de reconocimiento a gran escala. | Segunda pasada de *fingerprinting* y análisis de vulnerabilidades. |

---

## 🛠️ Requisitos del entorno

* **Sistema Operativo:** Linux (Recomendado para alto rendimiento).
* **Permisos:** Se requiere `root` o `sudo` para ejecutar Masscan, ya que utiliza *sockets* raw.
* **Ancho de Banda:** Una conexión de red robusta es crucial para el rendimiento a altas tasas (`--rate`).

> **Advertencia:** El uso de Masscan a tasas elevadas puede ser muy ruidoso y puede sobrecargar infraestructuras de red o *firewalls*. Utiliza siempre esta herramienta de forma ética y legal.

## ⚙️ Instalación

### En Debian/Ubuntu

```bash
`sudo apt install masscan` 
```

### En Windows

Descarga binario oficial desde:  
🔗 [https://github.com/robertdavidgraham/masscan](https://github.com/robertdavidgraham/masscan)

---

## 🚀 Uso básico

Escaneo rápido de un rango de IPs:

```bash
`sudo masscan 192.168.1.0/24 -p80` 
```

Escaneo de múltiples puertos:

```bash
`sudo masscan 192.168.1.0/24 -p22,80,443` 
```

Guardar resultados en JSON:

```bash
`sudo masscan 192.168.1.0/24 -p1-1024 -oJ resultados.json`
```

**¡Empieza ahora! Consulta la [cheatsheet.md](./assets/cheatsheet.md) para los comandos, el [flujo_trabajo.md](./assets/flujo_trabajo.md) para el flujo de trabajo y la [guia_masscan.md](./assets/guia_masscan.md) para el uso avanzado**

## ⚡ Casos prácticos

### 🧩 Caso 1: Detección de servicios HTTP/HTTPS en una subred corporativa

```bash
`sudo masscan 10.0.0.0/16 -p80,443 --rate 1000 -oG http_hosts.txt` 
```

**Interpretación:**  
Obtendrás una lista de hosts con servidores web activos, útil para priorizar análisis con **Nmap**, **Nikto** o **ZAP**.

---

### 🕵️ Caso 2: Escaneo de red masiva en entorno de auditoría

```bash
`sudo masscan 0.0.0.0/0 -p445 --rate 1000000 -oL smb_hosts.txt` 
```

**Objetivo:** detectar hosts con **SMB expuesto**.

### 🧪 Caso 3: Descubrimiento Rápido de Webservers

Este caso práctico simula el **reconocimiento inicial** en un proyecto de auditoría interna de una gran red corporativa (`172.16.0.0/16`). El objetivo es encontrar rápidamente todos los _web servers* activos (puertos 80 y 443) para priorizar el análisis con Nmap.

#### A. Definición del Escenario

-**Objetivo:** Red interna `172.16.0.0/16` (más de 65,000 IPs).
-**Puertos:** 80 (HTTP) y 443 (HTTPS).
-**Restricción de Red:** Usaremos una tasa conservadora para no sobrecargar la LAN interna.
-**Resultado Deseado:** Un archivo XML compatible con Nmap.

#### B. Ejecución del Comando de Reconocimiento

```bash
# Usamos una tasa de 10000 pps, ideal para LANs grandes sin ser excesivamente ruidosos.
# Especificamos la interfaz de red (ej. eth0) para asegurar que Masscan use la correcta.
sudo masscan 172.16.0.0/16 -p80,443 \
    --rate 10000 \
    --adapter eth0 \
    --banners \
    --open-only \
    -oX masscan_webservers_raw.xml
# NOTA: Reemplaza 'eth0' con la interfaz de red de tu máquina Kali (puedes verificarla con 'ip a').
```

#### C. Integración y Validación con Nmap

Una vez que Masscan termina (debería ser relativamente rápido para solo dos puertos), el analista debe validar el resultado y continuar el análisis.

```bash
echo "Iniciando Nmap para validación y fingerprinting de los webservers..."

# Nmap lee el XML generado por Masscan y sólo escanea las IPs/Puertos encontrados.
# Se añade el fingerprinting de versión (-sV) y scripts básicos (-sC).

nmap -iX masscan_webservers_raw.xml \
    -sV -sC \
    --min-rate 1000 \
    -oA nmap_webservers_final
```
