# Sensor-driven-car
# 🤖 Arduino Robot Autó

> Vonalkövető, akadálykerülő és infravörös távirányítóval vezérelhető robot — Arduino UNO + L298N motorvezérlő alapokon.


---

## 📑 Tartalomjegyzék

- [Áttekintés](#-áttekintés)
- [Funkciók](#-funkciók)
- [Hardver](#-hardver)
- [Bekötés és portkiosztás](#-bekötés-és-portkiosztás)
- [Szükséges könyvtárak](#-szükséges-könyvtárak)
- [Telepítés és feltöltés](#-telepítés-és-feltöltés)
- [Használat](#-használat)
- [Állapotgép](#-állapotgép)


---

##  Áttekintés

Egy négykerék-hajtású robot autó, amely három üzemmódban tud működni:

- **Vonalkövetés** — talajra rajzolt fekete vonal mentén halad
- **Akadálykerülés** — autonóm módon halad előre, akadálynál megáll és kerülő manővert végez
- **Kézi vezérlés** — infravörös távkapcsolóval irányítható

A mozgásokat egy nem-blokkoló, megszakítható állapotgép kezeli, így a felhasználó bármikor megállíthatja vagy módot válthat a kocsin a távkapcsolóval.

---

## Funkciók

- ✅ Öt állapotú állapotgép (`OFF` / `STANDBY` / `LINE_FOLLOW` / `OBSTACLE_AVOID` / `MANUAL`)
- ✅ Autonóm akadálykerülés szervo + ultrahangos szenzor kombóval, 3-pontos pásztázással
- ✅ Kézi távirányítás 5 iránygombbal
- ✅ Mediánszűrt ultrahangos távolságmérés a zajos jelek kiszűréséhez
- ✅ Megszakítható mozgások (távkapcsoló parancsa bármikor leállíthat egy manővert)
- ✅ Soros monitor naplózás minden lényeges eseményről
- ⚠️ Vonalkövetés — megírva, de finomhangolás szükséges (lásd [Ismert problémák](#-ismert-problémák))

---

##  Hardver

| Alkatrész | Mennyiség | Megjegyzés |
|---|---|---|
| Arduino UNO (ATmega328P) | 1 | Vezérlő |
| L298N motorvezérlő modul | 1 | Kettős H-híd |
| Egyenáramú motor (sárga, hajtóműves) | 4 | Két oldalra párhuzamosan |
| Szenzorpajzs (Sensor Shield V5.0) | 1 | Egyszerűbb bekötéshez |
| HC-SR04 ultrahangos szenzor | 1 | Távolságmérés |
| SG90 szervomotor | 1 | Az ultrahangos szenzort forgatja |
| Háromcsatornás vonalkövető modul | 1 | Bal / közép / jobb IR-reflexiós szenzor |
| Infravörös vevő (VS1838B) | 1 | 38 kHz |
| 21 gombos infravörös távkapcsoló | 1 | Mellékelt |
| 18650 Li-ion akkumulátor | 2 | Sorba kötve (~7,4 V) |
| Négykerekű alváz | 1 | Szerelvénnyel |

**Tápellátás:** a motorvezérlő és az Arduino külön kapja a tápot, **de a földpotenciáljuk (GND) közösítve van**.

---

## Bekötés és portkiosztás

### Motorvezérlés (L298N)

| Jel | Arduino láb | Funkció |
|---|---|---|
| `ENA` | **D5** *(PWM)* | Bal oldal sebessége |
| `IN1` | **D2** | Bal oldal irány A |
| `IN2` | **D4** | Bal oldal irány B |
| `IN3` | **D7** | Jobb oldal irány A |
| `IN4` | **D8** | Jobb oldal irány B |
| `ENB` | **D10** *(PWM)* | Jobb oldal sebessége |

### Szenzorok és kiegészítők

| Modul | Jel | Arduino láb |
|---|---|---|
| Vonalkövető — bal | `L_SENS` | **D6** |
| Vonalkövető — közép | `M_SENS` | **D9** |
| Vonalkövető — jobb | `R_SENS` | **D11** |
| Ultrahang — Trig | `TRIG` | **A1** |
| Ultrahang — Echo | `ECHO` | **A0** |
| Szervo jel | `SERVO` | **D3** *(PWM)* |
| Infravörös vevő | `RECV` | **D12** |


---


##  Használat

A robot bekapcsolás után az `OFF` állapotban van, és nem reagál a motorparancsokra. Mindig `CH-` gombbal indul, és `CH` gombbal teljesen kikapcsolható.

### Vezérlő gombok

| Gomb | HEX kód | Funkció |
|---|---|---|
| **CH-** | `0x45` | Bekapcsolás → `STANDBY` |
| **CH** | `0x46` | Kikapcsolás → `OFF` |
| **CH+** | `0x47` | Kilépés az aktuális módból → `STANDBY` |
| **PREV** | `0x44` | Vonalkövetés indítása |
| **NEXT** | `0x40` | Akadálykerülés indítása |
| **PLAY** | `0x43` | Kézi mód indítása |
| **5** | `0x1C` | STOP (megáll, de `MANUAL`-ban marad) |

### Kézi irányítás (csak `MANUAL` módban)

| Gomb | HEX kód | Akció |
|---|---|---|
| **2** | `0x18` | ⬆️ Előre |
| **8** | `0x52` | ⬇️ Hátra |
| **4** | `0x08` | ⬅️ Balra |
| **6** | `0x5A` | ➡️ Jobbra |
| **5** | `0x1C` | ⏹️ Megáll |

### Tipikus használat

```
CH-              → bekapcsolás (STANDBY)
PLAY             → kézi mód
2 → 6 → 5        → előre, jobbra, megáll
CH+              → vissza STANDBY-ba
NEXT             → akadálykerülés indul
CH               → teljes kikapcsolás
```

---

## 🧭 Állapotgép

```
       ┌──────┐  CH-   ┌─────────┐
       │ OFF  │ ─────▶ │ STANDBY │ ◀───── bármely mód: CH+ / STOP
       └──────┘        └────┬────┘
          ▲                 │
          │ CH              │ PREV       NEXT          PLAY
          │                 ▼            ▼             ▼
          │          ┌──────────┐  ┌────────────┐  ┌────────┐
          └──────────│LINE_FOLL.│  │OBSTACLE_AV.│  │ MANUAL │
                     └──────────┘  └────────────┘  └────────┘
```

| Állapot | Mit csinál? |
|---|---|
| `OFF` | Teljes nyugalom, csak `CH-` indítja el. |
| `STANDBY` | Áll, parancsra vár. |
| `LINE_FOLLOW` | Követi a fekete vonalat. |
| `OBSTACLE_AVOID` | Önállóan halad, akadálynál kerül. |
| `MANUAL` | Csak a távkapcsoló nyilai mozgatják. |

