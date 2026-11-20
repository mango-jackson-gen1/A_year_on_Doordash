# A Year on DoorDash: My Data and Research Process

Methodology companion to the research story *A Year on DoorDash*.

## The question

How much can be mined from one person's digital footprint? Under the EU's GDPR, people can request a copy of their data. DoorDash falls under US rules, not GDPR, and offers no one-click export. So I built my own record from one year of receipts, **July 1, 2025 to July 31, 2026**, and asked what it reveals about my habits.

## Step 1: Data gathering (web scraping with Tampermonkey)

**Tampermonkey** is a browser add-on that runs small custom scripts on a website. I wrote a script that runs inside my own DoorDash account and does what I would otherwise do by hand: open every past receipt and copy down what it says.

- **Two-page crawl.** The script scrolls the *Orders* page until no more orders load, collecting the link to every receipt. Then it opens each receipt and records the restaurant, each dish with its price and add-ons, every fee, the tip, the total, the date and time, and the delivery address.
- **Pacing.** The script waits a **random 1.5 to 3 seconds** before opening each receipt and pauses about 1.2 seconds while scrolling. That is about the pace of a person reading their own receipts. Firing off requests as fast as possible would look like a bot and could get the session blocked.
- **Resilience.** Progress is saved after every receipt, so an interrupted run resumes where it stopped. The script waits up to 8 seconds for each page to finish loading and retries a failed page up to 3 times before setting it aside.
- **Accuracy checks.** Before running it on the live site, I tested the script on saved copies of receipts. A discounted fee shows two prices, the original and the discounted one, and the script keeps both. A full copy of each receipt's text is saved so any record can be rechecked later.

At the end, it collected **149 receipts**.

## Step 2: Cleaning the data: from 149 receipts to 86 deliveries

Cleaning happened in two stages. **Filtering** decided which receipts belong in the analysis. It removed 44 of the 149 scraped receipts (about 30%) that were outside the time window, not meals, cancelled, or picked up, which left **105 delivery receipts**. **Restructuring and reconciliation** made the remaining receipts comparable. It flattened nested receipts into one table, merged duplicate restaurant names, separated add-ons from base prices, and linked Double Dash receipt pairs. The biggest effect came from Double Dash: **105 receipts collapsed to 86 real deliveries**. Without that step, the delivery count would be overstated by about 22% and the cost per delivery would be understated.

### Stage 1: Filtering (what counts)

| **Step** | **Rule** | **Receipts left** |
|---|---|---|
| Raw scrape | Every receipt in the account | 149 |
| Time window | Keep July 1, 2025 to July 31, 2026 | 136 |
| Not a meal | Drop gift cards and non-food stores (pharmacy, cosmetics, grocery) | 124 |
| Cancelled | Drop orders where nothing was delivered | 123 |
| **Delivery vs. pickup** | Drop pickup orders, since no courier was involved | **105** |

### Stage 2: Restructuring and reconciliation (making rows comparable)

- **Flattening nested data.** Each receipt holds a list of dishes and a list of fees. I flattened these into a **long table** with one row per dish or fee, and tagged each row with its receipt's unique ID so it traces back to its order. *Effect:* spend can be summed by dish, fee type, restaurant, or month.
- **Name cleanup.** Some restaurants appeared under two spellings. A matching test merged them. *Effect:* restaurant counts and the top-restaurant ranking aren't split across duplicates.
- **Separating add-ons from price.** One sandwich seemed to swing between $18 and $21. The $3 was avocado. *Effect:* add-ons don't show up as false price changes.
- **Reconciling Double Dash.** Double Dash adds a second store after checkout, so one delivery produces two receipts. I flagged pairs in two ways: "DoubleDash" printed on the receipt, and receipts billed within the same minute. I then checked each pair against the single card charge, and confirmed a few by hand. *Effect:* 105 receipts become 86 deliveries (105 − 19 = 86).
- **Geocoding.** I added each restaurant's street address and coordinates by hand. *Effect:* this makes the map in Step 3 possible.

## Step 3: Analysis and findings

1. **Total spend.** Adding up every receipt's final total (food, fees, tax, and tip) month by month: **$4,345.83 across 105 delivery orders**.
1. **Where.** Counting orders per restaurant and plotting each one on a map: the **102 Manhattan delivery orders** came from **32 restaurants**, clustered in the **East Village** and around **NoMad/Flatiron**, many within walking distance of home. The most frequent were **Dim Sum Sam (17 orders)** and **Din Tai Fung (8)**.
1. **Double Dash.** Comparing paired receipts and counting by month:
   1. **19 of 86 deliveries** (more than a fifth) were Double Dash. There were none in the first two months, and they peaked at **7 of 16 orders in March 2026**. Use rose sharply in the last six months.
   1. The add-on was the hidden expense, and a more persistent habit than I expected. Across the 18 pairs where both receipts were found, the add-on averaged **$27**, against an average main dinner of about **$40**, so the "extra" came to two-thirds the size of the meal. Over the year, add-ons totaled **$486**. 
   1. The add-ons were mostly dessert. **Lady M's** Amedeo chocolate mille-feuille cake was ordered 4 times (7 pieces), and **ando Patisserie's** Swiss cake roll 5 times (6 pieces).

**Conclusion:** The data shows a habit taking shape. DoorDash's "no extra delivery fee" made a second order feel free, but in practice it encouraged me to order more food and spend more than I meant to on a second meal. Over the one year, that added up to almost $500, mostly on dessert, and often from places I could have walked to.
