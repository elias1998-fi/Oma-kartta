# Oma-kartta — Arkkitehtuuridokumentti

> Tämä tiedosto toimii kehitysoppaana: kaikki komponentit, HTML-elementit, JavaScript-funktiot ja CSS-luokat on listattu hakua helpottamaan.

---

## 📁 Tiedostorakenne

```
Oma-kartta/
├── index.html              — Koko sovellus (HTML + CSS + JS, ~3250 riviä)
├── omakartta_style.json    — MapLibre-karttastiili (taustakartta)
├── helsinki_parkit.geojson — Helsingin pysäköintialueiden geodata
├── tampere_parkit.json     — Tampereen pysäköintidatan pohja (tyhjä)
├── README.md               — Lyhyt kuvaus
└── architecture.md         — Tämä tiedosto
```

---

## 🧱 Teknologiapino

| Teknologia | Versio | Käyttötarkoitus |
|---|---|---|
| **MapLibre GL** | 4.x | Interaktiivinen kartta |
| **Supabase** (PostgREST) | — | Tietokanta ja API |
| **Digitransit API** | v1 | Osoitehaku (geocoding) |
| **HTML/CSS/JS** | — | Ei frameworkeja |

### API-avaimet (`index.html` rivi 1341–1345)
```js
const API_KEY = 'a002614565b24559b3feddfca8d01918';          // Digitransit
const SUPABASE_URL = 'https://glphxelazjodpiekmoew.supabase.co';
const SUPABASE_KEY = 'sb_publishable_MEZLuJOl-QsZxgLZI3nnTA_e-xL5VQW';
```

### Supabase-taulut
| Taulu | Kuvaus |
|---|---|
| `parking_submissions` | Käyttäjien lähettämät alueet (status: pending/approved/rejected) |
| `parking_master` | Hyväksytyt alueet (community + hel_halli) |

---

## 🗺️ Karttalayerit

Pääkartta (`map`) — alustettu rivillä **1348**

| Layer ID | Lähde | Kuvaus | Min zoom |
|---|---|---|---|
| `helsinki-parkit-fill` | `helsinki_parkit.geojson` | Helsingin pysäköintialueet (täyttö, sininen) | 13 |
| `helsinki-parkit-outline` | `helsinki_parkit.geojson` | Reunaviiva | 13 |
| `helsinki-parkit-highlight` | `helsinki_parkit.geojson` | Korostettu alue (tummansininen) | 13 |
| `community-areas-fill` | Supabase `parking_master` | Yhteisön lisäämät alueet (vihreä) | 13 |
| `community-areas-outline` | Supabase `parking_master` | Reunaviiva | 13 |
| `community-areas-highlight` | Supabase `parking_master` | Hover-korostus | 13 |
| `halli-icons` | Supabase `parking_master` (source=hel_halli) | Pysäköintihallikuvakkeet | 13 |

Alueen piirtokartta (`areaMap`) — alustettu funktio `initAreaMap()` rivillä **1820**

| Layer ID | Kuvaus |
|---|---|
| `area-poly-fill` | Piirrettävän alueen täyttö |
| `area-poly-line` | Piirrettävän alueen reuna |

Admin-esikartta (`adminPreviewMap`) — alustettu funktio `initAdminPreviewMap()` rivillä **2221**

| Layer ID | Kuvaus |
|---|---|
| `admin-sub-fill` | Hakemuksen alueen esikatselutäyttö (oranssi) |
| `admin-sub-line` | Hakemuksen alueen reuna |

---

## 🖼️ HTML-elementit (tärkeimmät ID:t)

### Kartta ja osoitin
| ID | Rivi | Kuvaus |
|---|---|---|
| `#map` | 769 | Pääkartan kontti |
| `#center-pointer` | 770 | SVG-osoitin kartan keskellä (P-nuppi, väri fill) |

