---
name: Actionable
assetId: 25a7d401-8512-4bc2-905c-89ac7460ca4e
type: page
---

# 🌐 Network Team — Optimisely

```sql filtered_leads
SELECT
    brand,
    lead_name,
    owner_name,
    stage,
    deal_outcome AS lead_status,
    contact_name,
    company_name,
    amount_lakhs,
    age_in_days,
    lead_source,
    formatDateTime(toDate(last_activity_at), '%d %b %Y') AS last_activity_date,
    formatDateTime(toDate(created_at), '%d %b %Y') AS created_date,
    IF(closing_date IS NULL OR closing_date = toDate('1970-01-01'), 'Not Set',
        formatDateTime(closing_date, '%d %b %Y')
    ) AS closing_date_display,
    coalesce(closing_date, toDate('1970-01-01')) AS closing_date_raw,
    CASE
        WHEN network_relationship IN ('owned', 'owned_and_sourced') THEN
            CASE WHEN owner_name = 'Sumita' THEN 'Sumita Bajaj' ELSE owner_name END
        WHEN network_relationship = 'sourced' THEN
            CASE WHEN lead_sourced_by = 'Sumita' THEN 'Sumita Bajaj' ELSE lead_sourced_by END
    END AS network_member
FROM alloydb_marts_fct_network_pipeline
ORDER BY last_activity_at DESC
```

## Filters

{% dropdown
    id="brand_filter"
    data="filtered_leads"
    value_column="brand"
    title="Brand"
    multiple=true
/%}

{% dropdown
    id="member_filter"
    data="filtered_leads"
    value_column="network_member"
    title="Network Member"
    multiple=true
/%}

## 🏆 Recently Won Leads

```sql won_leads
SELECT
    lead_name,
    brand,
    owner_name,
    company_name,
    contact_name,
    amount_lakhs,
    closing_date_display,
    lead_source,
    created_date
FROM {{filtered_leads}}
WHERE lead_status = 'Won'
  AND {{brand_filter.filter}}
  AND {{member_filter.filter}}
ORDER BY closing_date_raw DESC
```

{% table data="won_leads" %}
    {% dimension value="lead_name" title="Lead Name" /%}
    {% dimension value="brand" /%}
    {% dimension value="owner_name" title="Owner" /%}
    {% dimension value="closing_date_display" title="Closing Date" /%}
    {% dimension value="company_name" title="Company" /%}
    {% dimension value="contact_name" title="Contact" /%}
    {% dimension value="amount_lakhs" title="Amount (₹L)" fmt="num1" /%}
    {% dimension value="lead_source" title="Lead Source" /%}
    {% dimension value="created_date" title="Created" /%}
{% /table %}

## ⏰ Leads Closing Soon (Next 15 Days)

```sql closing_soon
SELECT
    lead_name,
    brand,
    owner_name,
    stage,
    company_name,
    contact_name,
    amount_lakhs,
    closing_date_display,
    age_in_days
FROM {{filtered_leads}}
WHERE lead_status = 'Open'
  AND closing_date_raw != toDate('1970-01-01')
  AND closing_date_raw >= today()
  AND closing_date_raw <= today() + INTERVAL 15 DAY
  AND {{brand_filter.filter}}
  AND {{member_filter.filter}}
ORDER BY closing_date_raw ASC
```

{% table data="closing_soon" %}
    {% dimension value="lead_name" title="Lead Name" /%}
    {% dimension value="brand" /%}
    {% dimension value="owner_name" title="Owner" /%}
    {% dimension value="closing_date_display" title="Closing Date" /%}
    {% dimension value="stage" /%}
    {% dimension value="company_name" title="Company" /%}
    {% dimension value="contact_name" title="Contact" /%}
    {% dimension value="amount_lakhs" title="Amount (₹L)" fmt="num1" /%}
    {% dimension value="age_in_days" title="Age (Days)" /%}
{% /table %}

## 🔥 High-Value Open Leads

```sql high_value_leads
SELECT
    lead_name,
    brand,
    owner_name,
    stage,
    company_name,
    contact_name,
    amount_lakhs,
    last_activity_date,
    closing_date_display,
    age_in_days,
    lead_source
FROM {{filtered_leads}}
WHERE lead_status = 'Open'
  AND amount_lakhs > 0
  AND {{brand_filter.filter}}
  AND {{member_filter.filter}}
ORDER BY amount_lakhs DESC
LIMIT 15
```

{% table data="high_value_leads" %}
    {% dimension value="lead_name" title="Lead Name" /%}
    {% dimension value="brand" /%}
    {% dimension value="owner_name" title="Owner" /%}
    {% dimension value="stage" /%}
    {% dimension value="amount_lakhs" title="Amount (₹L)" fmt="num1" /%}
    {% dimension value="company_name" title="Company" /%}
    {% dimension value="closing_date_display" title="Closing Date" /%}
    {% dimension value="last_activity_date" title="Last Activity" /%}
    {% dimension value="age_in_days" title="Age (Days)" /%}
{% /table %}

## 😴 Stale Leads — No Activity in 30+ Days

```sql stale_leads
SELECT
    lead_name,
    brand,
    owner_name,
    created_date,
    last_activity_date,
    stage,
    company_name,
    contact_name,
    amount_lakhs,
    closing_date_display,
    age_in_days
FROM {{filtered_leads}}
WHERE lead_status = 'Open'
  AND age_in_days > 30
  AND {{brand_filter.filter}}
  AND {{member_filter.filter}}
ORDER BY age_in_days DESC
```

{% table data="stale_leads" %}
    {% dimension value="lead_name" title="Lead Name" /%}
    {% dimension value="brand" /%}
    {% dimension value="owner_name" title="Owner" /%}
    {% dimension value="created_date" title="Created" /%}
    {% dimension value="last_activity_date" title="Last Activity" /%}
    {% dimension value="stage" /%}
    {% dimension value="company_name" title="Company" /%}
    {% dimension value="contact_name" title="Contact" /%}
    {% dimension value="amount_lakhs" title="Amount (₹L)" fmt="num1" /%}
    {% dimension value="closing_date_display" title="Closing Date" /%}
    {% dimension value="age_in_days" title="Age (Days)" /%}
{% /table %}

## 🚨 Overdue Leads — Closing Date Crossed

```sql overdue_leads
SELECT
    lead_name,
    brand,
    owner_name,
    closing_date_display,
    stage,
    company_name,
    contact_name,
    amount_lakhs,
    age_in_days
FROM {{filtered_leads}}
WHERE lead_status = 'Open'
  AND closing_date_raw != toDate('1970-01-01')
  AND closing_date_raw < today()
  AND {{brand_filter.filter}}
  AND {{member_filter.filter}}
ORDER BY closing_date_raw ASC
```

{% table data="overdue_leads" %}
    {% dimension value="lead_name" title="Lead Name" /%}
    {% dimension value="brand" /%}
    {% dimension value="owner_name" title="Owner" /%}
    {% dimension value="closing_date_display" title="Closing Date" /%}
    {% dimension value="stage" /%}
    {% dimension value="company_name" title="Company" /%}
    {% dimension value="contact_name" title="Contact" /%}
    {% dimension value="amount_lakhs" title="Amount (₹L)" fmt="num1" /%}
    {% dimension value="age_in_days" title="Age (Days)" /%}
{% /table %}