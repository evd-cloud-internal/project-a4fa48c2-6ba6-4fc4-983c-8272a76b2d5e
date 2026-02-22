---
name: Pipeline Health
assetId: 707c0efe-24c4-4e52-99b8-5ecac688c22e
type: page
---

# 🌐 Network Team — Optimisely

```sql filtered_leads
SELECT
    brand,
    stage,
    deal_outcome AS lead_status,
    amount_lakhs,
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

## 📊 Pipeline by Stage

{% row %}

{% big_value
    data="filtered_leads"
    value="count(*)"
    title="Open Lead Count"
    where="lead_status = 'Open'"
    filters=["brand_filter", "member_filter"]
/%}

{% big_value
    data="filtered_leads"
    value="sum(amount_lakhs) / 100"
    title="Open Pipeline (₹ Cr)"
    fmt="num2"
    where="lead_status = 'Open'"
    filters=["brand_filter", "member_filter"]
/%}

{% big_value
    data="filtered_leads"
    value="count(*)"
    title="Won Lead Count"
    where="lead_status = 'Won'"
    filters=["brand_filter", "member_filter"]
/%}

{% big_value
    data="filtered_leads"
    value="sum(amount_lakhs) / 100"
    title="Won Pipeline (₹ Cr)"
    fmt="num2"
    where="lead_status = 'Won'"
    filters=["brand_filter", "member_filter"]
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
  AND {{brand_filter.filter}}
  AND {{member_filter.filter}}
```

{% table
    data="pipeline_by_stage"
    title="Pipeline Value by Brand & Stage (₹ Lakhs)"
%}
    {% dimension value="brand" /%}
    {% pivot value="stage_ordered" sort="asc" /%}
    {% measure value="sum(amount_lakhs)" fmt="num1" /%}
{% /table %}

{% table
    data="pipeline_by_stage"
    title="Pipeline Count by Brand & Stage"
%}
    {% dimension value="brand" /%}
    {% pivot value="stage_ordered" sort="asc" /%}
    {% measure value="count(*)" /%}
{% /table %}

