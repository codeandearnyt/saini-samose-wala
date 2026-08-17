# Saini Samose Wala — WhatsApp Bot Flow Script

> Works with **Botpress** (paste into Flows builder), **Voiceflow** (recreate nodes), or any webhook-compatible chat tool. Trigger: customer sends any message to the shop's WhatsApp Business number.

---

## 1. GREETING (entry)

**Bot:**
```
Namaste! 🙏 Welcome to Saini Samose Wala Kaithal — since 1994, Famous Pehowa Chowk wala!

How can we help you today?

[1] Menu & Today's Specials
[2] Book Bulk / Party Catering Order
[3] Shop Timings & Location
```

---

## 2. OPTION 1 → MENU

**Bot sends:**
```
🍽️ SAINI SAMOSE WALA — MENU

🥟 Samosa & Pakora
• Special Saini Samosa — ₹15/pc
• Paneer Bread Pakora — ₹25/pc

🍮 Sweets & Dhokla
• Special Dhokla — ₹180/kg
• Gulab Jamun — ₹220/kg

🥤 Beverages
• Spl. Malai Lassi — ₹50/glass

✅ Today's Special: Fresh Special Saini Samosa (hot pack from 8 AM)

👉 Order online: <MENU_URL>
```

**Fallback:** If customer says "menu", "special", "1" → send this block.

---

## 3. OPTION 2 → BULK / PARTY CATERING (4-step flow)

### Step A — Event
**Bot:** `Konsa event plan kar rahe ho? (A) Wedding / (B) Birthday / (C) Function`
- Capture answer → variable `event`

### Step B — Date & Headcount
**Bot:** `Event ki date aur approximate plates/people batao. Example: "18 August, ~300 plates"`
- Capture → variable `headcount`

### Step C — Preferred Items
**Bot:** `Kaunsa items chahiye? (A) Samosa (B) Sweets (C) Snacks & Lassi (D) Mixed platter — ek saath sab bata sakte ho`
- Capture → variable `items`

### Step D — Summary + Alert to Owner
**Bot:**
```
📋 Aapki party order summary:
• Event: {event}
• Date & Plates: {headcount}
• Items: {items}

Sahi hai toh [YES] bolo, warna [NO] se dobara batao.
```
On **YES**:
```
Shukriya! 🙏 Aapki bulk enquiry register ho gayi.
Hamari team 5 min me WhatsApp pe reply karegi bulk rates ke saath!
```

**🔔 OWNER ALERT (automated webhook):** after YES, fire a webhook to notify the shop:

- **CallMeBot (free, no-code):**
  `https://api.callmebot.com/whatsapp.php?phone=919350549635&text=NEW BULK ENQUIRY | Event: {event} | Date: {headcount} | Items: {items}`
  *(register the owner's number at callmebot.com first; accepts one-time/limited messages)*

- **Twilio / Meta WhatsApp Business Cloud API (reliable, production):**
  ```json
  POST https://graph.facebook.com/v19.0/<PHONE_NUMBER_ID>/messages
  {
    "messaging_product": "whatsapp",
    "to": "919350549635",
    "type": "text",
    "text": { "body": "NEW BULK ENQUIRY | Event: {event} | Date: {headcount} | Items: {items}" }
  }
  ```
  In Botpress: use an *Execute Code* node with `axios` + `Authorization: Bearer <WHATSAPP_TOKEN>`.

---

## 4. OPTION 3 → TIMINGS & LOCATION

**Bot:**
```
🏪 SHOP TIMINGS & LOCATION

📍 Address: Famous Pehowa Chowk, Kaithal, Haryana
🕗 Timings: Roz 8:00 AM – 9:00 PM (No off day!)

🗺️ Maps: https://maps.google.com/?q=Pehowa+Chowk+Kaithal+Haryana
```

---

## 5. FALLBACK

**Bot:** `Samajh nahi aaya ji 😅 — [1] Menu • [2] Bulk Order • [3] Timings`

---

## Botpress quick-setup

1. Botpress Studio → **Create Bot** → choose *Empty Bot*.
2. Copy `botpress_flow.json` into the **Flows** editor (or rebuild nodes per above script).
3. Set variables: `event`, `headcount`, `items` (type: text).
4. In `bulk_done` node add a **Webhook / Execute Code** action for the owner alert.
5. Connect channel: Botpress → **WhatsApp** (Twilio/Meta Cloud API) or **Telegram** for testing, then point the channel number to the shop's WhatsApp Business number.

## Voiceflow quick-setup

1. Create project → single **Main** flow.
2. Sequence of blocks: Text (greeting) → Choice (3 options) → branches to Menu / Bulk (4 steps: Ask → capture variables → Summary → IF yes/no) / Info.
3. Add a **API** block after the bulk confirmation to call the owner-alert webhook (CallMeBot URL or Meta Graph API).
4. Connect the **WhatsApp** channel, test, publish.