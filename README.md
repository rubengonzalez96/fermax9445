# Fermax 9445 — apertura remota con ESP32 + ESPHome

Control remoto del botón de apertura del telefonillo Fermax 9445 desde Home Assistant, usando un ESP32 alimentado por la propia placa del telefonillo.

> **Alcance real del proyecto**: solo apertura remota. No hay detección de llamada, no hay audio ni vídeo. Si buscas algo más complejo, este telefonillo no lo permite fácilmente.

---

## Cómo funciona

El Fermax 9445 tiene botones físicos en la placa: uno de **cámara** y uno de **llave** (apertura). Cada botón tiene dos pads en la PCB que, al pulsarlo, se cortocircuitan.

La idea es simple: soldar un optoacoplador PC817 en paralelo a cada botón, de modo que el ESP32 pueda cerrar ese circuito por software, simulando exactamente una pulsación física.

La automatización en Home Assistant hace:
1. Activar el pin de **cámara** (activa el telefonillo)
2. Esperar 1 segundo
3. Activar el pin de **llave** (abre la puerta)

---

## Hardware

- ESP32 (cualquier variante)
- 2× optoacoplador PC817
- Alimentación tomada de la propia placa del Fermax (ver más abajo)

### Pines utilizados

| Pin ESP32 | Función |
|---|---|
| GPIO 18 | Simula botón cámara |
| GPIO 19 | Simula botón llave (apertura) |

### Conexión de los PC817

Cada optoacoplador va soldado en paralelo al botón correspondiente de la placa del Fermax:

```
GPIO ESP32 → resistencia 1kΩ → LED+ del PC817
GND ESP32  →                   LED- del PC817

Colector PC817 → pad 1 del botón Fermax
Emisor   PC817 → pad 2 del botón Fermax
```

Cuando el ESP32 pone el pin en HIGH, el LED del optoacoplador conduce y el fototransistor cierra el circuito del botón — exactamente como si lo pulsaras a mano.

### Alimentación del ESP32

*Pendiente de documentar — el ESP32 se alimenta directamente desde la placa del Fermax.*

---

## Firmware (ESPHome)

El ESP32 corre ESPHome. La configuración es mínima: dos `output` que activan los pines durante un breve pulso.

```yaml
# Fragmento esphome — ajusta a tu configuración
switch:
  - platform: gpio
    pin: GPIO18
    id: btn_camara
    name: "Fermax cámara"
    restore_mode: ALWAYS_OFF

  - platform: gpio
    pin: GPIO19
    id: btn_llave
    name: "Fermax llave"
    restore_mode: ALWAYS_OFF
```

> El archivo completo estará en `firmware/fermax.yaml` cuando lo añada.

---

## Automatización en Home Assistant

La automatización que abre la puerta cuando se dispara manualmente (botón en el dashboard, NFC, lo que quieras):

```yaml
alias: "Fermax — Abrir puerta"
sequence:
  - service: switch.turn_on
    target:
      entity_id: switch.fermax_camara
  - delay: "00:00:01"
  - service: switch.turn_on
    target:
      entity_id: switch.fermax_llave
  - delay: "00:00:00.500"
  - service: switch.turn_off
    target:
      entity_id: switch.fermax_llave
  - service: switch.turn_off
    target:
      entity_id: switch.fermax_camara
```

---

## Limitaciones conocidas

- **No hay detección de llamada** — no sabemos cuándo llaman, hay que abrir manualmente desde HA.
- **No hay audio ni vídeo** — el protocolo del Fermax 9445 no lo expone de forma accesible.
- **Apertura a ciegas** — es tu responsabilidad decidir cuándo abrir.

---

## Fotos

*Pendiente de añadir fotos del montaje.*

---

## Licencia

MIT
