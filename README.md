README: Daily TCHA & Outage Analysis
Overview
This Jupyter notebook processes daily incident ticket data to calculate Mean Time to Repair (MTTR) metrics and generate Technology Health Analysis (TCHA) reports with outage summaries across multiple network technologies (2G, 3G, 4G, 5G, TDD).

Features
Data Processing: Cleans and standardizes incident ticket data including fault times, recovery times, and site counts

MTTR Calculation: Computes repair times with proper date boundary handling

Category Mapping: Enriches data with categories from a lookup table (SubRootcuz.xlsx)

Outage Analysis: Calculates technology-specific outage metrics

TCHA Reporting: Generates formatted summary reports for each technology with exclusion calculations

Input Files
Incident Ticket Data: Excel file containing daily incident records

SubRootcuz.xlsx: Lookup table mapping Root Cause/Sub Cause to categories

Output Files
result.xlsx: Processed data with all calculated columns

In-memory DataFrames: Technology-specific summary tables (2G, 3G, 4G, 5G, TDD)

Key Calculations
MTTR: Time difference between fault occurrence and recovery

Outage Metrics: Site-seconds outage by technology

Exclusion Formulas: TCHA adjustments based on category-specific outages

Data Filtering: Removes ER-category tickets and empty site provinces

Usage
Configure file paths and target date in the Configuration section

Run all cells sequentially

Check output for processing logs and summary statistics

Access result.xlsx for detailed data and summary DataFrames for reports

Dependencies
pandas

openpyxl

datetime
