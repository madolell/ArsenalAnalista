# 🧑‍💻 Guía de _Onboarding_ Masscan

¡Bienvenido/a al equipo! Masscan es nuestra herramienta principal para el **reconocimiento de activos a gran escala**. Úsala sabiamente.

![IA](./img/Gemini.png)

----------

## 1. 🚦 ¿Cuándo USAR Masscan y por qué?

Utiliza Masscan como la **primera capa de reconocimiento** en cualquier análisis, especialmente en proyectos grandes.

| **Escenario** | **Uso Recomendado** | **Razón Principal** |
|--------------|--------------|--------------|
| **Descubrimiento de Perímetro (Auditoría Externa)** | **Siempre.** En bloques CIDR grandes (`/16`, `/8`, o listas extensas de activos). | **Velocidad Extrema.** Identifica rápidamente puertos abiertos sin gastar tiempo en _fingerprinting_. |
| **Análisis de Redes LAN Grandes** | **Sí, con `--rate` bajo (ver Sección 2).** | Ideal para identificar rápidamente la dispersión de un servicio específico (ej. "todos los servidores con puerto 3389 abierto"). |
| **Identificación Rápida de un Servicio Común** | **Siempre.** Para puertos estándar (80, 443, 22, 3389, etc.). | **Optimización.** Minimiza el tráfico necesario para obtener el dato clave: _Open/Closed_. |

## 2. 🛑 Qué EVITAR: Las Reglas de Oro del Equipo

Masscan es una herramienta potente y ruidosa. Su mal uso puede causar problemas de red o alertar innecesariamente a los equipos de Blue Team.

| **Regla de Oro** | **Descripción** | **Acción Correctiva** |
|--------------|--------------|--------------|
| **No es un Reemplazo de Nmap.** | Masscan no está diseñado para el _fingerprinting_ de servicios o el análisis de vulnerabilidades. | **NUNCA** esperes información de versión detallada. Úsalo solo para obtener la lista de puertos abiertos. |
| **Controla la Tasa (`--rate`).** | Usar tasas altas (`>100k`) puede saturar nuestros enlaces o sistemas de detección de intrusiones (IDS/IPS). | En entornos internos, mantente en **`--rate 10000`** o menos. En entornos externos, confirma la tasa máxima segura con el _Lead Analyst_. |
| **Respeta las Exclusiones.** | Si tenemos listas de _honeypots_ o activos críticos (ej. sistemas SCADA), **debes** excluirlas. | Usa siempre la bandera `--excludefile` con el archivo de exclusión del proyecto. |
| **Evita la Pila TCP Nativa.** | Masscan usa su propia pila de red. Su uso incorrecto puede confundir al sistema operativo. | Limita el uso de Masscan a funciones de escaneo y banner-grabbing simple. Si necesitas manipulación de paquetes compleja, usa otra herramienta. |

----------

## 3. 🧩 Combinaciones que potencian Masscan (El _Workflow_ Profesional)

Masscan brilla cuando se integra en un _pipeline_ automatizado. Aquí tienes nuestras combinaciones más efectivas:

### A. Masscan + Nmap (El Estándar)

Esta es la combinación esencial para cualquier análisis. Masscan hace el trabajo pesado y Nmap, el trabajo de precisión.

| **Herramienta** | **Función** | **Comando Clave (Flujo)** |
|--------------|--------------|--------------|
| **Masscan** | Escaneo masivo y rápido. | `masscan [targets] -p[ports] -oX masscan_out.xml` |
| **Nmap** | Análisis detallado (versiones, scripts). | `nmap -iX masscan_out.xml -sV -sC -oA nmap_report` |

> **TIP:** Al usar la salida XML de Masscan (`-oX`) con Nmap (`-iX`), le estamos diciendo a Nmap: "Solo escanea las IPs y puertos que Masscan ya marcó como abiertos." Esto ahorra **horas** de tiempo de análisis.

### B. Masscan + Grep/AWK/JQ (Procesamiento de Resultados)

Para análisis rápidos o la creación de listas personalizadas.

| **Flujo** | **Propósito** | **Ejemplo de Comando** |
|--------------|--------------|--------------|
| **Masscan + `awk`** | Generar una lista limpia y única de IPs para una herramienta posterior. | `masscan ... -oL list.txt && cat list.txt |
| **Masscan + **Python** | Para un análisis más complejo, como verificar si un puerto abierto coincide con un banner conocido. | **Recomendado:** Escribir un pequeño _script_ en Python que lea el formato JSON (`-oJ`) y realice la lógica de verificación. |

### C. Masscan + ZGrab (Para _Banner Grabbing_ Masivo y Específico)

Aunque Masscan puede obtener _banners_ simples (`--banners`), ZGrab es más robusto para realizar _handshakes_ más complejos (ej. TLS, SSH).

1. **Masscan:** Descubre IPs con puerto 443 abierto.

2. **AWK/Grep:** Extrae la lista de IPs únicas.

3. **ZGrab/Similar:** Se alimenta con la lista de IPs para recolectar el certificado TLS completo y los datos del _handshake_.

> **Conclusión:** Recuerda, Masscan es tu **"francotirador de descubrimiento"**. Úsalo para identificar el objetivo y luego trae las herramientas de "precisión" (Nmap, ZGrab, etc.) para el análisis detallado.
