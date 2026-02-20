## Experiment Spec: Friends & Family Referral Campaign (Retail, Network Interference)

### 1. Business Objective

Measure the true incremental impact of a “Friends & Family” referral promotion (10% off coupon + shareable referral code) on profit and revenue for a mid-sized omnichannel retailer (stores + e‑commerce), recognizing that customers influence each other through social connections (network spillovers) which can bias a standard A/B test readout.

### 2. Primary Hypotheses

- **H0 (business outcome)**: Treatment (receiving the Friends & Family email + coupon) increases incremental profit per targeted customer versus control.
- **H1 (mechanism)**: Untreated customers who are connected to treated customers (treated neighbors) will show higher conversion and revenue (spillover effect) than untreated customers with few or no treated neighbors.


### 3. Metrics Framework

#### 3.1 North Star Metric

- **Incremental Profit per Customer (IPC)**  
  \[
  \Delta \text{Profit/Customer} = \overline{\text{profit}}_{T} - \overline{\text{profit}}_{C}
  \]
  - Profit should include discount cost and variable costs (or contribution margin if available).
  - This is the single headline metric used to decide whether to scale the campaign.



#### 3.2 Success Metrics (Primary Outcomes)

These are the main KPIs we want to improve:

- **Incremental Revenue per Customer (ARPC lift)**  
  - Definition: \(\overline{\text{revenue}}_{T} - \overline{\text{revenue}}_{C}\)
- **Incremental Conversion Rate (CR lift)**  
  - Definition: \(\overline{\text{converted}}_{T} - \overline{\text{converted}}_{C}\)
- **Incremental Orders per Customer** (optional, if purchase frequency is important)  
  - Definition: \(\overline{\text{orders}}_{T} - \overline{\text{orders}}_{C}\)
- **Referral-Driven Revenue Share**  
  - Definition: revenue from orders using a referral code ÷ total revenue  
  - Interprets how much of the lift is coming via the intended referral mechanism.

#### 3.3 Guardrail Metrics (Must Not Degrade)

Guardrails ensure that we do not “buy” growth in a way that harms margin, experience, or operations.

**Margin / Discount Health**

- **Gross Margin % / Contribution Margin %**  
  - Requirement: should not materially decrease versus control beyond an agreed tolerance.
- **Average Discount Rate**  
  - Requirement: ensure incremental revenue is not purely driven by excessive discounting.

**Customer Experience / Brand Risk**

- **Return / Refund Rate**
- **Order Cancel Rate**
- **Customer Support Contacts per 1,000 Customers** (or complaint rate)

Requirement: these should not increase materially in the treatment group vs control.

**Operational Health**

- **Stockout Rate / Inventory Availability**
- **Delivery SLA Breach Rate** (for e‑commerce orders)

Requirement: avoid operational overload caused by the campaign.

**Customer Quality (Short-to-Mid Term)**

- **Repeat Purchase Rate** in a follow-up window (e.g., next 30–60 days) or an LTV proxy.

Requirement: treatment should not attract only “one-time deal seekers” who never return.

---

### 4. Network / Interference Diagnostic Metrics

These metrics are used to understand how network effects create bias in the naive A/B results. They are **diagnostic**, not the direct success criteria.

- **Share of Treated Neighbors per Customer**
  - Definition: number of treated neighbors ÷ total neighbors.
  - Interpretation: measures exposure intensity to treated peers.

- **Spillover Lift Among Untreated Customers**
  - Compare outcomes for:
    - **Untreated-High-Exposure**: untreated customers with many treated neighbors.
    - **Untreated-Low-Exposure**: untreated customers with few or no treated neighbors.
  - Difference in conversion and revenue between these two groups quantifies the spillover.

- **Direct vs Indirect Effect Split**
  - **Direct Effect**: effect of own treatment on outcomes while holding neighbor exposure roughly fixed.
  - **Indirect Effect (Spillover)**: effect of neighbors’ treatment on untreated customers.

- **Cluster / Community-Level Lift**
  - Compute average lift at network community (e.g., graph clusters or store-region communities).
  - Helps show how effects concentrate in certain social or geographic clusters.

---

### 5. Segmentation

All success and guardrail metrics should be inspected across key retail segments:

- **Channel**: store vs online vs omnichannel.
- **Customer Lifecycle**: new vs active vs lapsed.
- **Value Segment**: low, mid, high value based on baseline spend or propensity.
- **Region / Store Cluster**: geographic or store-based clustering.

This helps ensure the campaign is not harming or underperforming for critical subgroups.

---

### 6. Decision Rule (Example)

Scale the Friends & Family referral campaign if:

1. **North Star Metric**  
   - Incremental Profit per Customer is positive and exceeds a predefined minimum uplift threshold (e.g., +X currency units per 1,000 targeted customers).

2. **Guardrail Metrics**  
   - No guardrail metric violates agreed thresholds (e.g., margin drop < Y percentage points, return rate increase < Z%, no major operational issues).

3. **Network Diagnostics**  
   - Network analysis clarifies whether lift is driven by:
     - Mostly **direct effects** (own coupon use), or
     - Significant **spillover effects** (friends/family using referrals).
   - Insights are used to refine **targeting strategy** (for example, focusing on high-centrality customers to maximize spillovers).

If these conditions are not met, either do not scale the campaign or iterate on the offer and targeting strategy before re-testing.
