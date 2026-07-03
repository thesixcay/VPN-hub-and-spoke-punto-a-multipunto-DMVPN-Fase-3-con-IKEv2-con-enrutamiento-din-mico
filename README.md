# VPN Hub-and-Spoke Punto a Multipunto DMVPN Fase 3 con IKEv2 y OSPF

## Descripción

Este laboratorio implementa una **VPN Dynamic Multipoint VPN (DMVPN) Fase 3** utilizando una arquitectura **Hub-and-Spoke**, donde el tráfico entre sucursales puede establecer comunicación directa gracias a **NHRP Redirect** y **NHRP Shortcut**, evitando que todo el tráfico pase por el Hub.

La solución utiliza:

- DMVPN Fase 3
- GRE Multipoint (mGRE)
- IPSec con IKEv2
- NHRP
- OSPF como protocolo de enrutamiento dinámico
- Cifrado AES-256
- Integridad SHA-256

---

# Objetivos

- Implementar una VPN Hub-and-Spoke mediante DMVPN Fase 3.
- Proteger el tráfico utilizando IPSec con IKEv2.
- Configurar NHRP para el descubrimiento dinámico de vecinos.
- Implementar OSPF para el intercambio automático de rutas.
- Permitir comunicación directa entre los Spokes mediante NHRP Redirect y Shortcut.
- Validar el funcionamiento del túnel y del enrutamiento dinámico.

---

# Tecnologías utilizadas

- Cisco IOS
- DMVPN Fase 3
- GRE Multipoint (mGRE)
- IPSec
- IKEv2
- NHRP
- OSPF Área 0

---

# Topología

```
                    HUB (R1)
              Tunnel 23.6.1.1
                 /         \
                /           \
               /             \
              /               \
             /                 \
     SPOKE1 (R2)         SPOKE2 (R3)
   Tunnel 23.6.1.2      Tunnel 23.6.1.3
```

Todos los Spokes se registran en el Hub mediante NHRP.

Cuando un Spoke necesita comunicarse con otro, el Hub únicamente informa la dirección pública mediante **NHRP Redirect**, y posteriormente la comunicación se establece directamente entre ambos utilizando **NHRP Shortcut**.

---

# Direccionamiento IP

## Enlaces físicos

| Enlace | Router | Interfaz | Dirección IP |
|---------|---------|----------|--------------|
| Hub ↔ Spoke1 | Hub | Ethernet0/0 | 10.0.12.1/30 |
| Hub ↔ Spoke1 | Spoke1 | Ethernet0/0 | 10.0.12.2/30 |
| Hub ↔ Spoke2 | Hub | Ethernet0/1 | 10.0.13.1/30 |
| Hub ↔ Spoke2 | Spoke2 | Ethernet0/0 | 10.0.13.2/30 |
| Spoke1 ↔ Spoke2 | Spoke1 | Ethernet0/1 | 10.0.23.1/30 |
| Spoke1 ↔ Spoke2 | Spoke2 | Ethernet0/1 | 10.0.23.2/30 |

---

## Direccionamiento del túnel DMVPN

| Router | Interfaz | Dirección |
|---------|-----------|-----------|
| Hub | Tunnel0 | 23.6.1.1/24 |
| Spoke1 | Tunnel0 | 23.6.1.2/24 |
| Spoke2 | Tunnel0 | 23.6.1.3/24 |

---

# Componentes principales

## GRE Multipoint

Permite que múltiples routers compartan una única interfaz Tunnel, eliminando la necesidad de crear un túnel por cada sucursal.

---

## NHRP

NHRP (Next Hop Resolution Protocol) mantiene una base de datos donde el Hub conoce las direcciones físicas de todos los Spokes.

Funciones utilizadas:

- Registro dinámico de Spokes.
- Resolución de direcciones.
- Comunicación directa entre sucursales.

---

## DMVPN Fase 3

En esta fase se utilizan dos características fundamentales:

### NHRP Redirect

Configurado únicamente en el Hub.

```
ip nhrp redirect
```

El Hub informa al Spoke cuando existe un camino directo hacia otro Spoke.

---

### NHRP Shortcut

Configurado únicamente en los Spokes.

```
ip nhrp shortcut
```

