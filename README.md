# VPN Site to Site - Basada en Políticas (IPSec IKEv2)

**Autor:** Roger Rodriguez  
**Matrícula:** 2025-0757  
**Fecha:** Julio 2026  
**Link:** https://youtu.be/v7FqvLsUuSY

---

## Objetivo del laboratorio

Demostrar la implementación de una VPN site to site punto a punto **basada
en políticas**, utilizando **IPSec con IKEv2**, entre dos routers peer (R1
y R2) separados por un router ISP de tránsito. Este script complementa al
Script 1 (mismo modelo de VPN, pero con IKEv1), demostrando que el modelo
basado en políticas es independiente de la versión de IKE utilizada.

---

## Objetivo de la configuración

1. Establece la asociación de seguridad IKEv2 mediante perfiles y keyrings,
   con cifrado AES-CBC 256, integridad SHA-256, autenticación por llave
   precompartida y grupo Diffie-Hellman 14
2. Define el Transform-Set IPSec con AES 256 y SHA-256 en modo túnel
3. Identifica el tráfico "interesante" mediante una ACL extendida (102)
4. Aplica un `crypto map` (referenciando el perfil IKEv2) sobre la
   interfaz WAN física

---

## Parámetros usados

| Parámetro | Valor | Descripción |
|---|---|---|
| `IKE` | `IKEv2` | Protocolo de intercambio de llaves |
| `ENCRYPTION` | `AES-CBC 256` | Cifrado |
| `INTEGRITY` | `SHA-256` | Integridad |
| `AUTH` | `pre-share` | Llave precompartida |
| `GROUP` | `14` | Grupo Diffie-Hellman |
| `TRANSFORM-SET` | `esp-aes 256 esp-sha256-hmac` | Modo túnel |
| Tráfico interesante (ACL 102) | 10.7.57.32/27 <-> 10.7.57.64/27 | |
| Interfaz de aplicación | FastEthernet0/0 | Ambos routers |

---

## Requisitos para utilizar la configuración

### Software / Plataforma
- EVE-NG
- Cisco IOS c7206vxr

### Acceso a modo de configuración
```bash
enable
configure terminal
```

---

## Documentación del funcionamiento

### ¿Cómo funciona esta variante?

Igual que la VPN basada en políticas con IKEv1 (Script 1): el tráfico a
cifrar se define con una ACL, y un `crypto map` sobre la interfaz física
intercepta y cifra el tráfico que coincide. La única diferencia es la
sintaxis de la Fase 1: en vez de una política ISAKMP monolítica, IKEv2 usa
un `crypto ikev2 proposal`, `crypto ikev2 policy`, `crypto ikev2 keyring` y
`crypto ikev2 profile` separados, y el crypto map referencia el perfil con
`set ikev2-profile` en lugar de depender de la llave ISAKMP global.

### Flujo de la VPN

```
PC1 --ping 10.7.57.65--> R1 (ACL 102 coincide) --cifra con crypto map-->
      ESP (10.7.57.1 -> 10.7.57.6) --> ISP --> R2 (descifra) --> PC2
```

---

## Documentación de la red

### Topología

|<img width="410" height="329" alt="image" src="https://github.com/user-attachments/assets/6fde71d5-9fd2-47e6-8984-c3cb664677ef" />
|

### Interfaces y direccionamiento

| Dispositivo | Interfaz | IP | Conecta a |
|---|---|---|---|
| R1 | Fa0/0 | 10.7.57.1/30 | ISP Fa0/0 |
| R1 | Fa1/0 | 10.7.57.33/27 | SW1 |
| ISP | Fa0/0 | 10.7.57.2/30 | R1 |
| ISP | Fa1/0 | 10.7.57.5/30 | R2 |
| R2 | Fa0/0 | 10.7.57.6/30 | ISP Fa1/0 |
| R2 | Fa1/0 | 10.7.57.65/27 | SW2 |
| PC1 | eth0 | 10.7.57.34/27 | SW1 |
| PC2 | eth0 | 10.7.57.66/27 | SW2 |

---

## Aplicación de la configuración

```bash
# Clonar el repositorio
git clone https://github.com/tu-usuario/vpn-politicas-ikev2

# Entrar al directorio
cd vpn-politicas-ikev2

# Copiar y pegar R1.txt en la consola de R1
# Copiar y pegar R2.txt en la consola de R2
```

### Resultado esperado
```
R1#show crypto ikev2 sa
Status: READY
Encr: AES-CBC, keysize: 256, Hash: SHA256, DH Grp:14

R1#show crypto ipsec sa
#pkts encaps: >0, #pkts decaps: >0
```

---

## Nota técnica

Durante la implementación se detectó un error de operación: al reutilizar
la topología física de scripts anteriores, un bloque de configuración
destinado a R2 se aplicó por error una segunda vez en R1, generando dos
peers y una ACL con entradas duplicadas en el mismo router. Se corrigió
removiendo las líneas del peer incorrecto (`no peer`, `no match identity`,
`no set peer`) y reconstruyendo la ACL con una sola entrada por sentido.
Recomendación: verificar siempre `show running-config | include hostname`
antes de pegar configuración en topologías con routers similares.

---


