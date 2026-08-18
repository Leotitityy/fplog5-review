# FPLOG5 Absorption - Implementation Notes (v1)

Toteuttaja: Claude. Pohjana ABSORPTION_INDICATOR_PLAN.md. Koodi: fplog5-pine/src/fplog5.pine (269 riv).

## Toteutetut ominaisuudet

Yksi request.footprint(ticksPerRow, vaPercent, imbalancePct) -kutsu. Pisteytys 0-100 erikseen bull- ja bear-suuntaan, viisi osaa: Footprint pressure (0-30, myyntipaine alakolmanneksessa/ostopaine ylakolmanneksessa + delta-z), Rejection/failed auction (0-25, close-sijainti + sweep-and-reclaim-bonus), Location (0-20, PDH/PDL, session high/low, VWAP 1SD-banda, footprint VAL/VAH, swing high/low), Imbalance evidence (0-15, pisin peraytainen imbalance-rivijono), Relatiivinen volyymi (0-10, sama molempiin suuntiin). Anti-noise: min relatiivinen volyymi, min rejection-ratio, cooldown baareissa, signaali vain barstate.isconfirmed-barilla. Kolme visuaalista tasoa (normal/strong bull/bear). Debug/research-taulu oletuksena pois. Ei dataa vs ei absorptiota erotettu.

## Oletusparametrit

ticksPerRow=1, vaPercent=70, imbalancePct=300, lookback=20, normalThresh=55, strongThresh=75, minRelVol=1.0, minRejection=0.35, cooldownBars=3, VWAP anchor=Session, sd1=1.0, sd2=2.0, locLookback=20, locTolTicks=4, showDebugTable=false, showMarkers=true.

## Pine/TV-rajoitukset (OPEN - verifioi Pine Editorissa)

OPEN: footprint.rows() ja volume_row.has_buy_imbalance()/has_sell_imbalance() -nimet perustuu kolmannen osapuolen dokumentaatioon (TradersPost), ei suoraan TV:n omaan kielireferenssiin - tarkista autocompletella.
OPEN: onko fp.poc()/vah()/val()/rows() turvallista kutsua kun fp on na - oletin nain koska buy_volume/sell_volume dokumentoidaan na-turvalliseksi.
OPEN: ta.vwap(source, anchor, stdev_mult) -kolmoispaluu on v5-dokumentoitu, oletin saman v6:ssa.
OPEN: locTolTicks=4 on arvaus, ei testattu NQ:lla.
OPEN: paivittyyko footprint retroaktiivisesti barin sulkeutuessa - dataOk vaatii barstate.isconfirmed varotoimena.

## Tunnetut heikkoudet

Footprint pressure -kertoimet (18x, 4x) ei validoitu datalla. Location-osuma binaarinen, ei asteittainen. Sweep-tunnistus kayttaa rullaavaa min/max:aa, ei oikeaa pivot-logiikkaa. RelVolScore-baseline nz(0)-taytolla vaimentaa na-baareja alaspain. Ei viela MFE/MAE-validoitu.

## Ehdotukset seuraavaan iteraatioon

1. Validoi pressure-kertoimet oikealla NQ-datalla.
2. Korvaa binaarinen location jatkuvalla etaisyyspisteytyksella.
3. Testaa sweep oikealla pivot-logiikalla.
4. Aja plan-vaiheet 4-7 (score-jakaumat, MFE/MAE) ennen kynnysten sovittamista.
5. RelVol-baseline vain saatavilla olevista footprint-baareista.