### Navigaatiopalkit
| ID | Rivi | Kuvaus |
|---|---|---|
| `#left-menu-pill` | 775 | Vasen navigaatiopilleripalkki |
| `#menu-btn` | 776 | Hampurilaisvalikko-nappi |
| `#profile-btn` | 782 | Profiili-nappi |
| `#filter-btn` | 788 | Suodatin-nappi |
| `#plus-menu-btn` | 796 | Lisää alue -nappi |
| `#search-btn` | 802 | Haku-nappi |
| `#location-btn` | 809 | GPS-sijaintinappi (oikealla ylhäällä) |

### Overlayit (koko näyttö)
| ID | Rivi | Suunta | Kuvaus |
|---|---|---|---|
| `#profile-overlay` | 815 | ← vasemmalta | Profiili ja asetukset |
| `#search-overlay` | 972 | ← vasemmalta | Hakutoiminto |
| `#plus-overlay` | 986 | ← vasemmalta | Ilmoita alueesta -valikko |
| `#add-parking-overlay` | 1017 | → oikealta | Rajaa pysäköintialue (karttapiirto) |
| `#area-details-overlay` | 1044 | → oikealta | Pysäköintialueen tiedot -lomake |
| `#gps-info-overlay` | 942 | ← vasemmalta | GPS-tarkkuuden selitys |
| `#comment-areas-overlay` | 1166 | → oikealta | Kommentoi alueita (tulossa) |
| `#parking-overlay` | 1269 | sisäänliukuu | Pysäköinnin aloitusruutu |
| `#admin-overlay` | 3222 | → oikealta | Hallintopaneeli (pending-lista) |
| `#admin-detail-overlay` | 3236 | → oikealta | Hakemuksen yksityiskohdat + kartta |

### Tason 2 asetussivut
| ID | Rivi | Kuvaus |
|---|---|---|
| `#account-detail` | 912 | Tilitiedot |

### Bottom sheet
| ID | Rivi | Kuvaus |
|---|---|---|
| `#bottom-sheet` | 1180 | Alhaalta liukuva infopaneeli |
| `#sheet-normal` | 1186 | Normaali sisältö (pysäköintialueen tiedot) |
| `#sheet-filter` | 1195 | Suodatinnäkymä (aikaväli-slider) |
| `#summary-header` | 1188 | Alueen nimi / "Lähennä karttaa" |
| `#summary-type` | 1189 | Pysäköintityyppi |
| `#summary-operator` | 1190 | Operaattori |
| `#sheet-details` | 1193 | HTML-sisältö (aukioloajat, hinta) |
| `#gps-warning` | 1182 | GPS-tarkkuusvaroitus |
| `#drag-handle` | 1181 | Vetokahva |
| `#btn-container` | 1209 | Pohja-CTA (Valitse pysäköintialue) |
| `#action-btn` | 1209 | CTA-nappi |

### Bottom sheetit (modal)
| ID | Rivi | Kuvaus |
|---|---|---|
| `#vehicle-sheet` | 1211 | Ajoneuvojen valinta |
| `#payment-sheet` | 1224 | Maksutavan valinta |
| `#price-detail-sheet` | 1237 | Maksuerittely |
| `#area-type-sheet` | 1125 | Pysäköintialueen tyyppi (kiekko/maksu/ilmainen) |
| `#area-location-sheet` | 1148 | Sijainti (maan päällinen/alainen) |

### Pysäköinti-overlay sisältö
| ID | Rivi | Kuvaus |
|---|---|---|
| `#p-code` | 1276 | Alueen koodi |
| `#p-owner` | 1277 | Omistaja |
| `#p-meta` | 1277 | Tyyppi + max-aika |
| `#plate-text` | 1287 | Rekisterikilven teksti |
| `#vehicle-nickname` | 1289 | Auton lempinimi |
| `#payment-icon` | 1297 | Maksutavan ikoni |
| `#payment-name` | 1298 | Maksutavan nimi |
| `#time-big` | 1307 | Suuri aikaesitys (esim. "1 h 30 min") |
| `#time-end-text` | 1309 | Päättymisaika |
| `#drum-scroll-zone` | 1311 | Canvas-rulla-alue |
| `#drum-canvas` | 1312 | Canvas-rulla |
| `#p-price` | 1324 | Kokonaishinta |
| `#start-btn` | 1328 | Aloita pysäköinti -nappi |

