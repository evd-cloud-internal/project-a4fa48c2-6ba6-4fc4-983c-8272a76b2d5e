---
name: Optimisely
assetId: 707c0efe-24c4-4e52-99b8-5ecac688c22e
type: page
---

# 🌐 Network Team — Optimisely

```sql network_leads
SELECT
    brand,
    lead_name,
    owner_name,
    stage,
    deal_outcome AS lead_status,
    network_relationship,
    contact_name,
    company_name,
    city,
    project_category,
    scope_of_work,
    amount_lakhs,
    age_in_days,
    lead_source,
    lead_sourced_by,
    formatDateTime(toDate(last_activity_at), '%d %b %Y') AS last_activity_date,
    formatDateTime(toDate(created_at), '%d %b %Y') AS created_date,
    toDate(created_at) AS created_date_raw,
    IF(closing_date IS NULL OR closing_date = toDate('1970-01-01'), 'Not Set',
        formatDateTime(closing_date, '%d %b %Y')
    ) AS closing_date_display,
    coalesce(closing_date, toDate('1970-01-01')) AS closing_date_raw,
    call_count,
    event_count,
    task_count,
    total_activities
FROM alloydb_marts_fct_network_pipeline
ORDER BY last_activity_at DESC
```

## Filters

{% dropdown
    id="brand_filter"
    data="network_leads"
    value_column="brand"
    title="Brand"
    multiple=true
/%}

{% dropdown
    id="owner_filter"
    data="network_leads"
    value_column="owner_name"
    title="Sales Owner"
    multiple=true
/%}

{% dropdown
    id="company_filter"
    data="network_leads"
    value_column="company_name"
    title="Company"
    multiple=true
/%}

```sql filtered_leads
SELECT *,
    CASE
        WHEN lead_status = 'Won' THEN concat(char(60), 'span style="color: #4ade80; font-weight: 600;"', char(62), lead_name, char(60), '/span', char(62))
        WHEN lead_status = 'Lost' THEN concat(char(60), 'span style="color: #f87171; font-weight: 600;"', char(62), lead_name, char(60), '/span', char(62))
        ELSE concat(char(60), 'span style="color: #fbbf24; font-weight: 600;"', char(62), lead_name, char(60), '/span', char(62))
    END AS lead_name_styled
FROM {{network_leads}}
WHERE {{brand_filter.filter}}
  AND {{owner_filter.filter}}
  AND {{company_filter.filter}}
```

## 📊 Pipeline by Stage

{% row %}

{% big_value
    data="filtered_leads"
    value="count(*)"
    title="Open Lead Count"
    where="lead_status = 'Open'"
    sparkline={
        type="bar"
        x="created_date_raw"
        date_grain="month"
    }
/%}

{% big_value
    data="filtered_leads"
    value="sum(amount_lakhs) / 100"
    title="Open Pipeline Value"
    fmt="#,##0.00' Cr'"
    where="lead_status = 'Open'"
    sparkline={
        type="area"
        x="created_date_raw"
        date_grain="month"
    }
/%}

{% big_value
    data="filtered_leads"
    value="count(*)"
    title="Won Lead Count"
    where="lead_status = 'Won'"
    sparkline={
        type="bar"
        x="created_date_raw"
        date_grain="month"
    }
/%}

{% big_value
    data="filtered_leads"
    value="sum(amount_lakhs) / 100"
    title="Won Pipeline Value"
    fmt="#,##0.00' Cr'"
    where="lead_status = 'Won'"
    sparkline={
        type="area"
        x="created_date_raw"
        date_grain="month"
    }
/%}

{% /row %}

```sql pipeline_by_stage
SELECT
    brand,
    CASE stage
        WHEN 'Enquiry' THEN '1. Enquiry'
        WHEN 'Qualified' THEN '2. Qualified'
        WHEN 'Scope & Selection' THEN '3. Scope & Selection'
        WHEN 'Sampling' THEN '4. Sampling'
        WHEN 'Quotation' THEN '5. Quotation'
        WHEN 'Negotiation' THEN '6. Negotiation'
        WHEN 'Commercials Closed' THEN '7. Commercials Closed'
        ELSE '8. Other'
    END AS stage_ordered,
    amount_lakhs
FROM {{filtered_leads}}
WHERE lead_status = 'Open'
```

{% table
    data="pipeline_by_stage"
    title="Pipeline Value by Brand & Stage (₹ Lakhs)"
%}
    {% dimension value="brand" /%}
    {% pivot value="stage_ordered" sort="asc" /%}
    {% measure value="sum(amount_lakhs)" fmt="#,##0.0' L'" /%}
{% /table %}

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
ORDER BY closing_date_raw DESC
```

{% table data="won_leads" %}
    {% dimension value="lead_name" title="Lead Name" /%}
    {% dimension value="brand" /%}
    {% dimension value="owner_name" title="Owner" /%}
    {% dimension value="closing_date_display" title="Closing Date" /%}
    {% dimension value="company_name" title="Company" /%}
    {% dimension value="contact_name" title="Contact" /%}
    {% dimension value="amount_lakhs" title="Amount (₹ L)" fmt="#,##0.0' L'" /%}
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
    {% dimension value="amount_lakhs" title="Amount (₹ L)" fmt="#,##0.0' L'" /%}
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
ORDER BY amount_lakhs DESC
LIMIT 15
```

{% table data="high_value_leads" %}
    {% dimension value="lead_name" title="Lead Name" /%}
    {% dimension value="brand" /%}
    {% dimension value="owner_name" title="Owner" /%}
    {% dimension value="stage" /%}
    {% dimension value="amount_lakhs" title="Amount (₹ L)" fmt="#,##0.0' L'" /%}
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
    {% dimension value="amount_lakhs" title="Amount (₹ L)" fmt="#,##0.0' L'" /%}
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
    {% dimension value="amount_lakhs" title="Amount (₹ L)" fmt="#,##0.0' L'" /%}
    {% dimension value="age_in_days" title="Age (Days)" /%}
{% /table %}