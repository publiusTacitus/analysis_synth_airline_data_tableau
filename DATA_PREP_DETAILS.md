# Data Preparation Overview (Pandas)

Data preparation for this Tableau project was primarily performed in Python using pandas, with the objective of 
transforming normalized CSV sources into analysis-ready, Tableau-friendly flat tables. The preparation pipeline 
emphasizes realistic airline-domain modeling, careful handling of nulls and aggregation semantics, and a clear 
separation between data engineering and visualization.

## 1. Data Loading and Standardization

- Loaded multiple CSV sources, including `flights`, `bookings`, `costs_per_flight`, `customers`, `routes`, `airports`,
  `aircraft`, and `weather`.
- Applied consistent parsing rules across all sources:
  - Semicolon-delimited files
  - German decimal format
  - Explicit datetime parsing for all temporal fields

## 2. Lookup and Dimension Tables

Several lookup and dimension tables were created to enrich fact tables and support clean, relationship-based modeling 
in Tableau.

#### Airports

- Cleaned and standardized airport names.
- Added climate region attributes.
- Used to:
  - Generate long-form route names
  - Enrich the weather hazards fact table with geographic and climate metadata

#### Routes

- Generated full route descriptors from origin and destination airport names.
- Created short route identifiers from airport codes.
- Added distance categories.
- Derived temporal dimensions from flight dates:
  - Travel season
  - Weekday group
- Used as a shared dimension connecting multiple performance fact tables in Tableau.

#### Customers

- Cleaned and standardized frequent flyer status descriptors.
- Included gender and nationality attributes.
- Calculated canonical customer ages using date of birth and earliest recorded flight dates.
- Assigned age group categories.
- Used to connect customer-related fact tables in Tableau.

## 3. Financial and Operational Performance Tables

### 3.1 Route Facts

As an intermediate step, an enhanced flight-level fact table was created:
- Aggregated metrics from per-booking and flight × cabin class levels to the flight level:
  - Total revenue per flight (sum of ticket payments)
  - Booking and passenger totals (booking counts and check-ins)
  - Cabin configuration, derived from non-zero cabin class capacities:
    - Economy Only
    - Economy, Business
    - Economy, Business, First
- Added seat capacities and per-flight operating costs.
- Included derived temporal dimensions (travel season, weekday group).

From this, the final `route_facts` table – supporting Tableau visualizations of financial and selected operational 
metrics – was created by aggregating flight-level data by:
- Route name
- Cabin configuration
- Travel season
- Weekday group

Aggregated metrics include:
- Flight counts
- Capacity, booking, passenger, and revenue totals
- Multiple cost-type totals

**Performance Tiers**  
Route-level performance tiers were derived from average booked rates and profit margins:
- **Booked Performance Tier**
  - (A) Top Performance (> 85% avg. booked rate)
  - (B) Within Target (76–85%)
  - (C) Sufficient (70–75%)
  - (D) Underperforming (< 70%)
  - (E) Unsustainable (< 60%)

- **Profitability Tier**
  - (A) Top Profit Margin (≥ 25%)
  - (B) High Profit Margin (20–24%)
  - (C) Healthy Profit Margin (15–19%)
  - (D) Small Profit Margin (5–14%)
  - (E) Loss Risk (< 5%)

### 3.2 Aircraft Usage

The `aircraft_usage` fact table summarizes flight activity by:
- Aircraft manufacturer, model, seat capacity, and range
- Route name
- Travel season and weekday group

This table supports fleet utilization and aircraft–route suitability analysis.

### 3.3 Flight Delays and Cancellations

Two fact tables were created to support operational performance analysis.

**Flight Delays**
- Calculated delay minutes from scheduled vs. actual arrival times.
- Derived delayed/on-time indicators using a 15-minute threshold.
- Carefully handled missing delay reasons to avoid artificial category inflation during aggregation.
- Aggregated delay statistics by:
  - Route name
  - Weekday group
  - Travel season
  - Departure and arrival delay reasons

Resulting metrics include:
- Flight count
- Average delay minutes
- Maximum delay minutes

**Cancellations**
- Counted canceled flight IDs.
- Aggregated by:
  - Route name
  - Weekday group
  - Travel season
  - Cancellation reason

## 4. Weather Hazards

- Weather observations were joined with airport metadata to add:
  - Airport name
  - Climate region
- Boolean hazard indicators were derived (e.g. fog, blizzard, extreme heat/cold, strong wind, low visibility).
- Hazards were aggregated by:
  - Climate region
  - Airport
  - Season
- Airport coordinates were included to support geographic visualization in Tableau.

## 5. Customers

Two customer-focused fact tables were created.

**Customer Attributes**  
- Counted distinct customers summarized by:
  - Nationality
  - Gender
  - Age group
  - Frequent flyer status
- Provides insight into demographic and loyalty distributions.

**Customer Behavior**
- Counted bookings and check-ins summarized by:
  - Booking lead-time windows (six categories, ranging from four to twelve weeks)
  - Booking time-of-day windows (four time ranges)
  - Cabin class
  - Demographic and loyalty attributes
- Provides insight into booking and travel behavior.

## 6. Export for Tableau Consumption

All final datasets were exported as CSV files with:
- Consistent delimiters and decimal formats
- Flattened schemas
- Explicit categorical dimensions

Minimal additional data preparation was required inside Tableau beyond defining relationships and 
visualization-specific calculated fields.

This preparation pipeline demonstrates a production-style approach to analytical data modeling, combining realistic 
domain logic with Tableau-optimized schemas.