### Alue-piirto-overlay sisältö
| ID | Rivi | Kuvaus |
|---|---|---|
| `#area-map` | 1023 | Alueen piirtokartta |
| `#corner-minus-btn` | 1027 | Poista kulma -nappi |
| `#area-lock-btn` | 1031 | Seuraa/lukitse kartta -nappi |
| `#sat-toggle-btn` | 1032 | Satelliitti-nappi (ei käytössä) |
| `#confirm-area-btn` | 1039 | Vahvista alueen muoto -nappi |

### Alue-tiedot-lomake
| ID | Rivi | Kuvaus |
|---|---|---|
| `#area-type-display` | 1054 | Valittu tyyppi |
| `#area-location-display` | 1063 | Valittu sijainti |
| `#max-parking-section` | 1069 | Max-pysäköintiaika-osio (piilotettu aluksi) |
| `#kiekko-times` | 1081 | Kiekkopysäköinnin aikarivit (piilotettu aluksi) |
| `#area-extra-section` | 1107 | Lisätiedot-tekstikenttä (piilotettu aluksi) |
| `#submit-parking-btn` | 1113 | Lähetä pysäköintipaikka -nappi |

### Kellotaulupoimijat (drum pickers) `#kiekko-times`-osiossa
| ID | Kuvaus |
|---|---|
| `drum-mape-from` | Ma–Pe alkuaika |
| `drum-mape-to` | Ma–Pe loppuaika |
| `drum-la-from` | La alkuaika |
| `drum-la-to` | La loppuaika |
| `drum-su-from` | Su alkuaika |
| `drum-su-to` | Su loppuaika |

### Muut UI-elementit
| ID | Rivi | Kuvaus |
|---|---|---|
| `#splash-screen` | 760 | Latausnäyttö |
| `#submit-toast` | 1118 | Ilmoitus lähetyksen onnistumisesta |
| `#zoom-indicator` | 1178 | Zoom-taso oikealla alhaalla |
| `#osm-attribution` | 1177 | © OpenStreetMap -teksti |
| `#gps-info-overlay` | 942 | GPS-selitysnäyttö |
| `#rotation-toggle` | 903 | Rotaatiotoggle profiilissa |

---

## ⚙️ JavaScript-funktiot

### Kartta ja sijainti
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `transformRequest(url)` | 1346 | Lisää Digitransit API-avaimen pyyntöihin |
| `calculateDistance(lat1,lon1,lat2,lon2)` | 1410 | Haversine-etäisyys km |
| `formatDistance(distanceKm)` | 1421 | Muotoilee "500 m päässä" / "3 km päässä" |
| `kestoToMinutes(kesto)` | 1429 | Muuntaa "2h" tai "30min" minuuteiksi |
| `highlightParking()` | 2833 | Korostaa kartan keskellä olevan alueen |
| `checkParking()` | 2839 | Päivittää bottom sheetin sisällön kartan liikkuessa |
| `updateZoomIndicator()` | 2560 | Päivittää zoom-tasonäyttöä |

### Bottom sheet -apufunktiot
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `showMasterFeatureInSheet(header,type,op,html)` | 2424 | Näyttää halli/yhteisöalueen tiedot bottom sheetissä |
| `releaseMasterLock()` | 2444 | Vapauttaa masterFeatureLocked-lukon |
| `getOwnerName(r)` | 2834 | Muotoilee omistajan nimen |
| `getTypeName(r)` | 2835 | Muotoilee pysäköintityypin |
| `getPriceText(l)` | 2836 | Muotoilee hintainfo-tekstin |
| `formatHours(v)` | 2837 | Muotoilee aukioloajat HTML:ksi |

