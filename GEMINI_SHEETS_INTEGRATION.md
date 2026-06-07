# Google Sheets + Gemini AI Integration Guide

You can connect your Google Sheet to the Gemini API using a simple **Google Apps Script**. This allows the bakery owner to type a raw item name, and have Gemini automatically fill in the category, image keys, Malayalam translations, and correct spelling errors.

---

## 🛠️ Step-by-Step Setup

### 1. Get a Gemini API Key (Free)
1. Go to the [Google AI Studio](https://aistudio.google.com/).
2. Log in with your Google Account.
3. Click **Get API Key** and copy it.

### 2. Add the Apps Script to Google Sheets
1. Open your Goutam Krishna Sweets Google Sheet.
2. In the top menu, go to **Extensions → Apps Script**.
3. Delete any code in the editor and paste the following script:

```javascript
const GEMINI_API_KEY = "PASTE_YOUR_GEMINI_API_KEY_HERE";

/**
 * Clean up item names, categories, or tags using Gemini AI.
 * =GEMINI("Prompt goes here", cell)
 */
function GEMINI(prompt, cellValue) {
  if (!cellValue) return "";
  
  const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${GEMINI_API_KEY}`;
  
  const payload = {
    contents: [{
      parts: [{
        text: `${prompt}\nInput: "${cellValue}"\nOutput ONLY the clean final result value, no explanations, no quotes, no extra text.`
      }]
    }]
  };
  
  const options = {
    method: "post",
    contentType: "application/json",
    payload: JSON.stringify(payload),
    muteHttpExceptions: true
  };
  
  try {
    const response = UrlFetchApp.fetch(url, options);
    const json = JSON.parse(response.getContentText());
    if (json.candidates && json.candidates[0].content.parts[0].text) {
      return json.candidates[0].content.parts[0].text.trim();
    }
    return "Error: Empty response";
  } catch (err) {
    return "Error: " + err.toString();
  }
}

/**
 * Auto-determine Type (cake or sweet)
 * =AUTO_TYPE(A2)
 */
function AUTO_TYPE(name) {
  return GEMINI("Classify this bakery item. Output exactly 'cake' or 'sweet'.", name);
}

/**
 * Auto-determine Category
 * =AUTO_CATEGORY(A2)
 */
function AUTO_CATEGORY(name) {
  return GEMINI(
    "Classify this item into one of these categories: custom, chocolate, fruit, special, occasion, laddu, pak, milk, fried. Output only the category name in lowercase.",
    name
  );
}

/**
 * Auto-determine the closest image key from the list of valid keys
 * =AUTO_IMAGE_KEY(A2)
 */
function AUTO_IMAGE_KEY(name) {
  return GEMINI(
    "Match this item to the single closest key from this list: customized, barbie, rainbow, dream, box, blackforest, chocolate, truffle, choconut, kitkat, ferrero, blackmagic, vancho, whiteforest, redvelvet, strawberry, blueberry, mango, orange, coconut, biscoff, cheese, nutty, almond, pista, pistachio, redbee, honeyalmond, marble, ghee, honey, plum, tea, carrot, butterscotch, wedding, birthday, babyshower, anniversary, mini, cupcakes, laddu, datesladdu, nutsroll, kajukatli, mysorepak, gheepak, carrotpak, milkgheepak, halwa, gulabjamun, rasgulla, rasmalai, milkkhova, jalebi, babyjalebi, sweetboondi, savories. Output only the key.",
    name
  );
}

/**
 * Auto-generate Malayalam translation/transliteration
 * =AUTO_ALT(A2)
 */
function AUTO_ALT(name) {
  return GEMINI("Translate/transliterate this bakery item name to Malayalam (e.g. 'ലഡ്ഡു'). Format like: 'English alt name · മലയാളം പേര്'. Keep it very short.", name);
}
```

4. Replace `"PASTE_YOUR_GEMINI_API_KEY_HERE"` with your real API key.
5. Click the **Save** (disk) icon at the top of the editor. Close the Apps Script tab.

---

## 🚀 How to use it in Google Sheets

Once saved, these custom formulas will work just like standard Excel formulas!

| If Column A is (name) | Column B formula (alt) | Column C formula (type) | Column D formula (category) | Column F formula (image_key) |
|-----------------------|------------------------|-------------------------|-----------------------------|------------------------------|
| `Laddu`               | `=AUTO_ALT(A2)`        | `=AUTO_TYPE(A2)`        | `=AUTO_CATEGORY(A2)`        | `=AUTO_IMAGE_KEY(A2)`        |

### Example:
If you type `baby jalebi` in Column A:
- **`=AUTO_ALT(A2)`** will automatically fill in: `Cheru Jilebi · ചെറു ജിലേബി`
- **`=AUTO_TYPE(A2)`** will automatically fill in: `sweet`
- **`=AUTO_CATEGORY(A2)`** will automatically fill in: `fried`
- **`=AUTO_IMAGE_KEY(A2)`** will automatically fill in: `babyjalebi`

This makes it extremely easy for a non-tech person. They only need to type the **name** and **price**, and the AI takes care of all the technical tags, keys, and categories automatically!
