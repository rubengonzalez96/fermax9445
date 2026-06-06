# Fermax 9445 — apertura remota con ESP32 + ESPHome

Control remoto del botón de apertura del telefonillo Fermax 9445 desde Home Assistant, usando un ESP32 alimentado por la propia placa del telefonillo.

> **Alcance real del proyecto**: solo apertura remota. No hay detección de llamada, no hay audio ni vídeo. Si buscas algo más complejo, este telefonillo no lo permite fácilmente.

---

## Cómo funciona

El Fermax 9445 tiene botones físicos en la placa: uno de **cámara** y uno de **llave** (apertura). Cada botón tiene dos pads en la PCB que, al pulsarlo, se cortocircuitan.

La idea es simple: soldar un relé de estado sólido AQY210 en paralelo a cada botón, de modo que el ESP32 pueda cerrar ese circuito por software, simulando exactamente una pulsación física.

La automatización en Home Assistant hace:
1. Activar el pin de **cámara** (activa el telefonillo)
2. Esperar 1 segundo
3. Activar el pin de **llave** (abre la puerta)

---

## Hardware

- ESP32 (cualquier variante)
- 2× relé de estado sólido [AQY210](https://es.aliexpress.com/item/1005009058582391.html?spm=a2g0o.productlist.main.2.642253b088SiRU&aem_p4p_detail=202511130344231234185261474500000038759&algo_pvid=35455d73-76f4-4f5b-87aa-74443dad6541&pdp_ext_f=%7B%22order%22%3A%224%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005009058582391%7C_p_origin_prod%3A&search_p4p_id=202511130344231234185261474500000038759_1)
- Alimentación tomada de la propia placa del Fermax (ver más abajo)

### Pines utilizados

| Pin ESP32 | Función |
|---|---|
| GPIO 18 | Simula botón cámara |
| GPIO 19 | Simula botón llave (apertura) |

### Conexión de los AQY210

Cada relé va soldado en paralelo al botón correspondiente de la placa del Fermax:

```
GPIO ESP32 → LED+ del AQY210
GND ESP32  → LED- del AQY210

Colector AQY210 → pad 1 del botón Fermax
Emisor   AQY210 → pad 2 del botón Fermax
```

Cuando el ESP32 pone el pin en HIGH, el LED del relé conduce y el fototransistor cierra el circuito del botón — exactamente como si lo pulsaras a mano.

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

La automatización que abre la puerta cuando se dispara manualmente:

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

## Limitaciones del proyecto

- **No hay detección de llamada** — no sabemos cuándo llaman, hay que abrir manualmente desde HA.
- **No hay audio ni vídeo** — el protocolo del Fermax 9445 no lo expone de forma accesible.
- **Apertura a ciegas** — es tu responsabilidad decidir cuándo abrir.

---

## Fotos

*Pendiente de añadir fotos del montaje.*

---

## Licencia

MIT
