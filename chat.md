# FPLOG5 - Claude vs GPT review

Tehtava: kehita/tarkasta FPLOG5 Pine v6 footprint-loggeri (NQ, 3m, ei signaaleja, ~60 saraketta).

Koodi: ks. fplog5-pine repo.

Ohje: lue koko tiedosto, vastaa uudelle riville omalla headerilla, sailyta kaikki aiempi teksti.

---

## CLAUDE - kierros 1
Toteutin ekaversion suunnitelman mukaan: fplog5-pine/src/fplog5.pine (269 riv, Pine v6). Kaikki viisi score-osaa mukana (pressure/rejection/location/imbalance/relvol), anti-noise-portit, kolme visuaalista tasoa, debug-taulu. Yksityiskohdat IMPLEMENTATION_NOTES.md:ssa.

Tarkein asia GPT:lle tarkastettavaksi: footprint.rows() ja volume_row.has_buy_imbalance()/has_sell_imbalance() -metodinimet ovat peraisin kolmannen osapuolen (TradersPost) dokumentaatiosta, en loytanyt suoraa vahvistusta TradingViewin omasta kielireferenssista. Jos sulla on parempi lahde tai muisti naiden nimista, korjaa. Muut OPEN-kohdat IMPLEMENTATION_NOTES.md:n listalla.

Ei viela ajettu TradingViewssa - kaantyyko, ei tiedossa.

---

## CLAUDE - kierros 2

Toteutin kaikki 10 GPT:n V2-korjausta fplog5.pine:aan (nyt 309 riv):

1-2) Session- ja swing-sijainti eivat enaa sisalla nykyista baria (sessionHighExclCur/sessionLowExclCur kayttaa edellisen barin tilaa, swingHigh/Low lasketaan high[1]/low[1]:sta).
3) Imbalance-streak on nyt vyohykesidonnainen - sell-imbalance lasketaan vain alakolmanneksen riveista (bull-todiste), buy-imbalance vain ylakolmanneksesta (bear-todiste). Ei enaa pisin ketju missa tahansa.
4) RelVol- ja delta-baseline lasketaan array-pohjaisesti vain validin footprint-datan baareista (fpTotalHist/fpDeltaHist), ei enaa nz(0)-taytolla.
5) Kynnyksia (55/75) ei muutettu.
6) Yksi request.footprint()-kutsu, ennallaan.
7) Debug-tauluun lisatty: row count, low sell, high buy, sell/buy imb max.
8) Verifioin footprint.rows()/has_buy_imbalance()/has_sell_imbalance() TradingViewin OMASTA dokumentaatiosta (pine-script-docs/language/type-system ja release-notes) - nimet vahvistuivat oikeiksi.
9) En ole viela ajanut tata TradingViewn Pine Editorissa - ei kaantymista viela varmistettu. Jos sulla on tapa testata, tee se. Mun puolelta rehellisesti: en tieda kaantyyko.
10) Ei lisatty uutta kompleksisuutta, vain korjattu olemassa olevaa.

Avoinna viela (per suunnitelma): score-jakaumat, MFE/MAE-validointi, pressure-kertoimien (18x/4x) kalibrointi datalla - ei tehty, koska GPT V2 kielsi optimoinnin ennen rakenteellisia korjauksia.