### Navigaatio (overlayit)
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `closeMenu()` | 1501 | Sulkee vasen-menupillin |
| `closeProfileOverlay()` | 1554 | Sulkee profiilioverlayn |
| `closeSearchOverlay()` | 1558 | Sulkee hakuoverlayn |
| `closePlusOverlay()` | 1564 | Sulkee plus-overlayn |
| `openAddParkingArea()` | 1569 | Avaa pysäköintialueen piirto-overlayn |
| `closeAddParkingArea()` | 1573 | Sulkee piirto-overlayn |
| `openAreaDetails()` | 1576 | Avaa tiedot-overlayn |
| `closeAreaDetails()` | 1579 | Sulkee tiedot-overlayn |
| `openCommentAreas()` | 2499 | Avaa kommentointi-overlayn |
| `closeCommentAreas()` | 2500 | Sulkee kommentointi-overlayn |
| `openGpsInfo()` | 2502 | Avaa GPS-info-overlayn |
| `closeGpsInfo()` | 2506 | Sulkee GPS-info-overlayn |
| `openAccountSettings()` | 2510 | Avaa tilitiedot-overlayn |
| `closeAccountSettings()` | 2514 | Sulkee tilitiedot-overlayn |

### Alueen piirtotyökalu
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `initAreaMap()` | 1820 | Alustaa piirtokartan |
| `initAreaCornersAndPolygon()` | 1804 | Luo oletusnelikulmion kulmamerkit |
| `addPolygonLayers()` | 1590 | Lisää täyttö- ja reunalayer piirtokartalle |
| `updateAreaLayer()` | 1632 | Päivittää polygonin GeoJSON-datan |
| `getAreaGeoJSON()` | 1628 | Palauttaa `areaCoords`-koordinaatit GeoJSON-muodossa |
| `createCornerEl()` | 1603 | Luo kulmapisteen DOM-elementin |
| `createMidpointEl()` | 1612 | Luo keskipiste-DOM-elementin |
| `setupMarkerEvents(marker)` | 1670 | Lisää drag-tapahtumat kulmamarkkerille |
| `handleMinusClick()` | 1690 | Käsittelee miinus-nappipainaluksen |
| `enterRemoveMode()` | 1696 | Aktivoi kulman poistotilan (punainen pulssi) |
| `exitRemoveMode()` | 1720 | Poistuu kulman poistotilasta |
| `removeCornerAt(idx)` | 1734 | Poistaa kulman indeksin mukaan |
| `showMidpointHandles()` | 1746 | Näyttää lisäyspisteet sivujen keskikohdissa |
| `clearMidpointHandles()` | 1744 | Piilottaa lisäyspisteet |
| `insertCornerAt(idx, coord)` | 1760 | Lisää uuden kulman |
| `toggleAreaLock()` | 1772 | Vaihtaa "Seuraa karttaa" / "Lukitse paikkaan" |
| `centerPolygonOnScreen()` | 1778 | Zoomaa kartalle näyttämään koko polygonin |
| `tuoPolygoniKartalle()` | 1786 | Siirtää polygonin kartan keskelle |
| `goToMyLocationInAreaMap()` | 1799 | Lentää sijaintiin piirtokartalla |
| `updateLockBtn()` | 1639 | Päivittää lukitusnapin tilan |
| `updateMinusBtn()` | 1645 | Päivittää miinusnapin tilan |
| `updateAddBtn()` | 1652 | Päivittää plusnapin tilan |
| `updateConfirmBtn()` | 1659 | Päivittää vahvistusnapin tilan |
| `handleConfirmClick()` | 1665 | Vahvistusnapin klikkikäsittelijä |

### Kellotaulupoimijat (Time Drum Pickers)
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `buildTimeDrumCol(values, initIdx, bgColor)` | 1860 | Rakentaa yhden sarakepoimijan |
| `addTDItem(parent, text)` | 1950 | Lisää yhden rivin poimijaan |
| `initTimeDrumPicker(containerId, h, m)` | 1957 | Alustaa yksi aika-poimija (tunti + minuutti) |
| `initAllTimeDrums()` | 1977 | Alustaa kaikki 6 kiekkopysäköinnin poimijaa |

