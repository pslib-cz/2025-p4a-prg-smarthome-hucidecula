# Imerzivní Minecraft místnost

Projekt IoT automatizace propojující Minecraft Paper server s fyzickou chytrou
místností. Stav hry (aktuální biom, vlhkost, teplota hráče) se promítá do
reálného prostředí prostřednictvím WS2812B LED pásku s dynamickými efekty, a
naopak - podmínky ve fyzické místnosti (teplota, vlhkost) ovlivňují dění
v herním světě (počasí, oheň). Vše běží lokálně přes Home Assistant, bez
závislosti na cloudových službách.

## Koncept

Fyzickým výstupem je kartónová krabice otevíratelná z jedné strany, uvnitř
které je instalován adresovatelný LED pásek (60 LED) a senzor teploty a
vlhkosti DHT22. Krabice představuje miniaturní "místnost", která reaguje na
herní svět.

Obousměrná integrace s Minecraftem je realizována přes RCON protokol -
Home Assistant se dotazuje Paper serveru na herní stav (PlaceholderAPI plugin,
placeholder `%player_biome%`) a současně posílá herní příkazy zpět (`weather
rain`, `setblock`, atd.). 

Klíčové komponenty:

- **Home Assistant** na Raspberry Pi 5 jako centrální řídící jednotka. Běží
  nad ním ESPHome Device Builder pro správu firmware ESP32 a Docker kontejner
  s Paper serverem.
- **ESP32-S3 DevKit** jako IoT klient. Ovládá LED pásek a čte data z DHT22.
  Komunikace s HA probíhá přes nativní ESPHome API (Noise protocol, ekvivalent
  TLS).
- **Paper MC server** v Docker kontejneru běžící na stejném RPi. HA se ho ptá
  na herní stav přes RCON tunelovaný přes SSH s key-based autentizací.
- **Dashboard** v prohlížeči přes HTTPS, rozdělený na dva samostatné pohledy
  (Monitoring pro sledování, Ovládání pro interakci).

## Jak to funguje

Home Assistant každé 3 sekundy volá přes SSH tunel RCON příkaz
`papi parse UsernameHrace %player_biome%` na Paper serveru. Placeholder API
vrátí aktuální biom hráče (např. `JUNGLE`, `NETHER_WASTES`, `RIVER`),
který se uloží do entity `sensor.minecraft_biome`. Podle jeho hodnoty HA
spustí automatizaci, která přiřadí LED pásku odpovídající vizuální chování
- statickou barvu pro většinu biomů, nebo vlastní animovaný efekt pro
specifické scény (voda, Nether).

Opačný tok: DHT22 senzor uvnitř krabice čte teplotu a vlhkost, hodnoty
posílá přes ESPHome API do HA. Automatizace pak vyhodnotí prahové hodnoty
(konfigurovatelné z dashboardu) a pošle RCON příkaz zpět na Minecraft
server - například spustí déšť (`weather rain`), když vlhkost překročí
danou hranici, nebo nekonečně zapaluje hráče (`setblock ~ ~ ~ minecraft:fire`) při
překročení teplotního prahu.

## LED efekty

LED pásek podporuje dva vlastní efekty implementované jako lambda funkce
v ESPHome firmware:

- **Water Ripple** - pro vodní biomy (ocean, river, shore). Sinusové vlny
  modré barvy s pomalu posouvající se fází, vizuální evokace vodní hladiny.
- **Nether Flames** - pro Nether biomy (nether_wastes, crimson, basalt,
  soul_sand). Simulace ohně pomocí modelu tepelných gradientů - každá LEDka
  má svou "teplotu", která v čase klesá, náhodně vznikají nové plameny a
  teplo se rozšiřuje na sousední pixely. Barva odpovídá fyzikálnímu modelu
  žhnutí (černá → červená → oranžová → žlutá).

Všechny přechody mezi barvami probíhají plynule (transition 2 sekundy),
aby nebyly skokové a podporovaly imerzivní dojem.

## Dashboard

Rozhraní je rozděleno na dva specializované dashboardy:

**Monitoring** - pohled určený pro sledování dat. Zobrazuje aktuální biom,
teplotu a vlhkost v místnosti, 24hodinové grafy historie obou veličin, a
logbook s nedávnými událostmi (změny biomu, spouštění automatizací).

**Ovládání** - pohled pro interakci. Obsahuje manuální ovládání LED pásku
(barva, jas, efekt), tlačítka pro spuštění jednotlivých scén, slidery pro
nastavení prahových hodnot automatizací (práh teploty, vlhkosti pro déšť)
a administrativní akce (restart Minecraft serveru, vypnutí LED).

## Zabezpečení

Projekt implementuje vícevrstvé zabezpečení:

- **HTTPS** na HA dashboardu - self-signed TLS certifikát, RSA 2048, SHA256
- **Šifrovaná komunikace ESP32 ↔ HA** - Noise protokol v ESPHome API
- **OTA firmware updates** chráněné heslem
- **RCON** na Minecraft serveru s dlouhým náhodně generovaným heslem
- **SSH tunel** pro RCON komunikaci mezi HA a Docker kontejnerem,
  autentizace přes RSA klíč s právy 600 (bez hesla)
- **WPA2-PSK** na WiFi síti
- **Lokální provoz** bez cloudových služeb, nulová expozice do internetu
