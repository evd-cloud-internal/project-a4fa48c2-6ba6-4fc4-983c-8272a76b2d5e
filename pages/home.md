---
name: Home
assetId: 332a607c-a791-4818-ab55-c3e5bb112879
type: page
---

# 🌐 Network Team — Consolidated Lead View

This dashboard provides a consolidated view of leads across all four brand CRMs for the Network Sales team (Utkarsh, Chandni & Sumita).

```sql filtered_leads
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
    ) AS closing_date,
    call_count,
    event_count,
    task_count,
    total_activities,
    CASE
        WHEN network_relationship IN ('owned', 'owned_and_sourced') THEN
            CASE WHEN owner_name = 'Sumita' THEN 'Sumita Bajaj' ELSE owner_name END
        WHEN network_relationship = 'sourced' THEN
            CASE WHEN lead_sourced_by = 'Sumita' THEN 'Sumita Bajaj' ELSE lead_sourced_by END
    END AS network_member,
    CASE
        WHEN deal_outcome = 'Won' THEN '🟢'
        WHEN deal_outcome = 'Lost' THEN '🔴'
        ELSE '🟡'
    END AS status_indicator,
    countIf(deal_outcome = 'Won') OVER () * 1.0 / count(*) OVER () AS win_rate
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

## Pipeline Snapshot

{% row %}

{% big_value
    data="filtered_leads"
    value="count(*) as open_leads"
    where="lead_status = 'Open'"
    title="Open Leads"
/%}

{% big_value
    data="filtered_leads"
    value="sum(amount_lakhs) as open_pipeline"
    where="lead_status = 'Open'"
    fmt="#,##0.0' L'"
    title="Open Pipeline Value"
/%}

{% big_value
    data="filtered_leads"
    value="sum(amount_lakhs) as won_value"
    where="lead_status = 'Won'"
    fmt="#,##0.0' L'"
    title="Won Value"
/%}

{% big_value
    data="filtered_leads"
    value="sum(amount_lakhs) as lost_value"
    where="lead_status = 'Lost'"
    fmt="#,##0.0' L'"
    title="Lost Value"
/%}

{% big_value
    data="filtered_leads"
    value="avg(win_rate)"
    fmt="pct1"
    title="Win Rate"
/%}

{% /row %}



## Lead Details

{% table data="filtered_leads" page_size=20 search=false subtotals=false %}
    {% dimension value="status_indicator" title="Status" /%}
    {% dimension value="brand" /%}
    {% dimension value="lead_name" /%}
    {% dimension value="owner_name" /%}
    {% dimension value="stage" /%}
    {% dimension value="contact_name" /%}
    {% dimension value="company_name" /%}
    {% dimension value="city" /%}
    {% dimension value="project_category" /%}
    {% dimension value="scope_of_work" /%}
    {% dimension value="amount_lakhs" title="Amount (₹L)" /%}
    {% dimension value="age_in_days" /%}
    {% dimension value="lead_source" /%}
    {% dimension value="last_activity_date" /%}
    {% dimension value="created_date" /%}
    {% dimension value="closing_date" /%}
    {% dimension value="total_activities" /%}
    {% dimension value="lead_status" /%}
{% /table %}

