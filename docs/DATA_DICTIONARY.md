# Data Dictionary — SaaS Growth & Unit Economics Dataset

Source: `SaaS_Growth_Unit_Economics_Dataset.xlsx`, sheet `Data_Dictionary` (63 documented columns across 7 tables). Reproduced here for quick reference; the workbook sheet is the source of truth.

## Customers — 5,000 rows

| Column | Type | Description | Business meaning | Example | Required | Relationships | Notes |
|---|---|---|---|---|---|---|---|
| CustomerID | Text | Unique customer identifier | Primary key for the customer dimension | CUST-00001 | Yes | PK; referenced by Subscriptions, Subscription_Events, Customer_Health | Sequential, zero-padded |
| CustomerName | Text | Company/account name | Identifies the customer organization | Nova Analytics Inc. | Yes | - | Contains a few intentional whitespace/duplicate-name data-quality issues for Power Query cleanup practice |
| SignupDate | Date | Date the customer first signed up | Customer acquisition date; cohort anchor | 2023-04-12 | Yes | Joins to Date table | Matches the StartDate of the customer's first Subscription |
| Country | Text | Customer's country | Geographic segmentation | United States | Yes | - | A few rows use inconsistent casing/abbreviations intentionally |
| Region | Text | Geographic region derived from Country | Roll-up geography for regional analysis | North America | Yes | - | Deterministically derived from Country |
| Industry | Text | Customer's industry vertical | Vertical/segment performance analysis | Technology | Yes | - | A few rows have inconsistent casing intentionally |
| CompanySize | Text | Company size band | Firmographic segmentation (Small/Medium/Large/Enterprise) | Medium | Yes | Correlates with CustomerSegment | - |
| CustomerSegment | Text | Commercial segment | SMB / Mid-Market / Enterprise segmentation used across KPIs | Mid-Market | Yes | Correlates with CompanySize and Plan mix | - |
| AcquisitionChannel | Text | Marketing/sales channel that acquired the customer | CAC and channel-performance analysis | Paid Search | Yes | Joins to Marketing.AcquisitionChannel | - |
| AccountManager | Text | Assigned account manager | Book-of-business / AM performance analysis | Priya Patel | No | - | A few rows intentionally blank |

## Plans — 5 rows

| Column | Type | Description | Business meaning | Example | Required | Relationships | Notes |
|---|---|---|---|---|---|---|---|
| PlanID | Text | Unique plan identifier | Primary key for the plan dimension | PLAN-01 | Yes | PK; referenced by Subscriptions, Subscription_Events | - |
| PlanName | Text | Plan name | Product tier naming | Starter | Yes | - | - |
| PlanTier | Text | Descriptive tier label | Groups plans for tier-level analysis | Entry | Yes | - | - |
| MonthlyPrice | Decimal | List monthly price (USD) | Base price used to normalize MRR for monthly-billed subscriptions | 49.00 | Yes | - | - |
| AnnualPrice | Decimal | List annual price (USD) | Base price for annual-billed subscriptions; ~20% discount vs Monthly x 12 | 470.40 | Yes | - | Normalized monthly MRR = AnnualPrice / 12 |
| IncludedUsers | Whole Number | Number of seats included in the plan | Used to contextualize ActiveUsers vs plan allowance | 5 | Yes | - | - |

## Subscriptions — 7,307 rows

| Column | Type | Description | Business meaning | Example | Required | Relationships | Notes |
|---|---|---|---|---|---|---|---|
| SubscriptionID | Text | Unique subscription identifier | Primary key for the subscription fact/bridge table | SUB-000001 | Yes | PK; referenced by Subscription_Events | A customer may have multiple sequential subscriptions (plan changes) |
| CustomerID | Text | Owning customer | Links subscription to customer dimension | CUST-00001 | Yes | FK -> Customers.CustomerID | - |
| PlanID | Text | Subscribed plan | Links subscription to plan dimension | PLAN-03 | Yes | FK -> Plans.PlanID | - |
| StartDate | Date | Subscription start date | When this subscription record became effective | 2023-04-12 | Yes | Joins to Date table | - |
| EndDate | Date | Subscription end date | When this subscription record stopped being effective (superseded or churned) | 2024-02-01 | No | Joins to Date table | Blank for currently active subscriptions |
| BillingFrequency | Text | Monthly or Annual billing | Determines MRR normalization method | Monthly | Yes | - | - |
| MonthlyPrice | Decimal | Normalized monthly recurring revenue for this subscription (USD) | Annual contracts are normalized to AnnualPrice/12; this is NOT the annual contract value | 199.00 | Yes | - | Reflects seat/usage adjustments made within the same plan |
| Status | Text | Active or Churned | Whether this specific subscription record is currently in force | Active | Yes | - | 'Churned' covers both customer churn and being superseded by a plan-change subscription |

## Subscription_Events — 19,532 rows

