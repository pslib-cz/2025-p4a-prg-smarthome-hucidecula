# IoT projekt - 4D imerzivní Minecraft místnost

## Nápad

Projekt bude tvořen ve formě místnosti v malém měřítku (kartonové krabice), která bude sloužit jako simulace imerzivního Minecraft zážitku. Podle stavu různých proměnných v našem Minecraft serveru se budou měnit podmínky ("počasí") v naší malé místnosti. Na základě denní doby v Minecraftu se bude měnit intenzita osvětlení, počasí bude ovlivňovat foukání větráku a podle aktuálně navštíveného biomu se budou měnit barvy LED pásku. Místnost bude z jedné strany otevíratelná, aby do ní šlo nahlédnout.

## Jak to bude fungovat

Minecraft server každých 5 sekund zapíše aktuální stav hry (biom, počasí, den/noc) do JSON souboru. Home Assistant běžící na Raspberry Pi si tento soubor stahuje a porovnává se scénami. Když se stav změní, HA pošle přes MQTT příkaz ESP32, které okamžitě přenastaví barvu LED pásku a rychlost ventilátoru. Celý systém běží lokálně, bez internetu. Dashboard na HA slouží k monitorování a manuálnímu ovládání.

- integrace: **2** (ESPHome + Minecraft REST)
- entity: **7** (LED, ventilátor, teplota, vlhkost, biom, počasí, den/noc)
- scénáře: **5** (les, déšť, bouřka, noc, Nether)
- dashboard pro monitoring a interakci
- zabebzpečená komunikace (MQTT s TLS certifikátem + HTTPS)
- vše řízeno lokálně přes HA, bez závislosti na cloudu

<img width="874" height="360" alt="Image" src="https://github.com/user-attachments/assets/461ef961-7c3f-43f7-9123-9ebf0746914e" />

## Výstup projektu

- **Fyzický výstup:** krabice s otvorem na jedné straně, uvnitř bude LED pásek a ventilátor
- **Softwarový výstup:** běžící Home Assistant na RPi s dashboardem přístupným přes prohlížeč (HTTPS), kde je vidět aktuální stav hry, teplota a vlhkost uvnitř krabice a možnost manuálního ovládání LED a ventilátoru

## Součástky

- Raspberry Pi 4
- MicroSD karta
- ESP32 Devkit C
- WS2812B LED pásek
- Ventilátor 5V 30x30mm
- MOSFET IRLZ44N
- DHT22 senzor