### Alueen tyyppi- ja sijaintivalinnat
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `openAreaTypeSheet()` | 2007 | Avaa tyyppi-bottom-sheet |
| `closeAreaTypeSheet()` | 2010 | Sulkee tyyppi-bottom-sheet |
| `selectAreaType(value, label)` | 2013 | Valitsee alueen tyypin |
| `openLocationSheet()` | 1990 | Avaa sijainti-bottom-sheet |
| `closeLocationSheet()` | 1993 | Sulkee sijainti-bottom-sheet |
| `selectLocation(value, label)` | 1996 | Valitsee sijainnin |

### Lähetys ja toast
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `submitParking()` | 2039 | Lähettää lomakkeen Supabaseen |
| `showToast()` | 2107 | Näyttää onnistumis-toastin |
| `closeToast()` | 2115 | Sulkee toastin |

### Supabase / datan lataus
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `loadApprovedSubmissions()` | 2314 | Lataa hyväksytyt alueet pääkartalle |
| `addHalliLayers(geojson)` | 2449 | Lisää pysäköintihallikuvakkeet kartalle |

### Admin / hallinto
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `openAdminOverlay()` | 2138 | Avaa hallintopaneelin |
| `closeAdminOverlay()` | 2145 | Sulkee hallintopaneelin |
| `loadPendingSubmissions()` | 2150 | Lataa odottavat ilmoitukset Supabasesta |
| `openSubmissionDetail(id)` | 2186 | Avaa yksittäisen ilmoituksen tiedot |
| `closeAdminDetail()` | 2216 | Sulkee ilmoituksen tiedot |
| `initAdminPreviewMap(submission)` | 2221 | Alustaa admin-esikatselukartan |
| `addAdminPreviewLayers(submission)` | 2256 | Lisää layerit admin-esikartalle |
| `approveCurrentSubmission()` | 2267 | Hyväksyy ilmoituksen (→ `rpc/approve_submission`) |
| `rejectCurrentSubmission()` | 2289 | Hylkää ilmoituksen (status → rejected) |

### Haku
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `searchPlaces(query)` | 2651 | Hakee osoitteita Digitransit API:sta |
| `selectPlace(lat,lon,name,addToHistory,layer,locality)` | 2738 | Valitsee hakutuloksen, lentää kartalle |
| `selectHistoryPlace(lat,lon,name,index)` | 2643 | Valitsee historiakohteen |
| `showSearchHistory()` | 2603 | Näyttää hakuhistorian |
| `addToSearchHistory(item)` | 2586 | Lisää kohteen localStorage-historiaan |
| `removeFromSearchHistory(index)` | 2596 | Poistaa kohteen historiasta |
| `getSearchHistory()` | 2577 | Hakee historian localStoragesta |

### Suodatin (filter)
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `openFilter()` | 2787 | Avaa suodatinnäkymän |
| `closeFilter()` | 2798 | Sulkee suodatinnäkymän |
| `applyRangeFilter()` | 2775 | Asettaa aikavälisuodattimen karttalayereihin |
| `updateSliderUI()` | 2774 | Päivittää sliderin visuaalisen tilan |
| `idxToMinutes(i)` | 2772 | Muuntaa slider-indeksin minuuteiksi |
| `formatHourLabel(i)` | 2773 | Muotoilee sliderin tunnisteen |

### Pysäköintioverlay (drum + hinta)
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `openParkingOverlay(p)` | 3126 | Avaa pysäköinti-overlayn valitulle alueelle |
| `goBackToMap()` | 3149 | Palaa pysäköinti-overlaylta karttaan |
| `drawDrum()` | 2921 | Piirtää canvas-rullan |
| `updateDisplay()` | 2966 | Päivittää ajan, hinnan ja napin tilan |
| `formatTimeDisplay(mins)` | 2923 | Muotoilee "1 h 30 min" |
| `formatEndTime(mins)` | 2925 | Muotoilee päättymisajan |
| `calculatePrice(minutes)` | 2898 | Laskee hinnan (parking + service fee + ALV 25,5%) |
| `formatPrice(amount)` | 2909 | Muotoilee "3,50 €" |
| `openPriceDetail()` | 3006 | Avaa maksuerittely-bottom-sheet |
| `closePriceDetail()` | 3016 | Sulkee maksuerittelyn |
| `showMaxLabel()` | 2949 | Näyttää "Max. X h" -labelin |
| `triggerMaxBounce()` | 2957 | Animoi bouncing + värinä max-rajalla |
| `stopInertia()` | 3022 | Pysäyttää rulla-inertian |
| `runInertia()` | 3023 | Jatkaa rulla-inertiaa |