| Column | Type | Description | Business meaning | Example | Required | Relationships | Notes |
|---|---|---|---|---|---|---|---|
| EventID | Text | Unique event identifier | Primary key for the MRR-movement fact table | EVT-000001 | Yes | PK | - |
| CustomerID | Text | Customer associated with the event | Links event to customer dimension | CUST-00001 | Yes | FK -> Customers.CustomerID | - |
| SubscriptionID | Text | Subscription associated with the event | Links event to the subscription it affected | SUB-000001 | Yes | FK -> Subscriptions.SubscriptionID | - |
| EventDate | Date | Date the MRR movement occurred | Drives MRR waterfall / time-intelligence measures | 2023-06-15 | Yes | Joins to Date table | - |
| EventType | Text | New / Expansion / Contraction / Churn | Classifies the MRR movement type | Expansion | Yes | - | - |
| PreviousMRR | Decimal | MRR immediately before the event (USD) | Basis for MRR waterfall | 199.00 | Yes | - | 0 for New events |
| NewMRR | Decimal | MRR immediately after the event (USD) | Basis for MRR waterfall | 399.00 | Yes | - | 0 for Churn events |
| MRRChange | Decimal | NewMRR - PreviousMRR (USD) | Net MRR impact of the event; sums to Net MRR Change | 200.00 | Yes | - | Always equals NewMRR - PreviousMRR |
| PreviousPlanID | Text | Plan before the event, if applicable | Plan-change analysis | PLAN-02 | No | FK -> Plans.PlanID (nullable) | Null for New events |
| NewPlanID | Text | Plan after the event, if applicable | Plan-change analysis | PLAN-03 | No | FK -> Plans.PlanID (nullable) | Null for Churn events |
| ChurnReason | Text | Reason for churn | Churn-reason analysis | Lack of Usage | No | - | Populated only for EventType = Churn; a few rows have inconsistent casing intentionally |

## Marketing — 288 rows (36 months x 8 channels)

| Column | Type | Description | Business meaning | Example | Required | Relationships | Notes |
|---|---|---|---|---|---|---|---|
| Date | Date | First day of the reporting month | Monthly grain for marketing performance | 2023-01-01 | Yes | Joins to Date table (month grain) | - |
| AcquisitionChannel | Text | Marketing/sales channel | Channel-level performance and CAC analysis | Paid Search | Yes | Joins to Customers.AcquisitionChannel | - |
| MarketingSpend | Decimal | Spend for the channel in the month (USD) | Numerator for CAC | 18450.00 | Yes | - | - |
| Leads | Whole Number | Leads generated for the channel in the month | Funnel/conversion analysis | 410 | Yes | - | - |
| NewCustomers | Whole Number | New customers acquired via the channel in the month | Denominator for CAC; reconciles exactly with Customers.SignupDate x AcquisitionChannel | 46 | Yes | - | - |

## Customer_Health — 25,283 rows

| Column | Type | Description | Business meaning | Example | Required | Relationships | Notes |
|---|---|---|---|---|---|---|---|
| HealthRecordID | Text | Unique health-observation identifier | Primary key for the health fact table | HR-000001 | Yes | PK | - |
| Date | Date | Observation date | Time-series tracking of customer health | 2023-07-01 | Yes | Joins to Date table | Snapshots occur roughly every 3 months per customer, plus signup and churn months |
| CustomerID | Text | Customer observed | Links health record to customer dimension | CUST-00001 | Yes | FK -> Customers.CustomerID | - |
| ProductUsage | Decimal | Normalized product usage score (0-100) | Engagement indicator; correlates with HealthScore and inversely with churn risk | 72.5 | Yes | - | - |
| ActiveUsers | Whole Number | Active seats in use | Adoption relative to Plans.IncludedUsers and CompanySize | 12 | Yes | - | - |
| SupportTickets | Whole Number | Support tickets opened since last observation | Support-burden indicator; tends to rise as HealthScore falls | 2 | Yes | - | - |
| NPS | Whole Number | Net Promoter Score style rating (-100 to 100) | Sentiment indicator, correlated with HealthScore | 35 | Yes | - | - |
| HealthScore | Decimal | Composite customer health score (0-100) | Primary churn-risk indicator; lower scores probabilistically increase churn hazard | 64.0 | Yes | - | - |

## Date — 1,096 rows (daily grain, 2023-2025)

| Column | Type | Description | Business meaning | Example | Required | Relationships | Notes |
|---|---|---|---|---|---|---|---|
| Date | Date | Calendar date | Grain of the date dimension | 2023-01-01 | Yes | PK; joins to all fact tables | One row per day, 2023-01-01 to 2025-12-31 |
| Year | Whole Number | Calendar year | Yearly roll-ups | 2023 | Yes | - | - |
| Quarter | Text | Calendar quarter label | Quarterly roll-ups | Q1 | Yes | - | - |
| QuarterNumber | Whole Number | Quarter number (1-4) | Sorting/filtering by quarter | 1 | Yes | - | - |
| MonthNumber | Whole Number | Month number (1-12) | Sorting/filtering by month | 1 | Yes | - | - |
| MonthName | Text | Month name | Display label for month | January | Yes | - | - |
| YearMonth | Text | Year-Month label (YYYY-MM) | Monthly grain display label | 2023-01 | Yes | - | - |
| YearMonthSort | Whole Number | Sortable year-month key (YYYYMM) | Ensures correct chronological sort of YearMonth | 202301 | Yes | - | - |
| WeekNumber | Whole Number | ISO week number | Weekly analysis | 1 | Yes | - | - |
| DayOfMonth | Whole Number | Day of month (1-31) | Daily analysis | 1 | Yes | - | - |
| DayOfWeek | Text | Day-of-week name | Day-of-week analysis | Sunday | Yes | - | - |
| DayOfWeekNumber | Whole Number | Day-of-week number (1=Monday..7=Sunday) | Sorting day-of-week | 7 | Yes | - | - |
| IsMonthEnd | Boolean | True if the date is the last day of its month | Period-end flag for MRR snapshot logic | FALSE | Yes | - | - |
| IsQuarterEnd | Boolean | True if the date is the last day of its quarter | Period-end flag | FALSE | Yes | - | - |
| IsYearEnd | Boolean | True if the date is Dec 31 | Period-end flag | FALSE | Yes | - | - |