Permite que el tráfico abandone el Hub y viaje directamente entre sucursales.

---

## IPSec

Todo el tráfico GRE es cifrado utilizando:

- AES-256
- SHA-256
- Modo Transport

---

## IKEv2

Se utiliza para negociar automáticamente la asociación de seguridad IPSec.

Parámetros utilizados:

- AES-CBC-256
- SHA-256
- Diffie-Hellman Grupo 14
- Clave precompartida

---

## OSPF

El intercambio dinámico de rutas se realiza mediante OSPF Área 0.

Cada router anuncia:

- Interfaces físicas
- Interfaz Tunnel0

Tipo de red:

```
point-to-multipoint
```

---

# Funcionamiento del laboratorio

1. Cada Spoke establece un túnel GRE Multipoint hacia el Hub.

2. IKEv2 negocia la asociación de seguridad.

3. IPSec protege el tráfico del túnel.

4. Los Spokes se registran mediante NHRP.

5. OSPF intercambia automáticamente todas las rutas.

6. Cuando un Spoke desea comunicarse con otro:

- El tráfico llega inicialmente al Hub.
- El Hub envía un Redirect.
- El Spoke solicita la dirección del destino mediante NHRP.
- Se crea un túnel directo entre ambos Spokes.
- El tráfico deja de pasar por el Hub.

---

# Seguridad implementada

## IKEv2

- AES-CBC-256
- SHA-256
- Grupo DH 14

---

## IPSec

- ESP
- AES-256
- SHA-256 HMAC
- Transport Mode

---

## Autenticación

NHRP

```
DMVPN123
```

Clave IKEv2

```
CLAVE123
```

---

# Verificación

## Estado de NHRP

```
show ip nhrp
```

---

## Vecinos OSPF

```
show ip ospf neighbor
```

---

## Base de datos OSPF

```
show ip route ospf
```

---

## Estado del túnel

```
show interface Tunnel0
```

---

## Asociación IKEv2

```
show crypto ikev2 sa
```

---

## Asociación IPSec

```
show crypto ipsec sa
```

---

## Estado de DMVPN

```
show dmvpn
```

---

# Pruebas realizadas

- Conectividad Hub ↔ Spoke1
- Conectividad Hub ↔ Spoke2
- Conectividad Spoke1 ↔ Spoke2
- Formación automática de vecinos OSPF
- Registro dinámico mediante NHRP
- Establecimiento automático de asociaciones IKEv2
- Cifrado IPSec activo
- Comunicación directa entre Spokes mediante DMVPN Fase 3

---

# Ventajas de DMVPN Fase 3

- Escalable para gran cantidad de sucursales.
- No requiere túneles GRE individuales.
- Menor carga sobre el Hub.
- Comunicación directa entre sucursales.
- Descubrimiento automático mediante NHRP.
- Compatible con protocolos de enrutamiento dinámico.
- Administración simplificada.

---

# Contramedidas y buenas prácticas

Para fortalecer la seguridad de una implementación DMVPN en producción se recomienda:

- Utilizar claves precompartidas robustas o certificados digitales en IKEv2.
- Restringir mediante ACL qué direcciones IP pueden establecer túneles hacia el Hub.
- Limitar el acceso administrativo mediante SSH en lugar de Telnet.
- Mantener el software Cisco IOS actualizado para corregir vulnerabilidades.
- Monitorear periódicamente las asociaciones IPSec e IKEv2.
- Implementar autenticación fuerte para NHRP y renovar las claves periódicamente.
- Aplicar listas de control de acceso para permitir únicamente el tráfico necesario por el túnel.

---

# Resultados

La implementación permitió:

- Crear una infraestructura VPN Hub-and-Spoke completamente funcional.
- Establecer túneles GRE Multipoint protegidos con IPSec e IKEv2.
- Intercambiar rutas automáticamente mediante OSPF.
- Optimizar el tráfico entre sucursales utilizando DMVPN Fase 3.
- Garantizar la confidencialidad e integridad del tráfico mediante cifrado AES-256 y SHA-256.

---

# Autor

Repositorio desarrollado con fines educativos para prácticas de redes y seguridad utilizando Cisco IOS, DMVPN Fase 3, IPSec, IKEv2 y OSPF.