### Ajoneuvo ja maksutapa
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `renderVehicleDisplay()` | 3039 | Päivittää valitun ajoneuvon näytön |
| `renderVehicleOptions()` | 3045 | Renderöi ajoneuvovaihtoehdot |
| `openVehicleSheet()` | 3062 | Avaa ajoneuvo-bottom-sheet |
| `closeVehicleSheet()` | 3062 | Sulkee ajoneuvo-bottom-sheet |
| `addVehicle()` | 3065 | Lisää uuden ajoneuvon (prompt-pohjainen) |
| `renderPaymentDisplay()` | 3096 | Päivittää maksutavan näytön |
| `renderPaymentOptions()` | 3102 | Renderöi maksutapavaihtoehdot |
| `openPaymentSheet()` | 3117 | Avaa maksutapa-bottom-sheet |
| `closePaymentSheet()` | 3117 | Sulkee maksutapa-bottom-sheet |
| `addPaymentMethod()` | 3120 | Lisää maksutavan (demo-alert) |
| `getPaymentIcon(icon)` | 3083 | Palauttaa SVG-ikonin (applepay/card/mobilepay) |

### Asetukset
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `toggleRotationSetting()` | 2537 | Vaihtaa kartan rotaation päälle/pois |
| `openAccountSettings()` | 2510 | Avaa tilitiedot |
| `openNotificationSettings()` | 2518 | Ilmoitukset (alert, tulossa) |
| `openPaymentSettings()` | 2522 | Maksutavat (alert, tulossa) |
| `openPrivacySettings()` | 2526 | Yksityisyys (alert, tulossa) |
| `openLanguageSettings()` | 2530 | Kieli (alert, tulossa) |
| `openHelpSettings()` | 2566 | Ohje (alert, tulossa) |
| `openAboutSettings()` | 2570 | Tietoja (alert, tulossa) |

### Karttamerkkijat (markers)
| Funktio | Rivi | Kuvaus |
|---|---|---|
| `createPinElement()` | 1353 | Luo punainen 3D-pistemerkki (hakutuloksille) |
| `getHistoryIconSVG()` | 1381 | Palauttaa kello-ikonin SVG:n (hakuhistoria) |
| `getPinIconSVG()` | 1392 | Palauttaa pienikokoisen pin-ikonin SVG:n |

---

## 🎨 CSS-luokat

### Layout
| Luokka | Kuvaus |
|---|---|
| `.pill-btn` | Navigaatiopalkin painike (48×48px) |
| `.pill-btn.active` | Aktiivinen tila (tummansininen tausta) |
| `.overlay-header` | Overlayn otsikkorivi |
| `.overlay-back-btn` | Takaisin-painike overlayssa |
| `.overlay-content` | Overlayn sisältöalue |
| `.overlay-title` | Overlayn otsikko |
| `.settings-detail-overlay` | Tason 2 overlay (liukuu oikealta) |
| `.settings-detail-overlay.open` | Avoin tila |

### Asetukset-lista
| Luokka | Kuvaus |
|---|---|
| `.settings-section` | Valkoinen osio asetuksissa |
| `.settings-item` | Yksittäinen asetus-rivi |
| `.settings-item-icon` | Emoji-ikoni rivillä |
| `.settings-item-text` | Tekstialue |
| `.settings-item-label` | Pääotsikko |
| `.settings-item-value` | Alaotsikko / arvo |
| `.settings-item-chevron` | Nuolimerkki (›) |

