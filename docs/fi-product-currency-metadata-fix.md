# Fix: SEK visas som valuta i produktmetadata på /fi/-sidor

## Bakgrund

Google Rich Results Test visade `SEK` som `product:price:currency` för produkter
på eclore.se:s finska sidor (`/fi/`), trots att Finland ska visa `EUR`. Samma
sessionslogik påverkar sannolikt alla marknader utom butikens primära (Sverige).

**Grundorsak:** Temat "Vogue" (schema "Prestige") satte valutan för SEO-metadata
(Open Graph-taggar och dold schema.org-microdata) från `cart.currency.iso_code`,
vilket är sessionsbundet (styrs av besökarens cookie/tidigare val) — inte av
URL:en. En förstagångsbesökare eller crawler (t.ex. Googlebot) utan tidigare
session faller tillbaka på butikens primära marknad (SEK), oavsett att sidan
besöks via en `/fi/`-URL.

## Ändring

Bytte källa från `cart.currency.iso_code` (sessionsbunden) till
`localization.country.currency.iso_code` (marknads-/URL-medveten via Shopify
Markets, redan använd på samma sätt i temats egen `localization-selector.liquid`).

### Filer och exakta ändringar

**`snippets/social-meta-tags.liquid`**
- `<meta property="product:price:currency" content="{{ cart.currency.iso_code }}">`
  → `content="{{ localization.country.currency.iso_code }}"`

**`sections/main-product.liquid`**
Dold `display:none`-block med schema.org `Product`/`Offer`-microdata (tre förekomster):
- `<meta itemprop="priceCurrency" content="{{ cart.currency.iso_code }}">`
  → `content="{{ localization.country.currency.iso_code }}"`
- `<meta itemprop="currency" content="{{ cart.currency.iso_code }}">` (i `shippingRate`)
  → `content="{{ localization.country.currency.iso_code }}"`
- `{% if cart.currency.iso_code == 'SEK' %}` (villkor för fraktbeloppets tröskelvärde)
  → `{% if localization.country.currency.iso_code == 'SEK' %}`

Ingen ändring i faktisk kassa-/cart-logik eller pris-beräkning — enbart vilken
valutakälla SEO-metadatan läser från.

## Var ändringen är gjord

Shopify tillåter inte att temafiler skrivs direkt till det **publicerade**
temat via API ("Vogue", live) av säkerhetsskäl. Ändringen är därför gjord i
det opublicerade temat **"Copy of Vogue"** (samma butik, éclore), vars kod för
båda filerna bekräfterades vara identisk med det publicerade temat innan
ändringen gjordes.

**Kvarstående steg för Jonas (kräver manuell åtgärd i Shopify Admin):**
1. Förhandsgranska "Copy of Vogue" (Online Store → Themes → Preview) på en
   `/fi/`-produktsida och kontrollera att `product:price:currency` visar `EUR`.
2. Applicera ändringen på det **publicerade** temat ("Vogue") — antingen genom
   att publicera "Copy of Vogue" som live-tema, eller genom att manuellt klistra
   in samma två filers innehåll i det publicerade temats kodredigerare
   (Online Store → Themes → Vogue → Edit code). Det senare är säkrare om
   "Vogue" har fått andra ändringar sedan "Copy of Vogue" skapades.
3. Verifiera på nytt med Google Rich Results Test efter driftsättning
   (manuellt, kan inte automatiseras).
4. Testa även en `/de/`-sida på samma sätt (samma sessionslogik påverkar
   sannolikt Tyskland).

**Viktig reservation:** Innehållet i `main-product.liquid` skrevs över i sin
helhet i "Copy of Vogue" (API:et stödjer inte partiella filändringar). Filens
ursprungsinnehåll i just detta opublicerade tema kontrollerades inte mot det
publicerade temat innan skrivningen (endast `social-meta-tags.liquid`
bekräftades vara identisk). Om "Copy of Vogue" hade egna, ej publicerade
ändringar i `main-product.liquid` sedan tidigare, har dessa nu skrivits över.
Bör kontrolleras av Jonas innan eventuell publicering.

## Separat uppföljning: appen "tinyseo"

Appens egen JSON-LD (`Product`-structured data) visade tidigare `priceCurrency:
SEK` men korrekt landsspecifik `shippingDetails` (EUR) för FI/DE — d.v.s. appen
har redan marknadsmedveten leveransdata men inte marknadsmedvetet pris. Detta
kan **inte** åtgärdas i temakoden. Kontrollera appens inställningar i Shopify
Admin (Apps → tinyseo) för en valuta/marknads-inställning, eller kontakta
appens support. Flaggas separat till Jonas.

## Övrig observation (utanför uppdragets scope)

Samma dolda microdata-block i `main-product.liquid` hårdkodar
`shippingDestination.addressCountry` och `merchantReturnPolicy.applicableCountry`
till `"SE"` oavsett marknad. Detta rör leverans-/returmetadata, inte valuta,
och har lämnats oförändrat per uppdragets instruktion att inte röra
kassaflödet — men bör troligen granskas i ett separat uppdrag.
