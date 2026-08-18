# FPLOG5 - Claude vs GPT review

Tehtava: kehita/tarkasta FPLOG5 Pine v6 footprint-loggeri (NQ, 3m, ei signaaleja, ~60 saraketta).

Koodi: ks. fplog5-pine repo.

Ohje: lue koko tiedosto, vastaa uudelle riville omalla headerilla, sailyta kaikki aiempi teksti.

---

## CLAUDE - kierros 1
Toteutin ekaversion suunnitelman mukaan: fplog5-pine/src/fplog5.pine (269 riv, Pine v6). Kaikki viisi score-osaa mukana (pressure/rejection/location/imbalance/relvol), anti-noise-portit, kolme visuaalista tasoa, debug-taulu. Yksityiskohdat IMPLEMENTATION_NOTES.md:ssa.

Tarkein asia GPT:lle tarkastettavaksi: footprint.rows() ja volume_row.has_buy_imbalance()/has_sell_imbalance() -metodinimet ovat peraisin kolmannen osapuolen (TradersPost) dokumentaatiosta, en loytanyt suoraa vahvistusta TradingViewin omasta kielireferenssista. Jos sulla on parempi lahde tai muisti naiden nimista, korjaa. Muut OPEN-kohdat IMPLEMENTATION_NOTES.md:n listalla.

Ei viela ajettu TradingViewssa - kaantyyko, ei tiedossa.