### Haku
| Luokka | Kuvaus |
|---|---|
| `.search-result-item` | Yksittäinen hakutulos |
| `.search-result-icon` | Ikoni-alue |
| `.search-result-text` | Tekstialue |
| `.search-result-name` | Paikan nimi |
| `.search-result-address` | Kaupunginosa/kunta |
| `.search-result-distance` | Etäisyys |
| `.search-history-header` | "Viimeisimmät haut" -otsikko |
| `.search-history-delete` | Poista historiakohdasta |
| `.search-empty` | "Ei tuloksia" -teksti |

### Bottom sheet ja filter
| Luokka | Kuvaus |
|---|---|
| `.hours-row` | Aukioloaikarivi |
| `.hours-day` | Päivä (esim. "Ma – Pe") |
| `.hours-time` | Aika (esim. "08:00–18:00") |
| `.price-label` | Hinnan otsikkoteksti |
| `.price-value` | Hinnan arvo |
| `.range-container` | Slider-konttiryhmä |
| `.range-track` | Sliderin harmaa taustaviiva |
| `.range-fill` | Sliderin aktiivinen täyttö |
| `.range-input` | Range-inputin tyyli |
| `.slider-ticks` | Sliderin arvojen otsikot |

### Modaalit (bottom sheets)
| Luokka | Kuvaus |
|---|---|
| `.sheet-backdrop` | Läpikuultava tausta |
| `.sheet-panel` | Valkoinen paneeli alhaalta |
| `.sheet-header` | Paneelin otsikkorivi |
| `.sheet-title` | Paneelin otsikko |
| `.sheet-close-btn` | Sulkemispainike |
| `.sheet-divider` | Erotinviiva |
| `.option-row` | Valintarivi |
| `.option-check` | Valintaympyrä |
| `.option-check.selected` | Valittu tila (tummansininen) |
| `.option-name` | Valinnan nimi |
| `.option-reg` | Rekisterinumero |
| `.add-option` | "＋ Lisää uusi..." -rivi |

### Pysäköinti-overlay
| Luokka | Kuvaus |
|---|---|
| `.p-cards-container` | Korttien kontti |
| `.p-card` | Yksittäinen valkoinen kortti |
| `.p-card-title` | Kortin otsikko (isolla, harmaa) |
| `.area-row` | Alueen rivinäkymä |
| `.area-code-badge` | Alueen koodi-badge (tummansininen) |
| `.area-name` | Alueen nimi |
| `.area-meta` | Meta-teksti (tyyppi · max-aika) |
| `.area-select-row` | Valintarivi karttaan palaamiseksi |
| `.vehicle-display` | Ajoneuvo-näyttörivi |
| `.plate` | Rekisterikilven tyyli |
| `.plate-eu` | EU-sininen osa |
| `.plate-text` | Rekisterinumeron teksti |
| `.vehicle-nickname` | Ajoneuvon lempinimi |
| `.payment-display` | Maksutapa-näyttörivi |
| `.payment-icon-box` | Maksutavan ikonilaatikko |
| `.payment-name` | Maksutavan nimi |
| `.time-big-number` | Suuri aikaesitys |
| `.time-big-number.at-max` | Punainen max-tila |
| `.time-big-number.bounce` | Palloanimaatio |
| `.drum-scroll-zone` | Canvas-rullan alue |
| `.total-price-row` | Hintarivi |
| `.total-price-label` | "Yhteensä" + info-ikoni |
| `.total-price-val` | Hinnan summa |
| `.info-icon` | ⓘ pyöreä ikoni |
| `.detail-row` | Erittelyrivi |
| `.detail-row.total` | Yhteensä-rivi (lihavoitu) |

### GPS-varoitus
| Luokka | Kuvaus |
|---|---|
| `.gps-info-icon` | ⓘ GPS-tarkkuuden selitys-ikoni |
| `#gps-warning.visible` | Näkyvä GPS-varoitus |

### Splash-screen
| Luokka | Kuvaus |
|---|---|
| `.splash-content` | Sisältö (auto + P-ympyrä) |
| `.splash-car` | Auto-SVG |
| `.splash-p` | Sininen P-ympyrä |
| `#splash-screen.fade-out` | Katoamis-animaatio |

---

## 🎨 Väripaletti

