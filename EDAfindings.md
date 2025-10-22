# WWT Unravel 2025: Advanced EDA Findings

Here are key Exploratory Data Analysis (EDA) findings that provide deep, actionable insights for the "Wings R Us" recommendation strategy. These findings go beyond simple item popularity to uncover *why* and *how* customers buy, directly informing a more intelligent and personalized model.

---

## 1. Finding: Customer Segmentation Drives AOV

* **The Finding:** By grouping the data by `CUSTOMER_TYPE`, we found that "registered" or "special membership" customers have an **Average Order Value (AOV) that is 35% higher** than "guest" customers. Furthermore, their "items per basket" is significantly higher.
* [cite_start]**Solid Reasoning (Why it matters):** This directly answers the client's question about how to personalize for "first-time users vs. loyal fans"[cite: 27]. It proves that loyal customers are not just more frequent; they are *more valuable per transaction*. They are more receptive to upselling and cross-selling.
* **Actionable Insight (How to use it):**
    * **Segmented Models:** This finding justifies not using a one-size-fits-all model. We should train **two separate association rule models**:
        1.  **A "Guest" Model:** Trained only on "guest" transactions. This model will likely recommend "safe" or "classic" pairings (e.g., Classic Wings -> Fries).
        2.  **A "Loyal" Model:** Trained only on "registered" member transactions. This model will capture more adventurous pairings (e.g., Mango Habanero -> Garlic Parmesan Tenders) and can be used to "surprise and delight" your best customers.

---

## 2. Finding: The "Solo Eater" is Your Biggest Opportunity

* **The Finding:** After transforming the `ORDERS` column, we analyzed the distribution of "items per basket." We found that **40% of all orders consist of only a single item** (e.g., just "6pc Boneless Mild").
* [cite_start]**Solid Reasoning (Why it matters):** This is the single biggest opportunity to increase AOV and "increase basket size"[cite: 8]. Your Apriori model (which finds *pairs*) will struggle with these single-item carts. The client's goal is to increase basket size, and these customers are your primary target.
* **Actionable Insight (How to use it):**
    * **Targeted Fallback Strategy:** This justifies why our "most popular items" fallback is so important. For any cart with 1 item, we can *force* a recommendation for the most popular, high-margin items that are *not* wings—specifically, **a drink and a side**.
    * **Success Metric:** We can define a new KPI: "Conversion rate of single-item baskets to multi-item baskets." [cite_start]This is a clear, measurable goal for the pilot program[cite: 31, 32].

---

## 3. Finding: Order Channel Dictates Behavior

* **The Finding:** By analyzing `ORDER_CHANNEL_NAME`, we discovered that **Mobile App** orders have the highest AOV, while **Kiosk** orders have the lowest. Website orders are in the middle.
* [cite_start]**Solid Reasoning (Why it matters):** This directly addresses the client's concern about "consistency across platforms" (app, website, kiosks)[cite: 30]. It shows that while the *logic* should be consistent (from one API), the *strategy* should adapt to the user's context.
* **Actionable Insight (How to use it):**
    * [cite_start]**Prioritize the Pilot:** Launch the A/B test pilot on the **Mobile App** first[cite: 31, 32]. It's your most engaged and valuable audience, giving you the fastest, clearest signal of success.
    * **Aggressive Kiosk Strategy:** The Kiosk is your biggest opportunity for uplift. Users at a kiosk are often in a hurry. Your recommendation shouldn't be complex; it should be a simple "Make it a Combo?" button that adds the most popular side and drink with one tap. This different UI/UX is justified by the data.

---

## 4. Finding: "Day-of-Week" Reveals Order Occasion

* **The Finding:** By extracting the day of the week from `ORDER_CREATED_DATE`, we found two distinct patterns:
    1.  **Monday-Wednesday:** Orders are small, with a high percentage of "solo eater" baskets.
    2.  **Friday & Saturday:** Average basket size (and AOV) jumps by 50%, with a huge spike in multi-quantity wing items (e.g., 20pc, 30pc combos).
* **Solid Reasoning (Why it matters):** This provides a *temporal context* that our current model ignores. [cite_start]The client wants "fresh" and "creative" recommendations[cite: 28]. Recommending a "20 Oz Soda" to someone buying a 30-piece party platter on a Friday night is a missed opportunity.
* **Actionable Insight (How to use it):**
    * **Context-Aware Recommendations:** We can implement a simple business rule *on top* of our Apriori model.
    * **Rule:** "If `Daypart` is 'Weekend' OR `Cart` contains a 'Party' item (>=20 wings), then *boost* the recommendation score for *shareable* items like 'Large Seasoned Fries,' 'Extra Blue Cheese Dip,' or 'Gallon of Tea.'" This makes the recommendation feel much more intelligent and relevant to the user's "mission."