| Väri | Hex | Käyttö |
|---|---|---|
| Tummansininen (pääväri) | `#0C0243` | Napit, korostukset, karttaelementit |
| Sininen (Helsingin parkit) | `#4A90D9` / `#2B6CB0` | Pysäköintialueiden täyttö ja reuna |
| Vihreä (yhteisöalueet) | `#27ae60` / `#1e8449` | Community-alueet |
| Oranssi (admin-esikatselu) | `#f39c12` / `#e67e22` | Hallinnon esikatselupolygoni |
| Punainen (kulmapisteet, max) | `#e53935` / `#d32f2f` | Remove-tila, max-aika |
| Harmaa tausta | `#f2f3f5` | Overlayjen tausta |
| Valkoinen | `#ffffff` | Kortit, paneelit |

---

## 🔑 Tärkeät globaalit muuttujat (JavaScript)

| Muuttuja | Rivi | Kuvaus |
|---|---|---|
| `map` | 1348 | Pääkartta |
| `areaMap` | 1584 | Piirtokartta |
| `adminPreviewMap` | 2128 | Admin-esikatselukartta |
| `currentMarker` | 1351 | Hakutuloksen pin-markeri |
| `userLocation` | 1408 | Käyttäjän sijainti `{lat, lon}` |
| `areaCoords` | 1584 | Piirrettävän polygonin koordinaatit `[[lng,lat],...]` |
| `areaMarkers` | 1584 | MapLibre-markerit kulmapisteille |
| `areaLocked` | 1585 | `true` = polygon seuraa karttaa |
| `polygonMoved` | 1587 | `true` = käyttäjä on muokannut polygonia |
| `masterFeatureLocked` | 2421 | `true` = halli/yhteisötieto näkyvissä, ei ylikirjoiteta |
| `parkingMinutes` | 2913 | Valittu pysäköintiaika minuutteina |
| `parkingMaxMinutes` | 2913 | Max pysäköintiaika (alueen rajoitus) |
| `parkingStartTime` | 2913 | Milloin pysäköinti alkaa |
| `vehicles` | 3036 | Ajoneuvolista `[{reg, name}]` |
| `selectedVehicleIdx` | 3037 | Valittu ajoneuvo |
| `paymentMethods` | 3076 | Maksutapalista |
| `selectedPaymentIdx` | 3081 | Valittu maksutapa |
| `currentParkingProps` | 2831 | Valitun alueen properties-objekti |
| `currentSubmission` | 2129 | Admin: käsiteltävä ilmoitusobjekti |
| `menuExpanded` | 1492 | Onko vasen menu auki |
| `filterMode` | 2785 | Onko suodatinnäkymä auki |
| `rotationEnabled` | 2536 | Onko kartan rotaatio käytössä |
| `PARKING_RATE` | 2894 | 3 €/h |
| `SERVICE_FEE_PERCENT` | 2895 | 10% |
| `VAT_PERCENT` | 2896 | 25,5% |

---

## 📦 Hintalaskenta (`calculatePrice`, rivi 2898)

```
parkingHinta = (minutes / 60) × 3 €
palvelumaksu = parkingHinta × 10%
välisumma    = parkingHinta + palvelumaksu
ALV          = välisumma × 25,5%
yhteensä     = välisumma + ALV
```

---

## 🔄 Käyttäjävirta (User Flow)

```
Splash → Kartta
         ├── Haku → Tulokset → Pin kartalla
         ├── Vasen menu
         │     ├── Profiili → Asetukset / Hallinto
         │     ├── Suodatin → Aikaväli-slider
         │     ├── + Ilmoita → Piirrä alue → Tiedot → Lähetä
         │     └── Haku
         ├── Bottom sheet (pysäköintialue valittuna)
         │     └── "Pysäköi tähän" → Pysäköinti-overlay
         │           ├── Valitse ajoneuvo
         │           ├── Valitse maksutapa
         │           ├── Aseta aika (drum-rulla)
         │           └── Aloita pysäköinti
         └── Kartan hallipisteet / yhteisöalueet → Bottom sheet info
```
